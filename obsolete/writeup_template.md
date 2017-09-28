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

[image1]: ./Report_Images/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./Report_Video/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./camcal.ipynb".  

The code wouldn't run so I reinstalled pyqt as follows:

conda install -c anaconda pyqt=4.11.4

(carnd-term1) C:\WINDOWS\system32>conda install -c anaconda pyqt=4.11.4
Fetching package metadata .............
Solving package specifications: .

Package plan for installation in environment d:\ProgramData\Anaconda3\envs\carnd-term1:

The following NEW packages will be INSTALLED:

    tk:           8.5.18-vc14_0      anaconda    [vc14]

The following packages will be UPDATED:

    pandas:       0.19.2-np112py35_1 conda-forge --> 0.20.1-np111py35_0 anaconda

The following packages will be SUPERCEDED by a higher-priority channel:

    h5py:         2.7.0-np112py35_0  conda-forge --> 2.7.0-np111py35_0  anaconda
    hdf5:         1.8.17-vc14_10     conda-forge [vc14] --> 1.8.15.1-vc14_4    anaconda [vc14]
    jpeg:         9b-vc14_0          conda-forge [vc14] --> 8d-vc14_2          anaconda [vc14]
    libtiff:      4.0.6-vc14_7       conda-forge [vc14] --> 4.0.6-vc14_2       anaconda [vc14]
    matplotlib:   2.0.0-np112py35_3  conda-forge --> 1.5.1-np111py35_0  anaconda
    numpy:        1.12.1-py35_0                  --> 1.11.3-py35_0      anaconda
    pillow:       4.1.0-py35_0       conda-forge --> 2.9.0-py35_0       anaconda
    pyqt:         5.6.0-py35_2       conda-forge --> 4.11.4-py35_7      anaconda
    pywavelets:   0.5.2-np112py35_0  conda-forge --> 0.5.2-np111py35_0  anaconda
    qt:           5.6.2-vc14_1       conda-forge [vc14] --> 4.8.7-vc14_9       anaconda [vc14]
    scikit-image: 0.13.0-np112py35_0 conda-forge --> 0.13.0-np111py35_0 anaconda
    scikit-learn: 0.18.1-np112py35_1             --> 0.18.1-np111py35_1 anaconda
    scipy:        0.19.0-np112py35_0             --> 0.19.0-np111py35_0 anaconda
    statsmodels:  0.8.0-np112py35_0  conda-forge --> 0.8.0-np111py35_0  anaconda

Proceed ([y]/n)? y

jpeg-8d-vc14_2 100% |###############################| Time: 0:00:00 260.36 kB/s
tk-8.5.18-vc14 100% |###############################| Time: 0:00:02 780.11 kB/s
hdf5-1.8.15.1- 100% |###############################| Time: 0:00:03 484.73 kB/s
libtiff-4.0.6- 100% |###############################| Time: 0:00:01 402.30 kB/s
numpy-1.11.3-p 100% |###############################| Time: 0:00:05 600.37 kB/s
pillow-2.9.0-p 100% |###############################| Time: 0:00:02 575.53 kB/s
h5py-2.7.0-np1 100% |###############################| Time: 0:00:01 573.65 kB/s
pywavelets-0.5 100% |###############################| Time: 0:00:06 639.42 kB/s
qt-4.8.7-vc14_ 100% |###############################| Time: 0:01:19 668.43 kB/s
scipy-0.19.0-n 100% |###############################| Time: 0:00:21 604.91 kB/s
pandas-0.20.1- 100% |###############################| Time: 0:00:18 447.23 kB/s
pyqt-4.11.4-py 100% |###############################| Time: 0:00:06 600.92 kB/s
scikit-learn-0 100% |###############################| Time: 0:00:05 810.09 kB/s
matplotlib-1.5 100% |###############################| Time: 0:00:09 710.84 kB/s
statsmodels-0. 100% |###############################| Time: 0:00:08 727.24 kB/s
scikit-image-0 100% |###############################| Time: 0:00:33 705.29 kB/s

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result for claibration1.jpg: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
