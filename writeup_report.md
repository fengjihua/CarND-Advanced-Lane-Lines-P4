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

[image1]: ./examples/pipeline_1.png "Chessboard"
[image2]: ./examples/pipeline_2.png "Calibration"
[image3]: ./examples/00_3_0.jpg "Undisroted"
[image4]: ./examples/pipeline_3.png "Color Channel"
[image5]: ./examples/pipeline_3_2.png "Color Channel"
[image6]: ./examples/00_3_1.jpg "Binary image"
[image7]: ./examples/00_3_2.jpg "Binary magnitude image"
[image8]: ./examples/41_19.jpg "Binary magnitude detect lines"
[image9]: ./examples/41_19_clear.jpg "Binary magnitude clear noise"
[image10]: ./examples/pipeline_4_3.png "Binary clear noise"
[image11]: ./examples/color_fit_lines.jpg "Color fit lines"
[image12]: ./examples/00_3_3.jpg "Binary clear noise"
[image13]: ./examples/00_3_4.jpg "Fit lane lines"
[image14]: ./examples/pipeline_7_2.png "Warp fit poly"
[image15]: ./examples/00_3_6.jpg "Final image"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

The code for this step is contained in the first code cell of the IPython notebook located in "./P4.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Apply a distortion correction to raw images.

I defined `undisort()` function to apply the distortion correction to one of the test images like this one:
![alt text][image3]

#### 2. Use color transforms, gradients, etc., to create a thresholded binary image.

I used a combination of color and gradient thresholds to generate a binary image (function `binary_combination()`).  Here's an example of my output for this step.

![alt text][image4]

![alt text][image5]

1. After comparing all RGB, HSV, HLS color channels, I found that S channel of HLS didn't show lane lines on dark shadowed roads, but the V channel of HSV worked better than other channels. I used threshold 230 on V channel to separate lane lines from other things, it showed lane lines clearly even on some white roads. The only problem was that it didn't show lane lines clearly only on some dark shadowed roads, then I used gradient to solve this problem. 

2. I used `cv2.Sobel` function to calculate the gradient, it showed lane lines clearly on dark shadowed roads, but didn't show lines on white roads. Don't worry about white roads, V channel can handle it perfectly. Gradient solved the dark shadowed roads problem, but it made more noises. It was a new problem.

3. I combined V (>230) and Gradient (sobel x) into a binary image. And calculate gradient magnitude of it. I added a mask to both images, because I only interested in the low half of images to detect lane lines.

![alt text][image6]

![alt text][image7]

#### 3. Apply a perspective transform to rectify binary image ("birds-eye view").

1. I defined `detect_lines()` function to detect lane lines in binary magnitude image. First, I used `cv2.findContours` to find contours, then used `cv2.minAreaRect` to fit contours and I got many rectangles. I used `cv2.contourArea` to calculate area of rectangles, and calculated slopes. I used area as weights to calculate average left and right lane line slope. Finally I used average slope to calculate top and bottom points. I used `corners` to save four points (2 for left, 2 for right).

2. I defined `clear_line_noise()` function to clear some noises in binary image. First, I used `corners` and `np.polyfit` to calculate left and right polyfit. Then used polyfit to find left and right 50 nonzero pixels, used `leftx, lefty, rightx, righty` save all the nonzeros pixels near the polyfits. Finally used `leftx, lefty, rightx, righty` to create a clear binary image without noises. (or only a little noises maybe).

Exaple images of `detect_lines()` and `clear_line_noise()`:

![alt text][image8]

![alt text][image9]

After two steps metioned above, I could do perspective transform easily. The `perspective()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. `src` and `dst` points are dynamically from `corners` I calculated before. 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image10]


#### 4. Detect lane pixels and fit to find the lane boundary.

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image11]

The `fit_measure()` function describes how to fit lane lines. Note that I cleared the noise before, it could help fit lines exactly.

![alt text][image12]

![alt text][image13]

#### 5. Determine the curvature of the lane and vehicle position with respect to center.

The `fit_measure()` function shows how to measure curvature and distance of center. I found it was very hard to calculate curvature exactly if my `corners` calculation result had only little erros. After some fine tunes (noise clear and `corners` calculation) I finally got a curvature around 1km on curved way and very large curvature (>3km) on straight way.

#### 6. Warp the detected lane boundaries back onto the original image.

The `draw_lane()` function shows the detail. First, I used `left_fit`, `right_fit` and `corners` to calculate a poly, then used `src`, `dst` and `cv2.getPerspectiveTransform()` to calculate Matrix inverse, finally used the matrix inverse and `cv2.warpPerspective()` draw the poly back to original image. 

![alt text][image14]


#### 7. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Final step, `draw_measure` function used `cv2.putText()` to draw text on the image

![alt text][image15]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

#### 1. Make lane lines clearly on all kinds of roads (normal/white/dark/shadowed/dirty).
1st step: Compare all 9 color channel of RBG HSV and HLS. V channel of HSV is the best. Only got some problems on shadowed roads.

2nd step: Use sobel x to make line clearly on shadowed roads.

#### 2. Detect lines and corners correctly.
Always got error results on detect lines and curvature. I used some other way to replace the Hought tranform, it works better. See function `detect_lines()`.

#### 3. Clear noises before perspective transform
Always got error results on polyfit calculation, because noise pixels sometimes are very close to the right pixels after perspective tranform. That's why I wrote `clear_line_noise()` function. And I think it maybe better if I re-detect the lines `detect_lines()` after `clear_line_noise()`.

#### 4. Search polyfit in perspective image
Search nearest fit is more efficient than always search a new fit. But some fit errors will cause the next image erros, search a new fit always be more robust than search nearest fit. It's a trade-off. Maybe we can do some change to search a new fit only when the last near fit seems not so good. 