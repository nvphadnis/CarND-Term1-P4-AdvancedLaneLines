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

1) **Color and gradient transforms (cell 4):**

The *image_transform* function takes a colored image and various image transform thresholds as inputs. The thresholds were used to convert images from a range of 0 to 255 pixels to binary images focused on the areas of interest i.e. lane lines. I experimented with the Sobel X and magnitude gradient transforms alongside using binary images of the R (from RGB) and S (from HLS) color channels. The lectures demonstrated quite well how the Sobel X gradient was good at identifying lane lines that are almost vertical in an image and hence have a pixel gradient along the image length. The magnitude gradient transforms seemed to intensify edges although it was not as good at isolating just the lane lines. The lectures also demonstrated the R and S channels to be better than others at detecting lane lines. Cell 4 has codes to generate binary images for each of these transformation techniques.

The [Sobel X OR S channel] combined binary image generated a good starting point for all test images. After playing around with their thresholds, I generated a video that was good for the most part but messed up detection over shadows. It also detected a slightly faded line along the road as the lane instead of the real lane line.  Using a [Sobel X OR S channel OR magnitude gradient] as well as a [(Sobel X OR S channel) AND magnitude gradient] combined binary seemed to increase shakiness of the lane detected over the video. I could tune the thresholds either for the light or shadowed patches on the road. All the random pixels around the lanes seemed to be interfering with the lane detection described later. Hence I thought of using the R channel binary which seemed to detect white lanes well in general and yellow lanes over normal to dark patches of the road. Using its binary as an AND i.e. using a **[(Sobel X OR S channel) AND R channel] combined binary** was most effective since it detected the few lane pixels confirmed both by the R and S channels, and eliminated all other pixels. The lack of lane pixels seemed wrong at first but the lane detection method worked because the only pixels left belonged to the lanes.

Image

Cell 5 plots many more test images than provided by the repository. After the first few iterations, I handpicked the frames in [project_video.mp4]() that did not have their lanes detected correctly and added them to the list of test images.

2) **Perspective transform (cell 6):**

The perspective_transform function was used with straight_lines1.jpg as input to calculate perspective and inverse perspective transform matrices using the cv2.getPerspectiveTransform function. These matrices were used throughout the video processing. The idea behind using straight lane lines was that the transformed image would have near vertical lane lines. This was a convenient check on the accuracy of the *src* and *dst* points used in the *cv2.warpPerspective function*.

Source image points (src)	Destination image points (dst)
(200, 720)	(320, 720)
(1130, 720)	(960, 720)
(695, 460)	(960, 0)
(585, 460)	(320, 0)


