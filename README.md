# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of five steps. 

- In the first step, I converted the image to grayscale.
- Next, I applied a gaussian blur to the grayscale image with a `5x5` kernel.
- Next, I defined a region of interest based on the location of lane lines in the sample images and cropped out the ROI. This led to a much smaller image which predominantly contained the region with the lane lines.
- In the fourth step, I performed canny edge detection with a low threshold of `100` and a high threshold of `255` to detect the edges around lane lines in the ROI.
- Finally, filtered the detected lines by hough transform and drew the lane lines in the `draw_lines` function. Since this function is at the heart of this assignment, it is described in detail next.

#### Description of `draw_lines` function:
This function receives the following arguments:
- Original image: `img`
- Lines detected by hough transform: `lines`
- A weighted average of co-ordinates of left lane found in previous frames. `llpoints`
- A weighted average of co-ordinates of right lane found in previous frames. `rlpoints`
- A list of slopes of the boundary of roi: `badslopes`.
- Color and thickness of the lines drawn

The outputs are the following:
- A weighted average of co-ordinates of left lane found from previous frames and this frame. `llpoints`
- A weighted average of co-ordinates of right lane found from previous frames and this frame. `rlpoints`

##### Inner working:
My implementation rests on two important facts:
- First, I observed that generally, the left and right lanes had opposite slopes in the videos.
- Second, I wanted to be able to use the fact that the lanes lines do not suddenly change directions. Thus, information from previous frames can be used to stabilize the estimate of the slope of the two lane lines.

The function begins by sorting the lines in ascending order of lengths, and calculates the slope of each line. The ROI we have defined can lead to spurious line detections along the edges of the ROI which can confuse the algorithm to treat the edges of the ROI as lane lines. To avoid such false positives, we filter out all the lines which have slopes close enough to the ROI boundary slopes with a lambda function called `notbadslope`. Once these spurious detections have been removed, the two largest lines with positive and negative slopes are identified as the left and right lane lines. The lines of the previous frames are given `10%` weight in the calculation of estimation of the true lane lines for this frame whereas the lane lines detected in this frame are given a weight of `90%`. Tuning this ratio was important to get the processing pipeline to perform well.

### 2. Identify potential shortcomings with your current pipeline

This being classical computer vision, there are several potential downsides to the current processing pipeline:

- This pipeline explicitly assumes a certain region of interest which is supposed to contain the lane lines. If the car is not in the center of the lane, this assumption will not be valid and the analysis would fail.
- This pipeline also assumes that the lane lines are linear. It is commonly observed that lane lines are curved along roads. Thus, this pipeline would not perform well for those cases.
- The detection of lines by hough transform is susceptible to garbage on the road. In the `solidYellowLeft.mp4` file, it is seen that the pipeline loses track of the lane line a couple of times when there is some garbage on the road close to the lane lines.


### 3. Suggest possible improvements to your pipeline

- We could possibly do away with the explicit specification of an ROI if we perform color conversion and transform the image into an HSV format. The lane lines have a uniform color and could be extracted out by clustering algorithms (k-means or GMMs). This would allow detection in cases where the car is not driving at the center of the road.
- Another possible improvement is to fit curved lines to the detected edges to detect curved lane lines. This would also require a mechanism to filter out outlier and noise using RANSAC or some other outlier detection method.
- Morphological opening of detected edges could help us to get rid of outliers occuring due to garbage on the road. This is a very commonly used processing step in computer vision.

### Short note

In order to utilize the information about lane line directions from the previous frames, I have re-implemented the video processing block. The default video processing calls provided in the notebook did not allow for sharing varialbes across calls to the `draw_lines` function.