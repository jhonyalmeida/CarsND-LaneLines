# **Finding Lane Lines on the Road** 


[//]: # (Image References)
[image1]: ./writeup_images/step1.png "Step 1. Grayscale"
[image2]: ./writeup_images/step2.png "Step 2. Gaussian Blur"
[image3]: ./writeup_images/step3.png "Step 3. Canny Edges"
[image4]: ./writeup_images/step4.png "Step 4. Region of Interest"
[image5]: ./writeup_images/step5_raw.png "Step 5-a. Hough Lines"
[image6]: ./writeup_images/step5_single.png "Step 5-b. Single Lines"
[image7]: ./writeup_images/final_result.png "Step 6. Final Result"

---

## Pipeline description

The presented pipeline to find the lane lines on images and videos consists of six steps:

#### 1. Grayscale

First, the input image is converted to grayscale, since next steps do not require (or even do not support) working with color channels.

![alt text][image1]

#### 2. Gaussian blur

To improve the edge detection results in next step, reducing some noise, a gaussian blur is applied, using a 5x5 kernel.

![alt text][image2]

#### 3. Canny edges

The third step is to detect the image edges using Canny function. After testing many threshold values, the chosen ones were 50 and 80, that fitted most cases.

![alt text][image3]

#### 4. Region of interest

To discard unwanted edges, a mask is applied to delimit the region of interest, where the lane lines are located. The shape of the mask is a trapezoid, whose points are calculated proportionally to the image dimensions. The result of this step is shown below.

![alt text][image4]

#### 5. Lines detection

In this step, the Hough Lines function is applied to the region of interest to find the rough lines segments. Like the Canny step, the parameters were tested with several different values, the chosen ones being 40 for the threshold, 40 for minimun line length, and 70 for max gap between line segments. The results shown below were obtained with the unmodified draw_lines function.

![alt text][image5]

In order to draw a single line on the left and right lanes, the draw_lines function was modified. Firstly, the identified line segments were divided in two groups, left and right ones, accordindly to their slopes (negative and positive). Parallel to this dividing, each analysed line segment is discarded if it don't meet some constraints:

1. The angle of the line must be between 20 to 70 degrees, so horizonal lines, for example, are ignored;

2. The  values of x1 and x2 must be located on the correspont side of the image, so left line segments found in the right side are ignored, and vice versa;

3. The line slope must be in the estimated interval, whose was defined 0 to 0.85 (or 0 to -0.85) based on most lane lines calculated slopes.

With the line segments separated and filtered, the mean values of x1, x2, y1 and y2 are calculated for the left and right lines, separately. Using these mean values, the numpy's polyfit function is applied to find the intercept and slope of the left and right lines.

Having the intercept and slope of both lines, and setting arbitrary values for y1 and y2 for extrapolated lines, the x1 and x2 values are calculated by (y - intercept) / slope. Finally, the full estimated extent of the lane lines can be drawn as below:
 
![alt text][image6]

#### 6. Final Result - Draw the lines

Finally, the estimated full lines are combined with the input image, producing the final result, with red annotated lane lines. Further results of the pipeline applied to videos can be found in the test_videos_output directory.

![alt text][image7]

## Potential shortcomings

When applied to videos, in some individual frames, the lane lines could simply not be detected, specially the yellow lines in the optional challenge video, where there are strong sunlight in some frames (between 4 and 5 seconds), raising the image brightness. 

With some parameter tweaks in the Canny and Hough functions, the high brightness problem was roughly solved, but there are no guarantees that in other cases, with even stronger sunlight, this could not happen again.

Furthermore, shadows cast by trees and other objects surrounding the highway can partially occlude lanes segments, also difficulting the lines detection. The presented pipeline did not perform so well in these cases, causing some frames to show imprecise lanes in the challenge video.

Lastly, the images and videos used for testing do not contain objects, like other cars, in the space between the lines, whose certainly would produce a lot of undesirable noisy lines. The pipeline do not adress that in any manner.


## Possible improvements

A possible major improvement to the presented pipeline could be to apply some tracking algorithm to the initially detected lines, instead of detecting they from zero in each frame. This could compensate frames where the lines are not detected due some noise caused by shadows or sunlight, simply estimating the line position based on previous frames.

Another potential improvement could be to change the mask of the region of interest to ignore most of the space between the lines, so objects like other cars in the front would not disturb the line detection significantly.

Also, maybe a function to raise the contrast of the image could improve the edges and lines detection steps in frames with high brightness.

Last, to find better parameters for the pipeline steps, including mask boundaries and Canny and Hough thresholds, a method like a genetic algorithm could be employed.
