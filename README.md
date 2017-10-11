# **Finding Lane Lines on the Road** 

## Cory Pruce

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline laid out can be segmented into 6 parts, including writing the image to persistent storage:

```python

def run(self, image):
        h, w, _d = image.shape
        polygon_vertices = self.get_lanes_polygon(h, w)
        blur_gray = gaussian_blur(grayscale(image), self.kernel_size)
        edges_image = canny(blur_gray, self.canny_low, self.canny_high)
        masked_image = region_of_interest(edges_image, polygon_vertices)
        lanes_image = self.get_lanes_image(masked_image, image, polygon_vertices)
        self.write_img(lanes_image)
        return lanes_image

```

Following the quizzes and a forum post, I first converted the image to grayscale, then blurring that image by sending through a Gaussian distribution. From there, I extract edges using the canny function with thresholds of 140 for lower and 145 for higher. Constrained to an adjustable trapezoid, I get the image with lanes drawn through the hough function and draw lines.    

After a bit of a struggle trying to get the lane lines right using `sohcahtoa`, I took to the forums for inspiration (and help with getting the video to update). There I found simple uses of the `polyfit`/`poly1d` functions to find the line that best fits the given points. At this point, I already had a slope threshold so creating good lane lists was trivial. Upon acheiving a "working" solution, I noticed that the lanes had a lot of volatility (bounced off the line). Seeing another forum post, I implemented a small buffer of "memory" that saves up to `eviction_threshold` previous lane pairs. This smoothed the videos out incredibly! :) 


### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when some object enters into the picture, literally! The given demonstrations assume an open road ahead. However, lines are not always perfect, objects like cars could be in front, and curves/angles/lighting can frequentely change.

Another shortcoming could be granularity. These lanes take the form of lines. However, lane "lines" ahead are usually not perfectly aligned with the orientation of the vehicle. Acheiving some flexibility in the "line" would probably really coincide with something like path-planning ;)

Addressing the optional challenge, I believe points being picked up on the left are skewing the lane line. I've tried removing outliers and choosing the furthest left best-fit line out of 3-5 rounds but each `polyfit` was seeded the same. I tried a few window sizes and shuffling. My guess is the most robust approach is random sampling `n` times to get the true best-fit line (I was going by highest `x1` value).


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to automate this even more with a convnet! The problem could be structured as either a supervised or unsupervised learning problem and would get the granularity issue down with enough data!

Another potential improvement could be to train on images with less-than-ideal circumstances. For example, images that contain tunnels, bridges, boats, other objects, road paint, similar looking objects (like a stop-sign bumper sticker). Maybe this last one is a bit more than just lane detection!
