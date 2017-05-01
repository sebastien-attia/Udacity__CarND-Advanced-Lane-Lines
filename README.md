## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


**Advanced Lane Finding Project**

The goals / steps of this project are the following:

1. Calibrate the camera
2. Undistort the image
3. Define the perspective transform to have a bird eye view of the road
4. Define the Sobel filter
5. Define the process pipeline  
    a- Undistort the image  
    b- Remove the part of the image to keep only the region of interest  
    c- Use color transform to highlight the white and yellow lanes  
    d- Apply the bird-eye view  
    e- Apply the Sobel filter to highlight the lanes  
    f- Find the lanes and compute the curvature  
    g- Unwarp the bird-eye view  
    h- Add the found lanes image to the original image  

[//]: # (Image References)

[image1]: ./output_images/Camera_calibration.png "Camera calibration"
[image2]: ./output_images/Undistort_image.png "Undistort image"
[image3]: ./output_images/Bird-eye-view_road.png "Bird eye view of the road"
[image4]: ./output_images/Thresholded_gradients.png "Sobel filter"
[image5]: ./output_images/find_lanes_1.png "Find lanes 1"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./output_images/debug_project.mp4 "Debug project video"
[video2]: ./output_images/project.mp4 "Project video"
[video3]: ./output_images/debug_challenge.mp4 "Debug project video"
[video4]: ./output_images/challenge.mp4 "Project video"
[video5]: ./output_images/debug_harder_challenge.mp4 "Debug project video"

---

#### 1. Calibration of the camera.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced_Finding_Lanes_Project.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![Camera calibration][image1]

### Pipeline (single image)

#### 1. Undistort the image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Undistort the image][image2]

#### 2. Keep only the region of interest to find the lanes.
The image is cropped to keep only the half bottom of the image and remove the part on the side, to keep only the perspective (see the debug video below).

#### 3. Apply the bird-eye view.
Apply the bird-eye view on the road lanes, to remove the perspective (see the debug video below).

The following transformation is used:

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 535, 460      | 0, 0        |
| 750, 460      | 1280, 0      |
| 1480, 720     | 1280, 720      |
| -180, 720      | 0, 720        |

![Bird-eye view][image3]

#### 4. Apply a color filter to highlight the white and yellow lanes.
This filter is applied in the function filter_white_yellow(), in the 11th cell.
The RGB image is transform to HSV and:
1. a (yellow OR white) mask is applied,
2. the previous OR mask is applied on the image

(see the debug video below).

#### 5. Apply the Sobel filter.
The class ImageSobelThresholder defines and apply the different steps of the Sobel filter,

![Sobel filter][image4]

#### 6. Find the lanes.

The function lanes_finder() find the lanes from the binary image (created after applying the Sobel filter).
2 algorithms are used to find a position of the lanes, in each vertical window:
1. the first algo use the convolution to find the peaks in the first part and the second part of the image;
2. the second algo finds all maximas and compute an average weighted with the value of the peak.

Each image is split vertically into (nb_window) windows.
The class Lanes keeps track models the positions of the lanes (right and left), for each vertical window.

At the construction of the class, if there is no position for a window, the missing data is computed as following:
1. if it is the last window (ie the one on top), take the position of the previous window;
2. if it is the first window, find the first window having a position and take this value;
3. else take the position of the previous window

To estimate the positions of the lanes:
1. for each window, compute a box with a margin around the position found previously;
2. take all points within this box activated (ie having a value of 1);
3. compute a second degree polynom for each lane, fitting the points found previously.
4. draw the zone of interest defines by these 2 computed lanes

![Find lanes][image5]

#### 7. Unwarp the bird-eye view.
Here the inverse of the bird-eye view is applied to have the image added to the original image.

#### 8. Display the result.
The following video is debug version of the final project video [Project debug video][video1]

And the final project video [Project video][video2].

For the challenge: [Challenge debug video][video3] and [Challenge video][video4].  
And for the harder challenge: [Harder challenge debug video][video5]
