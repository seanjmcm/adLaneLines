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

[image1]: ./Report_Images/undistort_output.png "Undistorted Chess"
[image2]: ./test_images/test1.jpg "Sample"
[image21]: ./Report_Images/undistort.jpg "undistorted image"
[image3]: ./Report_Images/thresh_bin_im.jpg "Binary Example"
[image4]: ./Report_Images/warped_lines.png "Warp Example"
[image5]: ./Report_Images/lanefit.png "Fit Visual"
[image6]: ./Report_Images/Mapped2Road.png "Output"
[video1]: ./Report_Video/video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./camcal.ipynb`.  

There was an issue with pyqt so I reinstalled pyqt as follows:

`conda install -c anaconda pyqt=4.11.4`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result for `calibration1.jpg`: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

From the previous code, I had saved the camera calibration coefficients (mtx and dist) in a pickle file called data.pkl.  Using "./adLane.ipynb". I loaded the values and then used them in conjunction with the sample image and the opencv function undistort.  The image below is the undistorted image.

![alt text][image21]

A very slight difference is noticeable in the area of the car visible in the frame.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at line #1 in code cell 3 of `adLane.ipynb`).  
`def pipeline(img, s_thresh=(90, 255), sx_thresh=(20, 100)):`

The values can be modified if required but assume the above defaults otherwise.  The defaults were used in this project.

I converted the undistorted image to grayscale and used the sobel operator to take the derviative in the x direction.

I then coverted the original undistorted RGB image to a HLS Color space and selected the S channel which was descriobed as the most effective at discovering yellow lines in the notes.

Both arrays were converted to binary images and then comibined into a single array using the following code

`combined_binary = np.zeros_like(sxbinary)`
`combined_binary[(s_binary == 1) | (sxbinary == 1) ] = 1`

Here's an example of my output for this step.  



![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `scene_unwarp()`, which appears in code cell 4  lines 1 through 16 in the file `adLane.ipynb`.  The `scene_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  As I was only working with 1280x720 images, I chose the to hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[200.,720.],  
    [453.,547.],
    [835.,547.],
    [1100.,720.]])
dst = np.float32(
    [[320.,720.], 
    [320., 590.5],  
    [960.,590.5],
    [960.,720.]])
```

I did not base the source and destination points on the actual image size as frther work would be required to see if that method worked with different image sizes.

The matching source and destination points were thus:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200.,720.     | 320.,720.      | 
| 453.,547.     | 320., 590.5    |
| 835.,547.     | 960.,590.5     |
| 1100.,720.    | 960.,720.      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.  The is shown in the below image which was converted to RGB in order to better show the lines.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I took a histogram of the bottom half of the warped image:

`histogram = np.sum(binary_warped[binary_warped.shape[0]/2:,:], axis=0)`

I cut this in half and looked for the left and right peaks.

Next I used 9 sliding windows to follow the lane lines in the warped image.  Each window had a height of (Warped Image Size/9) 

This develops a patterna as shown below.

![alt text][image5]

The identified pixels are then used to form a second order polynomoial fitted line as shown in green in the above picture.



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code section 6 in my code in `adLane.ipynb`.

I repeated the polynomial fit but this time multiplied the left and right lane pixels by ym_per_pix and xm_per_pix as appropriate.

As per the notes, these generally have the following values according to highway rules:
ym_per_pix = 30/720
xm_per_pix =  3.7/700

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code section 6 in my code in `adLane.ipynb` in the function `laneIm(warpedout,left_fitx, ploty, right_fitx, Minv,midpoint, curvature,undistIm)`.  I used the inverse perspective transform to place the warped detecftion lines back on the undistorted picture.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./Report_Video/video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The video kept crashing at approximately 96% because I was using array number 719 to detect the midpoint of the car on the road.  In general this works quite well but if there is significantly less detail on the road it will cause issues.  Instead it would be better to use an average of polynomial fitted curve points at approximately 95 - 100 pixels from the car.  This would be a more consistent approach.

The road shading is presenting problems to the algorithm.  A running average or smoothing would assist with rejecting false shading lines. 
