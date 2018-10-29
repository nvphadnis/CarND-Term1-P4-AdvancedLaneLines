# CarND-Term1-P4-AdvancedLaneLines
Self-Driving Car Engineer Nanodegree Program: Term 1 Project 4

## Introduction

The goal of this project was to identify a drivable region in front of a car using image processing algorithms. While the first project was capable of detecting lanes on a straight road, it failed if the road curved even slightly. This project extends the scope to cover all kinds of roads with lane lines. The image and video pipeline, and required functions defined by me are all included in [Advanced_Lane_Lines.ipynb](https://github.com/nvphadnis/CarND-Term1-P4-AdvancedLaneLines/blob/master/Advanced_Lane_Lines.ipynb). I will be referencing cells from this notebook along the way.

## Camera Calibration

The 20 chess board images were used for calibrating the camera. The *cv2.findChessboardCorners* function was used to create a list of image points for a corresponding list of known object points on a flat chessboard with 9x6 corners (cell 1). These two lists were then used with the *cv2.calibrateCamera* function to generate a calibration matrix *mtx* and distortion coefficients *dist* correcting for radial distortion induced by the camera lens (cell 2).

Image

Cells 2 and 3 undistort two test images that are used throughout the notebook to demonstrate all functions in the pipeline.

Image
Image

## Image Pipeline

The image pipeline consists of the following steps.

**1) Color and gradient transforms (cell 4):**

The *image_transform* function takes a colored image and various image transform thresholds as inputs. The thresholds were used to convert images from a range of 0 to 255 pixels to binary images focused on the areas of interest i.e. lane lines. I experimented with the Sobel X and magnitude gradient transforms alongside using binary images of the R (from RGB) and S (from HLS) color channels. The lectures demonstrated quite well how the Sobel X gradient was good at identifying lane lines that are almost vertical in an image and hence have a pixel gradient along the image length. The magnitude gradient transforms seemed to intensify edges although it was not as good at isolating just the lane lines. The lectures also demonstrated the R and S channels to be better than others at detecting lane lines. Cell 4 has codes to generate binary images for each of these transformation techniques.

The [Sobel X OR S channel] combined binary image generated a good starting point for all test images. After playing around with their thresholds, I generated a video that was good for the most part but messed up detection over shadows. It also detected a slightly faded line along the road as the lane instead of the real lane line.  Using a [Sobel X OR S channel OR magnitude gradient] as well as a [(Sobel X OR S channel) AND magnitude gradient] combined binary seemed to increase shakiness of the lane detected over the video. I could tune the thresholds either for the light or shadowed patches on the road. All the random pixels around the lanes seemed to be interfering with the lane detection described later. Hence I thought of using the R channel binary which seemed to detect white lanes well in general and yellow lanes over normal to dark patches of the road. Using its binary as an AND i.e. using a **[(Sobel X OR S channel) AND R channel] combined binary** was most effective since it detected the few lane pixels confirmed both by the R and S channels, and eliminated all other pixels. The lack of lane pixels seemed wrong at first but the lane detection method worked because the only pixels left belonged to the lanes.

Image

Cell 5 plots many more test images than provided by the repository. After the first few iterations, I handpicked the frames in [project_video.mp4](https://github.com/nvphadnis/CarND-Term1-P4-AdvancedLaneLines/blob/master/project_video.mp4) that did not have their lanes detected correctly and added them to the list of test images.

**2) Perspective transform (cell 6):**

The perspective_transform function was used with straight_lines1.jpg as input to calculate perspective and inverse perspective transform matrices using the cv2.getPerspectiveTransform function. These matrices were used throughout the video processing. The idea behind using straight lane lines was that the transformed image would have near vertical lane lines. This was a convenient check on the accuracy of the *src* and *dst* points used in the *cv2.warpPerspective function*.

Source image points (src)	| Destination image points (dst)
---- | ----
(200, 720) |	(320, 720)
(1130, 720) |	(960, 720)
(695, 460) |	(960, 0)
(585, 460)	| (320, 0)

Image

Cells 7 and 8 demonstrate the perspective transformation on additional test images.

**3) Finding lane lines (cell 9):**

The *find_lanes* function borrowed the code provided in the lectures to identify and best fit lane pixels into a continuous lane line in the birds-eye (warped) view of an image. The function took a histogram along the horizontal axis of the input image and identified the peaks as a dense column of white pixels hence a good starting point around which the lane line might be present. It then drew a box around that point and divided the image into 9 equal horizontal windows, each with the same height as that box. Two lists, one for each lane line, were created that had coordinates for white pixels within each box of each window, swept bottom to top. These lists of points were passed into a quadratic fit function to obtain an equation for each lane.

Cell 10 visualizes the result of this function on a warped image with curved lanes. The red and blue patches are the pixels identified as belonging to lanes. The yellow lines are the best fit curves. The green boxes are part of the sliding windows.

Image

Cell 11 contains the *find_lanes_again* function that could be used for subsequent images once the sliding window identified lanes in one image in the video. Cell 12 visualizes its result. I did not use this function in the final pipeline since the results were satisfactory without it.

**4) Calculating radii of curvature (cells 13 and 14):**

The radii of curvatures for both lanes were calculated using two methods.

Cell 13 contains the method described in the lectures. Ratios of 30m/720pixels along the y-direction and 3.7m/700pixels along the x-direction were used. Radii were evaluated at the bottom pixel of each lane.
Cell 14, containing the method I eventually used, calculates lane radius by selecting 3 points on the lane and passing a circle through them using the self-defined function *find_3p_circle_radius*. The 3 points I chose were the top, center and bottom lane pixels along the y-dimension (0 to 720 pixels).

**5) Drawing back on original image (cell 15):**

This sample code took the warped image, drew the quadratic fit lane lines and colored the area between them green. It then performed an inverse perspective transform and superimposed it on the undistorted color image.

Image

Cell 16 contains all steps described above in one function called *pipeline*. Some information that needed to be printed on each image was also calculated in this cell.
- Camera position: This is the difference (in m) between the image center and the average of the mean x-positions of pixels belonging to the left and right lanes.
- Steering angle: The *find_3p_circle_radius* was used to calculate radii of curvatures which were used to calculate individual steering angles based on each lane. The final steering angle was assumed to be the sum of both values.
- Turning radius: The turning radius was assumed to be the average of both radii of curvatures.
Cell 17 tested this pipeline on all test images with success.

Image

## Video Pipeline

Cell 18 used the moviepy library to import [project_video.mp4](https://github.com/nvphadnis/CarND-Term1-P4-AdvancedLaneLines/blob/master/project_video.mp4), execute the pipeline function on each frame and generate [project_video_out.mp4](https://github.com/nvphadnis/CarND-Term1-P4-AdvancedLaneLines/blob/master/project_video_out.mp4). Lane identification is fairly steady and accurate throughout the video.

The *image_transform* function structure and tuning took the longest since it had to work on all kinds of images present in the sample video; straight and curved lanes, dark and bright patches, 
tree shadows on the road and different shades of grey between the lane lines that occasionally tricked my pipeline into detecting the wrong lane line. Using the R channel to clean up the combined binary image was the final improvement. It also took me a while to understand that once the perspective and inverse perspective transform matrices were calculated, they need to be used throughout. Hence, using another input video would require recalculating them with new *src* and *dst* points on an image containing straight lanes. That may be one reason why I was not able to run the challenge videos through my pipeline. The *image_transform* function would also fail under different lighting conditions or lanes of a different color (although yellow and white lanes are fairly ubiquitous). Lanes cutting into each other in construction zones may also pose a challenge to the *find_lanes* function.
