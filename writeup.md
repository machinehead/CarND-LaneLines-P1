# **Finding Lane Lines on the Road** 


The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


## 1. Reflection

My pipeline consists of 5 steps.

### Color filter

First, I applied a color filter. The color filter detects white and yellow lines. It works in HSV color space in order to achieve that, because HSV made it easier for me to define yellow. 

The yellow lines are detected using a narrow band filter on Hue, a high-pass filter on Saturation and an even higher pass filter on Value.

The white lines are detected by constraining Saturation to be low and Value to be high, with no restrictions on Hue.

The reason to do that was supporting the "challenge" video. In that video there's a segment where the contrast between the yellow line and asphalt becomes so low that the gradient-based Canny filter fails to detect edges properly. However, the yellow line looks pretty distinctive in HSV channels.

In order to make debugging easier, I added four frames from the challenge video and a color picker image to the test images set.

The test frames were identified by dumping all the frames of the (post-processed) video as individual JPG files and identifying the ones where the initial version of line detector was doing a particularly bad job; then dumping all the original frames again to obtain the raw images.

### Grayscale

Second, the color-filtered image is converted to grayscale. There's nothing special about it.

### Canny filter

Third, a Canny filter detects edges. It feels like the thresholds here are lower than the ones given in video lectures, because of the effects of color filter.

### Region of interest

The region of interest is selected by applying a polygonal mask to the edges image. The mask was identified by applying a series of probe masks to original test images and making sure all necessary lane lines are included.

### Hough transform

I spent lots of time figuring out the right set of parameters for Hough transform, and yet I'm never happy with them.

The reason why I spent much time here is that originally I was not using the region of interest filter, so I was trying to find parameters that reduce the overall number of noisy lines across the image. This doesn't allow to set the value of MAX LINE GAP high, so I was stuck trying to make small values of MAX LINE GAP work.

### Drawing lines

I plotted a set of histograms that show the density of slopes and intercepts of the lines identified with Hough transform.

The histograms, quite expectedly, revealed that there are two sets of lines, and also allowed to constrain the parameters of lines to reduce noise.

So, the line plotting function was changed in a way that:

* First, all lines are identified, filtered by constraints on parameters and split into two buckets - left lines and right lines
* Second, a weighted average of their parameters (y=mx+b) are obtained; I'm using line length squared as the weight here. I suppose this makes the averages to depend more on longer lines and this probably makes the averages more stable.
* I use two values on the Y axis: the total size of the image and the upper limit of region of interest, as well as the averaged intercept of detected lines with the bottom edge of the image to predict the corresponding X coordinates of resulting line ends.
* After all X and Y values are determined, two lines are simply plotted between these coordinates.

## 2. Potential shortcomings

One potential shortcoming would be what would happen when the code is applied to videos that make the lane line color change, for example because of:

* Different paint used for lines (blue?)
* Other painted marks on the roads
* Different asphalt color
* Weather and light conditions

In such cases the color filter will start working incorrectly and the entire pipeline will produce wrong results.

Another shortcoming could be if the car position changes notably relative to the lane lines, since the lines taken into consideration are constrained by their slope parameter.

Actually, I assume there are lots and lots of other shortcomings here, coming from the fact that this is a very basic program that relies on  a set of parameters that has been very carefully tuned to work well on 3 rather similar videos. Any deviation from the original setup will cause the program to report wrong results.

## 3. Possible improvements

### Processing as a video stream

A possible improvement would be to make frame processing not independent. For example, there should exist an algorithm that allows to determine how much one image is shifted relative to another; given the shift value, the predictions from previous frame(s) can also be shifted and then averaged together with the current frame. If there were more details available about the physical model of the car and the camera, something like a Kalman filter could be employed to reduce noise, too.

### Parameter tuning

Another potential improvement could be to come up with a way to automatically tune the pipeline parameters. My current parameters were found by eyeballing a set of image pairs, but I imagine this is not the most efficient way to do it, and it probably produces suboptimal results.

One way to do it could be by using some sort of markup - images with lines explicitly labeled. However, obtaining this markup might be pretty expensive.

Another way could be to have some sort of iterative procedure that tries to reduce the variances of the parameters of the lines identified by the Hough transform, while not decreasing the number of lines. Or, vice versa, increase the number of identified lines while not increasing variances. This should mean that overall confidence for the resulting predictions increases. So if this can be converted into a scoring function, then this function could be used to optimize pipeline parameters for the best score.
