# Project 1 - Finding Lane Lines

## Overview: Finding Lane Lines on the Road
The goal of this project is to accurately and consistently identify lane lines on the road. The steps towards completing this project include the following:
* Develop a pipeline that finds lane lines on the road
* Reflect on project accomplishments and future improvements

[//]: # (Image References)
[image1]: ./test_images_output/POLYsolidWhiteCurve_1.gray.jpg “Grayscale”
[image2]: ./test_images_output/POLYsolidWhiteCurve_2.blur.jpg “Gaussian Blur”
[image3]: ./test_images_output/POLYsolidWhiteCurve_3.canny.jpg “Canny Edge”
[image4]: ./test_images_output/POLYsolidWhiteCurve_4.masked.jpg “Masked Edge”
[image5]: ./test_images_output/POLYsolidWhiteCurve_5.lines.jpg “Extrapolated Lines”
[image6]: ./test_images_output/POLYsolidWhiteCurve_6.OUTPUT.jpg “Output”
[image7]: ./extra/pipeline.jpg “Pipeline Diagram”
[image8]: ./test_images_output/ALL3solidYellowCurve2_6.OUTPUT.jpg “Overlay sYC2”
[image9]: ./test_images_output/ALL3whiteCarLaneSwitch_6.OUTPUT.jpg “Overlay wCLS”
[image10]: ./test_images_output/ALL3solidYellowLeft_6.OUTPUT.jpg “Overlay sYL”
[image11]: ./test_images_output/ALL3solidWhiteCurve_6.OUTPUT.jpg “Overlay sWC”


---

## Reflection

### 1. Developing the pipeline

My pipeline consisted of 5 steps:

#### Step 1: *Convert images to grayscale*
Grayscaling an image helps to efficiently calculate gradients on an image.

`grayscale(...)` is used to call `cv2.cvtColor(...)`

![alt text][image1]

#### Step 2: *Apply Gaussian smoothing*
Smoothing helps to reduce the occurrence of stray large gradients on an image.

`gaussian_blur(...)` is used to call `cv2.GaussianBlur(...)`

![alt text][image2]

#### Step 3: *Apply Canny Edge Detection*
Canny edge detection provides a gradient map of the input image.

`canny(...)` applies `cv2.Canny(...)`

![alt text][image3]

#### Step 4: *Apply Hough transform after defining region of interest*
Hough transform allows us to determine what lines are appropriate lane indicators.

`hough_lines(...)` utilizes a combination of `cv2.HoughLinesP(...)` and `draw_lines(...)`

`draw_lines(...)` uses `cv2.line(...)` to create an image with lines only

![alt text][image4]

#### Step 5: *Compound identified lane lines onto original image after extrapolating Hough lines*
This allows us to visualize the performance of the extrapolated lines on the original image.

`weighted_img(...)` uses `cv2.addWeighted(...)` to layer the identified lane lines onto the lane image

![alt text][image6]

#### Pipeline modifications

![alt text][image7]

`draw_lines(...)` is a function used in **Step 4** that takes an input `img` and array `lines` with shape `(m, 1, 4)` where `m` is the number of lines (points tracked using `x1`, `y1`, `x2`, `y2`)

Note that our line equation can be represented as follows:
```
        y = m * x + b
```
which can then be used to derive `m` and `b` as follows:
```
        m = (y2 - y1) / (x2 - x1)
        b = y1 - m * x1
```
**Mean method:** I took the `lines` array and separated them into new arrays representing the left and right halves of the image.  I then calculated (`m`, `b`) pairs for each half and took the **average** values to acquire (`m_avg`, `b_avg`) for each `line` with a slope of magnitude greater than 0.5.  The extrapolated lane lines were then created by marking a blank image array using the mask boundaries and line equations, yielding the final result.

**Median method:** Similar to the previously mentioned **means method**, I separated `lines` into two arrays (for left and right image halves) and calculated the (`m`, `b`) pairs for each half. But instead of taking the average values, I took the **median** values of each `line` with a slope of magnitude greater than 0.5. The extrapolated lane lines were then created by marking a blank image array using the mask boundaries and line equations, yielding the final result.

**Polynomial fit method:** This method uses a different approach to the prior **means method** and **median method**. In this case, instead of calculating values from sorted and filtered (`m`, `b`) pairs, I used `numpy.polyfit()` to determine the best fit, single degree polynomial (yields a best-fit coefficient vector (`m`, `b`)). The extrapolated lane lines were then created by marking a blank image array using the mask boundaries and line equations, yielding the final result.

### 2. Identifying potential shortcomings
While this pipeline may reach a certain point where it is practical for usage, there are still various scenarios in which some methods may perform poorly or perform better than others.

**Mean method:** This is the method suggested in the default `draw_lines(...)` function text. By taking the mean `m` and `b` parameters, we can build a fairly representative line for each of the lanes (left/right). However, any stray lines (not part of the lane edge) contributes error to the mean calculations of `m` and `b`. In addition to that, since the perspective of our image contains more edge markings towards larger `y` values, the mean calculations of `m` and `b` will tend towards the `m` and `b` values of edgers on the lower portion of the image.

**Median method:** To counter the bias introduced by stray edges and larger `y` edges using the **mean method**, I decided to try this median approach. By taking the median `m` and `b` parameters, the outer `m` and `b` values in each line array (left/right) are omitted. However, when testing on the provided test images, it was apparent that counting on a single `m`,`b` reference would not be consistent for all cases, since any tiny edge error would result in a huge variance in extrapolated line error.

**Polynomial fit method:** This approach tries to address issues from both the **mean method** and **median method** with the `numpy.polyfit(...)` function. To do so, we utilize all of the `line` data points in the calculation of (`m`, `b`). One advantage with this is that the resultant (`m`, `b`) is directly calculated from the image edge points instead of from derived (`m`, `b`) pairs of those edge points. However, if the input image contains a non-linear set of data points (curved lane or stray edges), the resultant lines may still end up being inaccurate.

![alt text][image8]	![alt text][image9]
![alt text][image10]	![alt text][image11]

Note that (MEAN, MEDIAN, and POLY) are characterized with (red, green, blue) lines, respectively.
In the case there there is complete overlap, the line should be white RGB=`[255 255 255]` 

### 3. Suggesting possible improvements to the pipeline
Based on the development of this pipeline, I have come to the notion that **Steps 1, 2, and 3** are fairly standard. Thus the improvements would likely take place with **Steps 4 and 5**. After watching the pipeline performance on the provided test videos, the largest margin of improvement appears to be with the `mask`, `Hough Transform`, and `draw_lines(...)`.

#### Region of interest, mask
Although the camera sits in a fixed location on the vehicle, the image objects and lane locations will not be as consistent. Too small of a mask may result in insufficient data on a lane merge or on a road with larger lanes. Too big of a mask may result in data corruption (as seen in the challenge test video). Since these are both valid considerations, it is likely that our lane detection pipeline will require the use of supplemental lane detection techniques to dynamically resolve the best-fit region of interest mask.

#### hough_lines(...) 
The `hough_lines(...)` function provides an adjustable method to determine real edges. However, roads aren't always as well-maintained as the ones shown in the test images. While it may be beneficial to achieve the most optimal `hough_lines(...)` inputs, I foresee that most of the improvement being made to its input image and the embedded `draw_lines(...)` function.

#### draw_lines(...)
All of the aforementioned methods of 	`draw_lines(...)` demonstrate slightly different approaches to create accurate extrapolated lane lines. But clearly, according to the challenge test video, there are many improvements still to be made. Similar to defining the mask, it is not realistic to assume an ideal road condition when extrapolating lines. Specifically with curves, the use of the **polynomial fit method** (with an increased polynomial degree) could yield a more accurate result. Complementary to this technique, data parameters from the actuation of the steering wheel could help us calculate an optimal polynomial degree.


