#**Finding Lane Lines on the Road** 

##By Heron Ordonez

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Use the same pipeline to process images and videos

[image1]: ./test_images/processed_solidYellowCurve2.jpg "Sample Processed Image"
[image2]: ./writeup/pipeline_process.jpeg "Transformation of the image through the pipeline"
[image3]: ./writeup/gray.jpg "Image transformed to grayscale"
[image4]: ./writeup/contrast_adjust.jpg "Contrast adjusted image"
[image5]: ./writeup/canny_img.jpg "Canny transform for edge detection"
[image6]: ./writeup/masked_img.jpg "Region of interest mask applied to Canny transformed image"
[image7]: ./writeup/hough_img.jpg "Hough lines plotted"
[image8]: ./writeup/classified_img.jpg "Resulting lines from averaging the extended Hough lines to the edges of the region of interest"
[image9]: ./writeup/overlay.jpg "Final result"
---

### Reflection

###1. The pipeline is based around the Hough Transform, where multiple points are converted into lines from an image.
The pipeline is basically preparing the image to make the Hough Transform as effective as possible in matching the lane lines and then merging the results into a single line that is then overlaid on the original image.

The process for videos is to first break up the video in individual frames and then process each frame as a static image independently.

![pipeline transformations][image1]

#### The pipeline process
In order to make the Hough Transform effective in an image it needs to first be simplified:
##### first the image needs to be converted into grayscale color space
    This allows the image to be represented in a bidimensional matrix instead of a tridimensional one.
![grayscale][image3]

##### second the contrast of the image needs to be increased
    In preparation for the next step, a higher contrast means for a more steep gradient field across the image.
    This is done by detecting creating a second matrix that corresponds to the image, but with a binary bit that indicates if the value in the image passes a provided threshold or not. This secondary image is then alpha blended into the original image.
    The result is an image where darker areas are darker and brighter areas are darker, increasing the local gradient.
![contrast adjusted][image4]

##### third the noise in the image is reduced using the gaussian blur. This is done to improve the accuracy of the Canny transform.

##### fourth the image is passed through the Canny transform to detect where the local gradient is greater, detecting points where edges are more likely to be present.
![canny transform][image5]

##### fifth an image mask is applied
    leaving information only in a relevant area of interest. This step can not be applied before the canny transform to reduce the information that has to be processed,  this is because the gradient between the mask and the rest of the image would be detected as an edge by the Canny transform.
![masked image][image6]

##### sixth The Hough transform is used
    from the image, resulting in a list of (x1, y1, x2, y2) points representing start and end points of lines.
![hough lines][image7]

##### seventh the Hough Lines are passed to a detection function
    that simplifies the lines in a few steps:
       -Line slopes are calculated and if the slope is not a real number the line is removed form the set.
       -Lines are extended to the approximate edges of the relevant area mask, using the single point and slope equation.
           The form of this new lines is in the form of (x3, y3, x4, y4) where x3 and x4 represent the value where the edges of the area of interest will be.
           In this way, the only values that we have now are y3 and y4, representing where the Hough lines would intersect the edges of the area of interest.
       -Lines are sorted by the sign of their slope, as lines along the same lane edge will have to have the same approximate slope.
       -Finally, values for each line group are calculated, since x3 and x4 are always given they will always retain their values, but he values of y3 will be averaged to find one point at x3 and y4 will be averaged to find a second point at x4.
       -using (x3, y3avt, x4, y4avg) for each group we now have 2 lines that most likely match the edges of the lane.
![average lines][image8]

###### eight a new image is drawn with these new lines in different colors and blended over the original image for the final result

![final result][image9]


###2. Some shortcomings with the current implementatipon

There are many shortcomings in this pipeline, as can be seen from the results in the videos:
- Lines jump from one frame to the next when contrast changes
- Different shades of the same color can throw the line off
- Sometimes only one edge is detected

###3. Possible improvements

A few things to try to improve accuracy:
-Use the previous frames as reference for where the lane edge is and smooth over videos.
-Use a different approximation for the edges instead of a simple average
-Use color selection before the transformation to allow the edges of the lines to be clearer.
