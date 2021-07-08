# Advanced Lane Finding Project

###Rijo Thomas

---
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

[image1]: ./output_images/calibration3_unDistor.jpg "Undistorted calibration image"
[image2]: ./test_images_undist/undist_straight_lines1.jpg "Undistorted straiht lane"
[image3]: ./output_images/undist_straight_lines1_thresh_binary.jpg "Undistorted straight lane Binary"
[image4]: ./output_images/BirdView_undist_straight_lines1.jpg "Warp Example"
[image5]: ./output_images/fit_poly_test6.jpg "Fit Visual"
[image6]: ./output_images/lane_curve_dev_test6.jpg "Output"
[video1]: ./output_video/project_video.mp4 "Project Video Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration


The code for this step is contained in the code cells 2,3 and 4 of the IPython notebook located in "./Advanced_Lane_lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell 9 of the  ./Advanced_Lane_lines.ipynb file).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes in the cell 13,14 and 15 of the file `./Advanced_Lane_lines.ipynb`. The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[592,450],[688,450],[1100,700],[210,700]])
dst = np.float32([[200,0],[1100,0],[1100,700],[200,700]])
    
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 592,450       | 200,0         | 
| 688,450       | 1100,0        |
| 1100,700      | 1100,700      |
| 210,700       | 200,700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The steps for identifying the lane line pixels and marking them are included in the cells 20,21 and 22 of the file. The birds eye view of road is used in this phase. First of all, I filtered out the points which make the two lanes and stored them sepaerately. I used sliding window technique for the same. Then I calculated the polynomial functions which fit these points using the function polyfit. I used these polynomial fits to calculate the points which make up the lane. I used a function called polyfil to represent the area between the lanes using a coloured polygon.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for calculating the radius of curvature and position of the vehicle from the center is provided in the cell 22 of the file. 

Radius of curvature

I used insight of below expression from the lesson, to clculate the calculate the coeficients:
x= mx / (my ** 2) *a*(y**2)+(mx/my)*b*y+c.Here, the scaled fit coefficients are mx / (my ** 2) *a and (mx/my)*b. here mx and my are the scaling coefficients in x and y axis repectively and are in m per pixel.

Vehicle deviation form center:

lane_sep= position_of_left_lane - position_of_right_lane
lane_center = position_of_left_lane+(lane_sep)/2
vehicle_center = width_of_image/2
vehicle_deviation = vehicle_center - lane_center

I converted these values to metre from pixel using scaling coefficients mx and my.
mx=3.7/700 
my=30/720


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the cell 24 and 25 my code. I did a reverse perpective transform of the identified lanes to form a mask. Then, I superimposed this mask on the original image of the road. I also added textual information like radius of curvature and deviation of vehicle from center.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

In order to approach the problem in a more organized way, I decided to use oops concept for a pipeline which detects lane from the video. This was very helpful for resolving errors. I created two classes, frame and lane, which have a relationship between them. The reused the code from the above image pipeline in the class frame. In order to keep track of values which are changed across different frames I used several class variables. 

To smooth out the lane values I used an approach which calculates the average for the same from last n frames.

I included a logic which reuses the coefficients of the fit calculated in the previous frame to predict the new lane points.
To make sure that this feature is ket in check, I also included a sanity check logic which checked for anomalies in the detected lanes.

The sanity check looked for anomalies in n consecutive frames and would trigger a more cautious lane detection.

I noticed that the pipeline is prone to failure in the presence of walls and other markings which translate to false edges in the binary image.

I had to modify the sanity logic to produce a more complete output for the challenge videos. This made the sanity state very sesitive to even small anomolies thereby slowing down the lane detection.

To improve the deficiency, I would include a better sanity check logic and a better binary image generator.



