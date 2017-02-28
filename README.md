#**Finding Lane Lines on the Road** 
![pipeline transformations][image1]
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

### Pipeline Overview
The pipeline is based around the Hough Transform, where multiple points are converted into lines from an image, most of it prepares the image to make the Hough Transform as effective as possible in matching the lane lines and then merging the results into a single line that is then overlaid on the original image.

The process for videos is to first break up the video in individual frames and then process each frame as a static image independently.

#### The pipeline process
##### 1.- The image is converted into grayscale color space
This allows the image to be represented in a bidimensional matrix instead of a tridimensional one.
![grayscale][image3]

##### 2.- the contrast of the image is increased
In preparation for the next step, a higher contrast means for a more steep gradient field across the image.
This is done by detecting creating a second matrix that corresponds to the image, but with a binary bit that indicates if the value in the image passes a provided threshold or not. This secondary image is then alpha blended into the original image.
The result is an image where darker areas are darker and brighter areas are darker, increasing the local gradient.
![contrast adjusted][image4]

##### 3.- Noise reduction
The noise in the image is reduced using the gaussian blur. This is done to improve the accuracy of the Canny transform.

##### 4.- Canny transform
The image is passed through the Canny transform to detect where the local gradient is greater, detecting points where edges are more likely to be present.
![canny transform][image5]

##### 5.- Relevant Area Mask
An image mask is applied leaving information only in a relevant area of interest. This step can not be applied before the canny transform to reduce the information that has to be processed,  this is because the gradient between the mask and the rest of the image would be detected as an edge by the Canny transform.
![masked image][image6]

##### 6.- The Hough transform
    The Hough transform is used on the image, resulting in a list of (x1, y1, x2, y2) points representing start and end points of lines.
![hough lines][image7]

##### 7.- Simplification
The Hough lines are passed to a function that simplifies the lines in a few steps:
- Line slopes are calculated and if the slope is not a real number the line is removed form the set.
- Lines are extended to the approximate edges of the relevant area mask, using the single point and slope equation.
- The form of this new lines is  _(x3, y3, x4, y4)_ where _x3_ and _x4_ represent the value where the edges of the area of interest will be. In this way, the only new values that we have now are _y3_ and _y4_, representing where the Hough lines would intersect the edges of the area of interest.
- Lines are sorted by the sign of their slope, as lines along the same lane edge will have to have the same approximate slope.
- Finally, values for each line group are calculated, since _x3_ and _x4_ are always given they will always retain their values, but he values of y3 will be averaged to find one point at _x3_ and _y4_ will be averaged to find a second point at _x4_.
- Using _(x3, y3<sup>avg</sup>, x4, y4<sup>avg</sup>)_ for each group we now have 2 lines that most likely match the edges of the lane.
![average lines][image8]

###### 8.- Final result
A new image is drawn with the new lines in different colors and blended over the original image for the final result.

![final result][image9]


### Some shortcomings with the current implementation

There are many shortcomings in this pipeline, as can be seen from the results in the videos:
- Lines jump from one frame to the next when contrast changes
- Different shades of the same color can throw the line off
- Sometimes only one edge is detected

### Possible improvements

A few things to try to improve accuracy:
- Use the previous frames as reference for where the lane edge is and smooth over videos.
- Use a different approximation for the edges instead of a simple average
- Use color selection before the transformation to allow the edges of the lines to be clearer.

### Final words
This was a fun project, even when the results are not great for challenging videos. I think that lane detection in this way may not be as safe as possible with other techniques, including more advanced computer vision or deep learning.
