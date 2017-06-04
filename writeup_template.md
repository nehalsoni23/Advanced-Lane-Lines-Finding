## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image0]: ./examples/corners_drawn.png "Original Image with corners"
[image1]: ./examples/undistored_image.png "Undistorted Image"
[image2]: ./test_images/test6.jpg "Original Image"
[image3]: ./examples/undist_example.png "Example of Undistorted Image"
[image4]: ./examples/S_Channel_failing.JPG "Demostration of S Channel failure for certain images"
[image5]: ./examples/R_channel_processing.JPG "Demostration of R Channel working fine"
[image6]: ./examples/warped_binary.jpg "Warp Example"
[image7]: ./examples/lane_detected.JPG "Lane Detected with the use of polynomial"
[image8]: ./examples/final_result.JPG "Final Result with Curvature of Lane"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

As I learned in module that distortion can change an apparent size and shape of an object and it can appear closer or further away than they actually are, calibration is very needed to be done as a pre-processing of an image.

For calibrating image, we are using chessboard pictures since in chessboard we can define the points in 3D space as a corners of chessboard and to get an image points I have used multiple 9*6 chessboard images.

In order to calibrate an image and undistort it, we need to define objpoints (3D points in real world space) and imagepoints (2D points in image plane). Objpoints will be (x, y, z) coordinates of the chessboard corners in the real world space i.e. (0,0,0), (1,0,0), (2,0,0) ....,(8,5,0).

To find image points, first image is being converted into grayscale. Then, I am using `findChessboardCorners` function of cv2 which will return image points for corners in (54, 1, 2) shape for 9*6 coners chessboard.

Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image0]
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like shown below:

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

In order to detect lane I only used SobelX threshold and R-channel color threshold. After so many iteration I have choosen these two methods to create an binary image. Please find the following points which I have taken into consideration while adopting these image pre-processing:

- I have used only SobelX operator to find gradient since gradient in the x direction emphasizes edges closer to vertical.

- At first I used S channel of the image after converting the RGB image to HLS. But this processing was failing when there is shodow on lane or in general any dark color upon lane line. Below is a demostration of the explanation:

![alt text][image4]

This exact same image when R channel threshold is being used it gives result as shown below:

![alt text][image5]

Even though R channel is not detecting lane well while using alone but with gradient it works well. And for this same image even with modifying threshold to find the lane line best, s channel fails. 

I have also provided the video of S channel processing for specific timing where it was failing even with proper threshold which works fine for other frames in video.

Here is the link of the video.
Here's a [link to my video result with the use of S channel](./s_channel_video.mp4)

Note: The test image I demostrated here is most affected by using S channel. For write up purpose I have used [this image](./test_images/test6.jpg) as shown on top.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_tranform()`, which appears in 4th code cell of jupyter notebook. The `perspective_tranform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
	[[180, img_size[0]], 
	[575, 460], 
	[705, 460], 
	[1150, img_size[0]]])

dst = np.float32(
	[[280, img_size[0]], 
	[280, 0], 
	[960, 0], 
	[960, img_size[0]]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for detecting line is written in code cell 5. I have followed following steps as explained in `33. Finding the Lines` lesson in 4th module. Find the steps below:

1. First I plotted the histogram of warped image which will have two peak points as shown in image below:

2. Then I devided the histogram in 2 parts vertically and found the index of max pixel value which I have written in code cell for the test image I used. Because of that I found the base index of lane lines.

3. I choose 9 windows to be fit in the image and find height of single window with dividing total height of warped image by window. i.e `window_height = np.int(binary_warped.shape[0]/nwindows) -> 720/9 = 80`

4. The pixel limit is set to 50 `minpix = 50`. So if in one window there are less than 50 pixels found, the base position for life and right lane will be recentred.

5. To find lane pixels on each of the 9 windows for loops is written which will first set the window's height and width. I have written the values of indexes for first window which starts from bottom of the image. Non-zero indexes are found within that window using numpy's nonzero function. Then after comparing number of pixels found in left and right side of image is being compare to `minpix` and 4th step is evaluated.

6. Once the loop is over each window's left and right nonzero indexes are concatenated. From all nonzero indexes of left/right lane indexs found from each of the windows are extracted. From those data second degree polynomial constants are found using numpy's `polyfit` function.

Then I filled the green color to window's area and highlighted the lane lines found with red  and blue color respectively as shown in an image below:

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for finding radius of curvature is written in function `curvature_offset()` in code cell 6. First of all meter per pixels in x and y dimensions is being measured with the help of actual height and weight. Then new polynomial constants are found for new dimension for real world space. After that with the help of equation given in `Measuring Curvature` lesson curvature and vehicle offset are found.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my final video result](./processed_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I am processing each image frame with all the steps explained inspite of saving the polyfit constants for first image and using it for consecutive frames. Still I was able to get the desired output well. I am aware that this approach needs to be changed since it is more time consuming but to achieve that at this moment I need to give more time to sharpen my python programming skills which can take a while.
The main problem I faced with existing approach is as I explained in step 2 above which is the implementation was failing when lane lines are being covered in tree shadow. Using R channel has proved to be beneficial for this particular kind of frames. Although R channel also seems to have an disadvantage when there is more sun light in road. But that time gradient with respect to X can very well detect lanes and combining  them gives desired output. The other approach of applying convolution also seems to be great and also shorter than the one I implemented.
