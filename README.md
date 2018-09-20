## Project: Search and Sample Return
### The repositary is my modifications of **Search and Sample Return** Modifications mainly in __Rover_Project_Test_Notebook.ipynb__, __perception.py__, and a few lines added in __decision.py__. I will use this README to indicate the tasks in this project and how I address each task. 

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

[image1]: ./output/obstacle_mask.jpg
[image2]: ./output/rock_thresh.png
[image3]: ./output/test_mapping.png
[image4]: ./output/Capture.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### I implement the codes and wite up this README to address the above Rubric for this project.

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

This is the README indicates the rubic :)

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

* Selection of obstacles
I use `mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))` to create obstacles mask.
The mask will be fit to red value of worldmap after coordinations mapping.
```
obs_x, obs_y = rover_coords(mask)
obstacle_x_world, obstacle_y_world = pix_to_world(obs_x, obs_y,rover_xpos,
                                                rover_ypos, rover_yaw, 
                                                data.worldmap.shape[0], scale)
data.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1                                                
```

![alt text][image1]

* Rock samples
I use [inRnage function of opencv](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html) to indentify dark-yellow rocks. Here is the code inside `color_thresh` funtion:
```
# RGB to BGR gor opencv process
bgr = img[...,::-1]
# upper and lower bound of dark-yellow
yellow_lower = np.array([0, 100, 120], dtype='uint8')
yellow_upper = np.array([50, 150, 185], dtype='uint8')
# get masking with opencv
rock_mask = cv2.inRange(bgr, yellow_lower, yellow_upper)
```
![alt text][image2]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
In `process_image()` function, naviagte, obsticale and rock mapping to worldmap is implemented. 
You may view the faster mp4 version of [this gif](./output)
![alt text][image3]

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
The logic `perception_step()` is basically the same in `process_image()` in Jupyter Notebook. Except one extra function to get the angle and distance for robot control purpose:
```
Rover.nav_dists = rover_centric_pixel_distances
Rover.nav_angles = rover_centric_angles
```
Prevent obstacles and navigation overlapping:
```
Rover.worldmap[navigable_y_world, navigable_x_world, 2] += 1
Rover.worldmap[navigable_y_world, navigable_y_world, 0] -= 1
```

I make a litte bit modification in `decision_step()` example to test out if I can reach better result:
```
if np.mean(Rover.nav_dists) > 60:
    Rover.max_vel = Rover.basic_vel * 2
else:
    Rover.max_vel = Rover.basic_vel 
```
```
if Rover.vel == 0:
    Rover.mode = 'stop'
```
```
Rover.steer = Rover.stop_steer
if len(Rover.nav_angles) >= Rover.go_forward:
    # Set throttle back to stored value
    Rover.stop_steer = Rover.stop_steer * -1
```

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

* It takes 150 second to reach 47% Mapping
I try to set `Rover.max_vel` higher if the distance seems further. But it doesn't help. May need a precise number of nav_angles.
Sometimes, if the robot reach the large open area on the right side of the map, it just goes round and round. I can add mapped pixel to its memory to prevent going to area that already been mapped
* The fedility is 70%. I can make better decision to the robot to prevent hitting the wall.
* Sometimes, if the robot hit the large obstacles in the center of the map, it will stop. I add the logic when it stops (vel == 0), it should go to stop mode and turn.

![alt text][image4]


