#**Finding Lane Lines on the Road**

##The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road on a series of images
* Apply the pipeline to video streams

[//]: # (Image References)

[image1]: ./output_images/solidWhiteCurve.jpg "solidWhiteCurve"
[image2]: ./output_images/solidWhiteRight.jpg "solidWhiteRight"
[image3]: ./output_images/solidYellowCurve.jpg "solidYellowCurve"
[image4]: ./output_images/solidYellowCurve2.jpg "solidYellowCurve2"
[image5]: ./output_images/solidYellowLeft.jpg "solidYellowLeft"
[image6]: ./output_images/whiteCarLaneSwitch.jpg "whiteCarLaneSwitch"

---

### Reflection

###1. My pipeline to identify lane lines on road images consisted of 5 main steps. 

##Apply Gaussian Smoothing

First, I apply the grayscale transform which returns an image with only 1 color channel. Then, I apply Gaussian smooothing which suppresses noise and spurious gradients by average. The Canny transform which will be applied in the next steps actually applies Gaussian smoothing internally, but I include it here to get a different result by applying further smoothing with customizable parameters. The kernel_size can be any odd number and a larger kernel_size implies averaging, or smoothing, over a larger area. 

##Apply the Canny transform 

Next, I apply the Canny transform on the grayscale, smoothed image. This algorithm will first detect strong edge pixels above high_threshold and reject pixels below low_threshold. Pixels, with values between the low_threshold and high_threshold will be included as long as they are connected to strong edges. The output edges is a binary image with white pixels tracing the out the detected edges and black everywhere else.

##Apply image mask over a region of interest

Since the Canny transform for edge detection was applied over the whole image, I need to pinpoint the vertices of a polygon region within the image over which I can apply an image mask. My region of interest is an isosceles trapezoid shape that trims the full image down to just the lane that the car is inside. Ideally, the left and right lane line markers should just be inside the left and right trapezoid borders which narrow toward the horizon in the center of the image. I define the 4 vertices of this trapezoid region of interest.

Next, I apply a mask to extract only the lane lines, because there are still some other objects detected around the periphery outside the trapezoid region of interest that aren't lane lines. The mask only keeps the region of the image defined by the polygon formed from the vertices. The rest of the image is set to black.

##Run Hough transform on edge detected image

After that, I apply the Hough transform on the resulting image (Canny edges within a trapezoid shape). The Hough transform is the conversion from image space to Hough space. The characterization of a line in image space is a single point at the position (m (slope), b (y-intercept)) in Hough space. Consequently, a point in image space corresponds to a line in Hough space. Two points in image space corresponds to 2 intersecting lines in Hough Space and the intersection point of 2 lines in Hough space corresponds to a line in image space that passes through both (x1, y1) and (x2, y2).

I use the HoughLinesP in OpenCV to apply the Hough transform on the image which takes several parameters. Rho and theta are the distance and angular resolution of our grid in Hough space. The threshold parameter specifies the minimum number of votes (intersections in a given grid cell) a candidate line needs to have to make it into the output. The min_line_length is the minimum length of a line (in pixels) that I will accept in the output, and max_line_gap is the maximum distance between segments that I will allow to be connected into a single line. The result is a collection of line segments, which hopefully are limited to only the immediate left and right lane markers of the current lane the car is driving in.

##Draw lane lines on image

With all the line segments, I categorize the lines as being a part of the left line or right line based on the slope. Positively sloped lines are left lines and negatively sloped lines are right lines. I calculate the average slope and average y-intercept of all left lines and right lines, respectively. For calculating the average, I only take the last x left line segments and last x right line segments. In this way, the algorithm should be more robust to sudden curves. Then, I compute a single average left line to represent the collection of all left lines extending from the bottom of the image to the middle of the image. This process is repeated to compute a single average right line. Finally, I overlay these lines on top of the road image.

<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/solidWhiteCurve.jpg" width="480">
<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/solidWhiteRight.jpg" width="480">
<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/solidYellowCurve.jpg" width="480">
<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/solidYellowCurve2.jpg" width="480">
<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/solidYellowLeft.jpg" width="480">
<img src="https://github.com/pengjoshua/CarND-LaneLines-P1/blob/master/output_images/whiteCarLaneSwitch.jpg" width="480">

###2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen with differently sized images. My pipeline works well for the solidWhiteRight.mp4 file and solidYellowLeft.mp4 file but fails for the optional challenge.mp4 file that has a different size.

My pipeline would also have problems with different image vantage points and perspectives. Video streams from taken from cameras installed at even slightly different angles and viewpoints of the road would likely create problems for my draw_lines algorithm. 

My pipeline would likely have problems with double dashed or solid line markers on the left or right. These additional lines would be factored into the line average.

My pipeline would have problems with additional road markings inside the current lane such as diamond marking for HOV lanes, right or left turn markings, or any letters as any additional artifacts with strong edges would be considered a line segment.

Similarly, my pipeline would also temporarily have problems if the car ever changed lanes as line segments would shift from being classified as left to right lines and vice versa. I can only imagine what would happen in adverse weather conditions with rain and snow!

###3. Possible improvements to my pipeline

A possible improvement would be to base my trapezoid region of interest based on percentages of the image height and width instead of pixel values.

I could also try to filter out all colors and brightness values that have obvious incorrect dark colors so that the Hough transform will have an easier time finding actual line segments.

My pipeline contained many tunable parameters for gaussian smoothing, Canny transform, and Hough transform. Finding appropriate values for these parameters was very challenging. These parameters are highly dependent on road orientation and camera positioning.
