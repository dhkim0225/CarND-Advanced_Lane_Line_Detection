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

[image1]: ./output_images/undistort_chessboard.png "Undistorted"
[image2]: ./output_images/test_images_output/combined_threshold_straight_lines1.png "Binary Example"
[image3]: ./output_images/test_images_output/warped_straight_lines.png "Warp Example"
[image4]: ./output_images/test_images_output/find_lines_with_window.png "Find lane_line with windows"
[image5]: ./output_images/test_images_output/find_lines_without_window.png "Find lane_line without windows"
[image6]: ./output_images/test_images_output/final_output.png "Output"
[video1]: ./project_video.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first, second code cell of the IPython notebook located in "./examples/example.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I have objpoints and imgpoints so I could easily transform and store distorted images.

like this.

img1 = cal_undistort(img1, objpoints, imgpoints)


#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`.
The `unwarp()` function takes as inputs an image (`img`),
I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[300, 655], [550, 480], [730, 480], [1000, 650]])
dst = np.float32([[400, 650], [400, 0], [800, 0], [800, 650]]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 300, 655      | 400, 650      |
| 550, 480      | 400, 0        |
| 730, 480      | 800, 0        |
| 1000, 650     | 800, 650      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.
Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image2]

Its' process is in `getCombined()` function.
The getCombined function receives the image of the image and return binary image.

I use 6 datas to make lane recognition.

Here's 2 datas.
The weight of x direction and y direction was set to 7:3 when xy-sobel was used.

v-channel of hlv and sobel by xy-direction - threshold 220, 255.
r-channel of rgb and sobel by xy-direction - threshold 220, 255.

Here's 4 datas. (Just use cv2.in_range function)
HSV - threshold (20, 100, 100), (50, 255, 255)
HSV - threshold (0, 0, 197), (255, 20, 255))
HLS - threshold (0, 195, 0), (255, 255, 60))
RGB - threshold (200,200,200), (255,255,255))

Return the combined binary created by 6 elements in the getCombined function.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

Find lane_lines with windows.
![alt text][image4]

1. Use the nonzero function to get the index of all non-zero values.
2. Calculate the vertical axis where the nonzero value is most distributed through the histogram, and draw a window based on the vertical axis.
3. Divide the image 9 times and repeat the second operation 9 times.
4. Based on the obtained nonzero values, the best fit quadratic function was calculated. Using polyfit, you can easily find the quadratic function.

Find lane_lines without windows.
![alt text][image4]

This function is used when polyfit values are obtained by using window first.
1. Find the nonzero value.
2. Find nonzero values for nonzero quadratic functions.
3. Get new polyfit values based on the data obtained.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This is code to calculate curvature and the position of the vehicle.
```python
# measure curvature
y_eval = np.max(ploty)

# Define conversions in x and y from pixels space to meters
# meters per pixel in y dimension
ym_per_pix = 30.0/720
# meters per pixel in x dimension 
xm_per_pix = 3.7/700 

# Fit new polynomials to x,y in world space
left_fit_cr = np.polyfit(ploty*ym_per_pix, left_fitx*xm_per_pix, 2)
right_fit_cr = np.polyfit(ploty*ym_per_pix, right_fitx*xm_per_pix, 2)

# Calculate the new radii of curvature
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

#Calculate distance about vihacle left form center
bottom_left = 320 - left_fitx[719]
left_from_center = bottom_left * xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in`draw_info()` function.  Here is an example of my result on a test image:

![alt text][image6]

I draw lines with many informations and after drawing lines, I transformed it back to original img.
And then, I put texts in top of image.
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_output.mp4)

I get the frame via videopy
1. Undistort the distorted image.
2. Use the getCombined function to obtain a binary image.
3. Use the unwarp function to look down the road from the sky.
4. Use the window to locate the line.
5. If you have found a line using the window, use the window to find the line.
6. Find the value of curvation and how far it is from the center.
7. If the curvation value found outside the window is far from the expected value, use the window again to find the line.


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As you can see from the video, my pipeline does not properly handle shader-like values. It seems to fail to find appropriate threshold values.
In actual road conditions, road form changes very much, though it may not be appropriate in real time, but I thought of a good solution.
Cut the image horizontally, like my pipeline, and keep adjusting the threshold value until the number of nonzero values you want is reached. Continue to change the threshold values and get the polyfit values even if the desired threshold value is found. This value is compared with the polyfit value obtained from the previous frame and the closest polyfit value is used.

I will also make this pipeline if time permits.

