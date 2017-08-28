## Project: Search and Sample Return

---


**The goals / steps of this project are the following:**

**Training / Calibration**

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook).
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands.
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.

[//]: # (Image References)

[image1]: ./output/warped_threshed.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg
[image4]: ./output/warped_threshed_obstacles.jpg
[image5]: ./output/warped_threshed_clipped_obstacles.jpg
[image6]: ./output/threshed_rock.jpg
[image7]: ./output/capture.JPG

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
Instead of modifying the given `color_thresh()` function, I added 3 new functions for the given tasks: `color_bel_thresh()`, `color_bet_thresh()`, and `clip_img()`.

The function `color_bel_thresh(img, rgb_thresh)` does the oppoisite of the `color_thresh()` function. It returns the image with the pixels below the threshold set to 1. To do this, I added a `~` operator in front of the boolean expression. However, since the persepective transformation introduces black pixels for the unmapped regions, these pixels' values are all set to 1 just like the image below.

![alt text][image4]

In order to correct this undesired feature, I introduced `clip_img(img, source, destination)`. This maps a white image with perspective transformation and uses it as a mask. The result of applying this function to the above is below.

![alt text][image5]

Finally, to detect yellowish rocks, I added `color_bet_threshold(img, lb, ub)` function. By manually inspecting the color pixels of the rocks, I found that they were in all in a range between (100, 100, 0) and (255, 255, 50). The `color_bet_threshold()` selects such pixels.

![alt text][image3]
![alt text][image6]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.

The output movie is consisted of 5 sections. The upper row consists of the original image, warped image, and warped & color threshed image(red for the obstacles, blue for the navigable terrains). The lower row has color threshed for rocks(yellow for the rocks) and a worldmap. To make the worldmap, I just used the given functions. I colored red for the obstacles and blue for the navigable terrains. If a pixel was found to be both a navigable terrain and an obstacle, it was deemed navigable. I didn't record the information about rocks on the worldmap.

![alt text][image7]

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

For the `perception_step()`, most of the changes are identical to the ones made for the jupyter notebook. Namely, I added `color_bel_thresh()`, `color_bel_thresh()`, and `clip_img()` for the detection. Because the cumbersome parts of image creation are handled in `supporting_function.py`, I simply added 1 to the world map for each detected navigable terrain/obstacles. However, for rocks, I added only the distant pixel to prohibit the warped pixels to be added. I added `rocks_dists` and `rocks_angles` attribute for `Rover` class to use it for later possible pick-up challenge, but they were not used in this submission.

I left the `decision_step()` as it is for this submission. Basically, it has only two states: forward and stop. Each states has sub-states, which are identified by some threshold values. For each sub-states, corresponding actions or state transititons are executed.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Simulator settings:

  * OS : Windows
  * Screen : 1024x768
  * Graphics quality : Fantastic
  * FPS : 37

Most of the situations were handled with no problem, but there were some caveats. If the rover bumps into an obstacle, it bounces off and makes the warping inaccurate. This situation can be detected by referencing pitch and roll of the rover. Also, the rover sometimes visit the same place again and again. This could have been avoided by making the rover follow through the left wall or somehow keep track of its path.
