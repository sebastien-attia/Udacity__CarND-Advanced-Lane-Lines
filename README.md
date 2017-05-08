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
6. Discussion on how to improve the process

[//]: # (Image References)

[image1]: ./output_images/Camera_calibration.png "Camera calibration"
[image2]: ./output_images/Undistort_image.png "Undistort image"
[image3]: ./output_images/Bird-eye-view_road.png "Bird eye view of the road"
[image4]: ./output_images/Thresholded_gradients.png "Sobel filter"
[image5]: ./output_images/Find_lanes_1.png "Find lanes 1"
[image6]: ./examples/example_output.jpg "Output"

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
The following video is debug version of the final project video:
<iframe width="560" height="315" src="https://www.youtube.com/embed/gg59lBWPJfM" frameborder="0" allowfullscreen></iframe>

And the final project video:
<iframe width="560" height="315" src="https://www.youtube.com/embed/AFnZ4edENXo" frameborder="0" allowfullscreen></iframe>

For the challenge, the debug version:
<iframe width="560" height="315" src="https://www.youtube.com/embed/ykTGXsY4k8E" frameborder="0" allowfullscreen></iframe>

And the final version:
<iframe width="560" height="315" src="https://www.youtube.com/embed/kAk3AjkK7aQ" frameborder="0" allowfullscreen></iframe>

And for the harder challenge, the debug version:
<iframe width="560" height="315" src="https://www.youtube.com/embed/M0PpyVI4Lls" frameborder="0" allowfullscreen></iframe>
And the final version:
<iframe width="560" height="315" src="https://www.youtube.com/embed/95LBjvIq7LM" frameborder="0" allowfullscreen></iframe>

#### 9. Discussion

The biggest problems I have is when:
a. the color of the road changes, from dark to light for example,
b. there are shadows of a tree for example on the road,
c. the road has not an homogeneous color (some dark zone on a white road for example),
In these cases, the different filters (thresholded Color and Sobel filters) are not performant enough to
select only the lanes and add some noises in the cloud of points got by the algorithm.

Several ideas could be tested to improve the quality to find the lanes:
a. A first idea would be to have several set of filters, compute the best polynomial
fitting for each set of filters, compute a scoring for each set (based for example on the
  standard deviation) and keep only the best one.
In the current version of the code, I use a similar approach, where I use 2 algorithms
to find the lanes, and I keep only the one giving the result closer to the middle of the image.

b. A second idea is to use the result of the previous lanes to check the consistency with the
result of the current lanes finding. I currently use only the previous lane to complete
with data missing on the current lane.
We could maybe imagine to superpose the n last bird-eye view images and fit a polynomial
on this "big" image of (n x 720)x1280. And for each frame, we would take the (n-1) frames
to compute the polynomial fit.
