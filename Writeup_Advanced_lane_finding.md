
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

[Undistorted]: ./images/distorted_undistorted.png "Undistorted"
[Transformed]: ./test_images/test2.jpg "Road Transformed"
[Binary]: ./examples/binary_combo_example.jpg "Binary Example"

[FitVisual]: ./examples/color_fit_lines.jpg "Fit Visual"
[testOutput]: ./output_images/test2.jpg "Output"
[histogram]: ./images/histogram.png "histogram"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  
---
All the rubric points are explained in the writeup and in code comments.

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!


I have implemented a code in the "Advance_lane_finding.ipynb" jupyter notebook.


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd and 3rd code cell of the IPython notebook located in "./Advance_lane_finding.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][Undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][Transformed]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in code cell 5).  (note: this is not actually from one of the test images)


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective()`, which appears in code cell 6 in "Advance_lane_finding.ipynb".  The `transform_perspective()` function takes as inputs an image (`img`), as well as source and destination points. If only an image was passed to the function, default 'src' and 'dst' values are calculated as a function of image sized in the following manner:

```python
if M is None:
        if source is None:            
            src = np.array([[575. / 1280. * img_size[1], 460. / 720. * img_size[0]],
                            [705. / 1280. * img_size[1], 460. / 720. * img_size[0]],
                            [1127. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                            [203. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
        else:
            src = source

        if dest is None:
            dst = np.array([[320. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                            [960. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                            [960. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                            [320. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
        else:
            dst = dest
```

The result in a 720x1280 image is the following source and destination points:

| Source        |-------->| Destination   | 
|---------------|-|---------------| 
| 575, 460      |.| 320, 0        | 
| 203, 720      |.| 320, 720      |
| 1127, 720     |.| 960, 720      |
| 705, 460      |.| 960, 0        |



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After the transformations in the image like undistorting the image, binary data extraction, perspective transformation, the next step is the extract the pixels which defines the lane lines. sliding window search is the best methdo to do it. here start point is taken as histogram peak of the lower image.
![alt text][histogram]


i used this to fit detected lane pixels using second order polynomial 

![alt text][FitVisual]

All this is done in the function `find_lanes()` in the code cell 9 in "Advance_lane_finding.ipynb".

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

this is also done in the function `find_lanes()` in code cell 9 "Advance_lane_finding.ipynb". It is done by defining conversions in x and y from pixel space to meters, fitting new polynomials inthe world space and then calculating radius of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 8 in "Advance_lane_finding.ipynb", in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][testOutput]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_lanes.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### Challenges faced
* the first and the major challenge faced is the extraction of the binary data. I have used operator named Sobel X combined with a thresholds in S-chanel from HSV color space. This is the initial implementation but it failed in few frames in project video file. Accordingly i migrated to another method of extraction, taking into account 3 different masks like yellow , white and gradient.
    - The white mask perfectly detected white lane lines. it was in the RGB color space and within a range of (200,200,200) to (255,255,255).
    - Yellow Mask was extracted from HSV color space and it is within a range of (0,100,100) to (80,255,255). It is very good in detecting yellow lines in the image.
    - the gradient mask is extracted with the use of sobel operator in the X- direction with the threshold limits of 30 to 150. It worked perfectly for detecting the verticle lines that are missed due to shadows and bad lighting conditions.

##### Further improvements 
* the method used to detect lanes is very limitted to near perfect light conditions. when pipeline run for the chalenge video this the reason of failure. this perticular binary extraction method needs further tuning to work in badly lit conditions.will try to tune it further.


