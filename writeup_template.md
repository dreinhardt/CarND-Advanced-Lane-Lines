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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"


### Short outline what I did after the review
Because it took me around 20h to integrate the new code, I hope it is sufficient for you.
I do not know more to improve it and I came to my programming limits...
- integrated wished sanity checks
- re-engineered the overall code
- image quality is much more stable and the warped image is perfect!
Please see further topics within the discussion

### Writeup / README

First thing is the switch at the beginning of the code "picturemode" (picturemode = True/False).
Here you can switch between displaying the pictures or generating the final video.



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


The code starts with calibration of the camera.
here I generate the opject point arrays in the chessboard. I loop through every image and call the findChessboardCorners function of cv2.
After I have the corners I will picture a chessboard with that corners as an example

![image1](./output_images/a.jpg "findChessboardCorners")

After that, I call the calibrateCamera function to receive the necessary parameters for image undistortion. It returns the camera matrix, distortion coefficients, rotation and translation vectors. These parameters are now global.
After that, I generate example pictures of distorted and undistorted chessboards.

![image2](./output_images/b.jpg "undistorted chessboards")


### Pipeline (single images)

The function pipeline(img) is the driving (run) function of the code.

In terms there are problems to see my images, I placed them into the output_images folder.

#### 1. Provide an example of a distortion-corrected image.

The distortion correction I generated a seperate picture for that test which is in the code, as well. It is an identical step to the chessboard pictures before in section 1. The distortion in the original picture is clearly visible.

![image3](./output_images/c.jpg "Thresholds")

Within pipeline(img) the first part is to undistort every picture. Because the parameters for undistort are global, there is no need for a seperate function call.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In function findLanes(.) I use the different methods to clarify my picture. Here I use the HLS color space to separate the S channel. I used the code from  the lessons and modified it a little bit. I played around with the thresholds Furthermore, I used Sobel for the X gradient. That gradient was good to make the lines (lane markers) on the street much more visible.

![image4](./output_images/1.jpg "Undistorted Real Pics")

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Within the function warpImage(.), I chose source points which are close to the lane markers on a steight_lane picture. Here I modified the coordinates for that polygon. Furthermore, I calculated the destination points for the transformed image.
The transforming is done with M = cv2.getPerspectiveTransform(source_points, destination_points) and the inverse (for later) Minv = cv2.getPerspectiveTransform(destination_points, source_points).

I created two image sets for the warping ot clarify whats going on.

![image5](./output_images/2.jpg "Warping Real Pics")


#transformation
source_points = np.float32([[x_shape//2-75, 450],[x_shape//2+75, 450],[x_shape-dx, y_shape], [dx, y_shape]])
destination_points = np.float32([[dx,dy],[x_shape-dx,dy],[x_shape-dx,y_shape-dy],[dx,y_shape-dy]])
    

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 565, 450      | 100, 100      | 
| 715, 450      | 1180, 720     |
| 1180, 720     | 1180, 620     |
| 1280, 720     | 1280, 6200    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![image6](./output_images/3.jpg "Warping Threshold Pics")

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Within function findingLines(.) I search the real lines. I look into the warped picture and setup a histogram for it, to measure where the gradient for the lane lines is high.
I identify 9 nwindows with a margin of 100 and a minimum number of pixels of 50. Every lane is marked with such rectangles, where the higheest peak of the histogram is the starting point of the first rectangle. In further steps, I made that rectangles invisible (switchable in code).
Here, I strongly used the lesson code. This was a very hard part for me.

The polynominal (second order) is the lane within the mean/middle of the rectangle. This is done in function fit_polynominal.

![image7](./output_images/4.jpg "Poly")

As suggested within the project review, for extremly curved lines, in case I find a proper lane, I recenter the next window at is mean position
Code: 
        # If you found > minpix pixels, recenter next window on their mean position
        if len(good_left_inds) > minpix:
            leftx_current = np.int(np.mean(nonzerox[good_left_inds]))
        if len(good_right_inds) > minpix:        
            rightx_current = np.int(np.mean(nonzerox[good_right_inds]))
          

After Review the code went into fit_polynomial(.) function.
            

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I have two functions here. calculateRadius(.) is for my internal tests the the radius in the image.
The used function is calculateRealRadius(.), which calculates the real data sets.
Furthermore, there is a function called offCar(.), which results the distance of the car to the center of the lane.
That information, including the radius of both lane lines are visualized on the top of the picture and video.

After Review:
That code went into the function fit_polynomial(.).
Here everything is computed. Additional sanity checks are called here in the function lane_sanity_check(.)


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Within pipeline(img) function I finally call createImage(.) which places the text (readius, car offset) in the picture. I used the code from the project hints and instructions and modified it. 
For sure, a fresh undistorted image is used to draw on. the two lines with their points are brought into a proper format (x and y coordinates). 
Then the green bar is painted on the street between the lane markers. The already warped image is inversed and combined with the original image.
The following picture shows my final image result.

![image7](./output_images/5.jpg "Result")

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my original video result](./project_video.mp4)
Here's a [link to my resulting video result](./out_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My problems after the review:
It took our to reengineer the source code. I integrated several sanity checks and avoided most of the flickering. Unfortunately, at the end of the video, I was not able to avoid it and stopped after days to invest more. For the the overall calculation it is good and stable.
I introduced further more the search with sliding windows AND the search from prior function.
Everything working stable. All learned lane detection algorithms are used within the code!

How to improve it? The image/video quality for my algorithm is mandatory. My painted lanes are flickering as soon, there are different lights or shadows on the street. Or something (other cars) come close to my warped image zone.
Sure, there can be added more filters and techniques to make that process for lane detection better!

According to the reviewers comment, I add that additional sections:
- Considerations of issues faced: 
-- The pipeline concept is nice, but not a proper and stable concept for a huge car software. I know that, because I work for BMW as a software engineer. It is not very readably and improvable by others to write such "snake source code". 
-- The computation time of that pipeline is still high. It has to compute every single frame by frame from scratch!

- Possible improvements to the pipeline: 
-- The pipeline (of mine) could be structured more and possibly a real software framework could be used. It would be easier, to have all the possible filters a.s.o. already implemented as final tested function. 
-- Another point are the repetetive calculations which could be optimized to save computation power (and energy). 
-- You could use more information from frame to frame for much better prediction and solve suddenly hard edged curves problems.
-- you can reduce the are of interest within a picture to improve the computation times
-- the algorithm could use much more provious polynoms to smooth the detection of lanes on the street.

- Scenarios where the pipeline might fail: 
-- Again, there are problems in case of differing image quality. 
-- In case the street surface changes. 
-- In case there is shadow on the the road or the lane lines are simply missing...
