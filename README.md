
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

[image1]: ./camera_cal/calibration1.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"


---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

- Original Image

<img src="./camera_cal/calibration1.jpg" alt="drawing" width="300"/>

- Undistored Image

<img src="./output_images/camera_calibrated_img.jpg" alt="drawing" width="300"/>

Once the camera is calibrated, various threshold like H, S, V, Gradient Magnitude, Direction Mag etx. are applied.
The results after applying different thresholds are as shown below.

<img src="./output_images/various_thresolds.jpg" alt="drawing" width="600"/>


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

<img src="./output_images/undistorted_test2.png" alt="drawing" width="800"/>

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.Here's an example of my output for this step. 

<img src="./output_images/combined_threshold.jpg" alt="drawing" width="800"/>

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_img()`, which appears in lines 303 in 8th block (Perspective transform block) in P2.ipynb jupyter notebook.  The `warp_img()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 746, 491      | 1090, 0       | 
| 1090, 700      | 1090, 700     |
| 220, 700     | 200, 700     |
| 540, 491     | 200, 0]       |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<img src="./output_images/birds_eye.png" alt="drawing" width="800"/>

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used sliding window technique to identify the lane-line pixels.
Histogram peaks are used to detect the lanes. Small rectangles (windows) were drawn to get the all the windows and eventually the polynomial.

<img src="./output_images/sliding_Window.png" alt="drawing" width="800"/>



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature of the both left and right lanes are calculated by using the following code snippet - 

```
    left_curverad = ((1 + (2*left_fit[0]*y_eval + left_fit[1])**2)**1.5) / np.absolute(2*left_fit[0])
    right_curverad = ((1 + (2*right_fit[0]*y_eval + right_fit[1])**2)**1.5) / np.absolute(2*right_fit[0])
    
```
Then the average of the both are taken and displayed on the image.
The offset of the lane center from the center of the image (converted from pixels to meters) is your distance from the center of the lane.
Offset is the distance between the lane center from the center of the image. The logic for this is implemented in `get_offset` method.


<img src="./examples/color_fit_lines.jpg" alt="drawing" width="400"/>


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

All the logic for this pipeline has been written in the `process_image` method which takes a colored image and returns the image with the lanes plotted.

Input - 

<img src="./test_images/straight_lines1.jpg" alt="drawing" width="400"/>

Output - 

<img src="./output_images/processed_image.jpg" alt="drawing" width="400"/>

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main issue that I faced was while applying the thresholds. It was bit difficult to get the correct threshold values for each channel and combining them.
I am pretty sure there's a better combination than what I am currently using.

Processing the video is taking a lot of time.

Also, I faced some issue with Jupyter notebook. My method was taking the value that was defined in the previous code block and it took a lot of time to figure out what actually was going wrong.

The pipeline will fail to detect the lanes under bad lighing conditions.
The results for the current frame can be used to detect the lanes in the next frame.
Deep learning techniques could yield better results in detecting lanes.