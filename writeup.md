# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals of this project is the following:
* Make a pipeline that finds lane lines on the road

[//]: # (Image References)

[image1]: ./test_images/solidWhiteRight.jpg "White Test Image"
[image2]: ./test_images/whiteCarLaneSwitch.jpg "Yellow Test Image"
[image3]: ./test_images_output/1greysolidWhiteRight.jpg "White Grey Image"
[image4]: ./test_images_output/1greywhiteCarLaneSwitch.jpg "Yellow Grey Image"
[image5]: ./test_images_output/2blursolidWhiteRight.jpg "White 3x3 Blur Image"
[image6]: ./test_images_output/2blurwhiteCarLaneSwitch.jpg "Yellow 3x3 Blur Image"
[image7]: ./test_images_output/3cannysolidWhiteRight.jpg "White 50-150 threshold Canny Image"
[image8]: ./test_images_output/3cannywhiteCarLaneSwitch.jpg "Yellow 50-150 threshold Canny Image"
[image9]: ./test_images_output/4regionsolidWhiteRight.jpg "White Edges Region Image"
[image10]: ./test_images_output/4regionwhiteCarLaneSwitch.jpg "Yellow Edges Region Image"
[image11]: ./test_images_output/5houghsolidWhiteRight.jpg "White Hough Image"
[image12]: ./test_images_output/5houghwhiteCarLaneSwitch.jpg "Yellow Hough Image"
[image13]: ./test_images_output/solidWhiteRight.jpg "White Lane Line Image"
[image14]: ./test_images_output/whiteCarLaneSwitch.jpg "Yellow Lane Line Image"


---

### Reflection

### 1. Describe the pipeline.

In this project, we used OpenCV (`cv2`) to find lane lines in the road images.

The pipeline consists in the following steps:
- Convert the image into gray scale.
- Blur the gray scale image to remove noise.
- Detect edges wiht the Canny edge detection algorimth.
- Delimit the interest region where find lane lines
- Get line segments with the Hough Transform
- Extrapolate the line segments to map out the full extent of the lane lines
- Display the lane lines in the imagen.

First, we apply this steps to test images to tune algorimth parameters. Then, we apply them to videos.

**Test Images**

|![alt text][image1]|![alt text][image2]|
|-------------------|-------------------|

**Convert to Gray Scale**

The first thing to do is convert the image to a grey scale.
To do this, we can use the method `cv2.cvtColor()` passing in the image and a constat of OpenCV that tell it between what color scales to convert. In this case the constat to use es `COLOR_RGB2GRAY`.

|![alt text][image3]|![alt text][image4]|
|-------------------|-------------------|

**Blur the Gray Scale Image**

Now we want to smooth the image to be able to eliminate the noise that may hinder the following steps.
We are going to use the method `cv2.GaussianBlur()`. Internaly, this method uses a square matrix of odd range, and it is necessary provide it the range value (`3, 5, 7, ...`). The larger the value the more blurred.
For the imgages below, it has been used a 3x3 Matrix.

|![alt text][image5]|![alt text][image6]|
|-------------------|-------------------|

**Detect Edges wiht the Canny Edge Detection**

We use the Canny edge detection to find the egdes of the lane lines.
This algorimth works finding the intensity gradient of the image, i.e. detecting changes in intensity between pixels.

We use the method `cv2.Canny()`. We have to provide two threshold values, `low_threshold` and `high_threshold`.
Any detected edges with intensity gradient more than `high_threshold` are sure to be edges and those below `low_threshold` are sure to be non-edges, so discarded. Those who lie between these two thresholds are classified edges or non-edges based on their connectivity. If they are connected to "sure-edge" pixels, they are considered to be part of edges.

The images below use values of `50` and `150` as low and high thresholds.

|![alt text][image7]|![alt text][image8]|
|-------------------|-------------------|
**Delimit the Interest Region**

Now that we have an image with only edges, we have to concentrate only on the region of the image that has the edges of lane lines.

To stablish the region of interest, we use the method `cv2.fillPoly`. We pass in it a blank mask of the size of the image, and a list of vertices that delimit ther region and a color to fill the region (white in this case). This method returns the mask of the region which is then applied to the edge's image. The result is a image only with the edges wihtin the region.

|![alt text][image9]| ![alt text][image10]|
|-------------------|---------------------|

**Get Line Segments with the Hough Transform**

We apply the Hough tranform to the edges region in order to get the line segments of the lane lines.

To do that, we use the method `cv2.HoughLinesP()`. We need to especify the following parameters:
- *img*: image of the edges region.
- *rho*: The resolution of the parameter $\rho$ in pixels.
- *theta*: The resolution of the parameter $\theta$ in radians.
- *lines*: A vector that will store the parameters $(x_{start}, y_{start}, x_{end}, y_{end})$ of the detected lines. In Python a numpy array empty.
- *threshold*: The minimum number of intersections to “*detect*” a line.
- *minLineLength*: The minimum number of points that can form a line. Lines with less than this number of points are disregarded
- *maxLineGap*: The maximum gap between two points to be considered in the same line.

This method gives as output a list of the extremes of the detected line segments $(x_{0}, y_{0}, x_{1}, y_{1})$.

These extremes are passen in to the method `cv2.line()` to draw these line segments on a blank mask of the image size.

Images with the result of apply Hough Transfor with parameters *rho = 1*, *theta = 1 radian (np.pi/180)*, *threshold = 35*, *minLineLength = 5*, *maxLineGap = 2*.

|![alt text][image11]|![alt text][image12]|
|--------------------|--------------------|

**Display the lane lines in the imagen**

Finally we merge the original image with the result of applying Hough Transform.

To do this, we use the method `cv2.addWeighted()`.

Images whit lane line segments drawn

|![alt text][image13] | ![alt text][image14]|                      
|---------------------|----------------------|



**Pipeline Apply to a Video Test**

As a video is made up of a sequence of images, known as frames, we can apply this pipeline to a video test.

[Video with lane line segments (solidWhiteRight)](./test_videos_output/)

**Improve the draw lines function**

Up to now, we have drawn the lane line segments onto the road, but what we want is to draw the full extent of the left and right lane lines.

First we have to identify the segments that belong to the left and right lane line. To do this, we calculate the slope of each segment and if it is negative, it belongs to the left line, and otherwise, to the right line.

Then, the segments are averaged (slope and intercep) from each side to build a unique one within the region of interes.

The problem with this solution is that some segments with extreme slopes can affect the slope
average, moving the resulting line from the optimum.

The provided solution has been to define, according to some observations made, an average slope and a range
tolerance (+/-0.6 slopes and 30% tolerance) to be able to exclude segments with extreme slopes.

``` python
left_m_reg = -0.6             # left slope average (fixed)
right_m_reg = 0.6             # right slope average (fixed)
left_m_tol = left_m_reg*0.3   # left slope tolerance
right_m_tol = right_m_reg*0.3 # right slope tolerance
left_segments = []
right_segments = []

for line in lines:
    for x1,y1,x2,y2 in line:
        if x1 == x2:
            continue
        m = (y2-y1)/(x2-x1)
        b = y1 - m*x1
        # Reject slopes out of range
        # left line. y is in the reverse direction
        if m < 0 and (m > left_m_reg + left_m_tol) \
                and (m < left_m_reg - left_m_tol):
            left_segments.append((m,b))
        # right line
        elif m > 0 and (m > right_m_reg - right_m_tol) \
                and (m < right_m_reg + right_m_tol):
            right_segments.append((m,b))
```
We has also defined a line history for the previous frames (its size is parameterizable). Its purpose is double:
- Average the lines of the frame with those of the historical to smooth transitions.
- If a frame has no line, we get the one from the previous frame.

```python
if line is not None:
    m, b = line
    x1 = int((y1 - b)/m)
    x2 = int((y2 - b)/m)
    y1 = int(y1)
    y2 = int(y2)
    frames_lane.append(((x1, y1),(x2, y2)))
else:
    if not frames_lane: # if historic lane lines is empty, return
        return None
    ends = frames_lane.pop()
    x1, y1 = ends[0]
    x2, y2 = ends[1]

    frames_lane.append(((x1, y1),(x2, y2)))
    frames_lane.append(((x1, y1),(x2, y2)))


    lane = np.mean(frames_lane, axis=0, dtype=np.int32)
```
The result is a single, solid line over the left lane line and a single, solid line over the right lane line.

[Video with solid lane lines (solidYellowLeft)](./test_videos_output/)

###**Optional Challenge**

The video of the challenge has some areas in which the lane lines are not clearly distinguished (change of asphalt and
shaded areas) and the Canny algorithm does not detect edges in this area

[Original challenge video (challenge)](./test_videos/)

The implemented pipeline solves partially this problem using the history of frame lines.

A more robust solution is to use a yellow and white filter as the first step of the pipeline. This filter is going
to carry out in HLS scale, that distinguishes better the yellow and white colors in different types of zone.

[HLS scale yellow-white mask video (HLS_Y_W_filter)](./other_videos_output/)

After building the yellow and white mask on HLS color scale it is converted back to RGB scale.

[Yellow-white mask video convert to RGB scale (HLS2RGB_Y_W_filter)](./other_videos_output/)

Then, this mask is passed through all the steps of the original pipeline

[Challenge video wiht lane lines detected (challenge)](./test_videos_output/)
