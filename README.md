# Euclidean Clustering with ROS and PCL (Ex1 / Ex2)

A simple stick robot with an RGB-D camera attached to its head via a pan-tilt joint is placed in front of the table.  For the detailed steps of how to carry out this exercise, please see the [Clustering for Segmentation](https://classroom.udacity.com/nanodegrees/nd209/parts/586e8e81-fc68-4f71-9cab-98ccd4766cfe/modules/e5bfcfbd-3f7d-43fe-8248-0c65d910345a/lessons/2cc29bbd-5c51-4c3e-b238-1282e4f24f42/concepts/02428d63-6f79-40dc-8105-31eda8e0def4) lesson in the RoboND classroom.

Here's a brief summary of how to get setup for the exercise:

1. First of all copy/move the `sensor_stick` package to `/src` directory of your active ros workspace. 

2. Make sure you have all the dependencies resolved by using the rosdep install tool and run `catkin_make`:  

```sh
$ cd ~/catkin_ws
$ rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y
$ catkin_make
```
3. Add following to your .bashrc file
```
export GAZEBO_MODEL_PATH=~/catkin_ws/src/sensor_stick/models

source ~/catkin_ws/devel/setup.bash
```

4. Test the simulation setup by launching the gazebo environment. The command stated below will open a gazebo world along with an rviz window. 

```sh
$ roslaunch sensor_stick robot_spawn.launch
```

5. After RViz and Gazebo are online start the segmentation.py. Cd to the file location

```sh
$ ./segmentation.py
```

[image1]: ./misc_images/exercise2.png
[image2]: ./misc_images/exercise3.png


![screen shot 2017-07-05 at 12 56 36 pm](https://user-images.githubusercontent.com/20687560/27895526-30da599c-61c8-11e7-80ab-4b4224cfbb10.png)
To perform the solution build your perception pipeline, that must perform following steps:

1. Create a python ros node that subscribes to `/sensor_stick/point_cloud` topic. Use the `template.py` file found under /sensor_stick/scripts/ to get started.

2. Use your code from Exercise-1 to apply various filters and segment the table using RANSAC. 

3. Create publishers and topics to publish the segmented table and tabletop objects as separate point clouds 

4. Apply Euclidean clustering on the table-top objects (after table segmentation is successful)

5. Create a XYZRGB point cloud such that each cluster obtained from the previous step has its own unique color.

6. Finally publish your colored cluster cloud on a separate topic 
![clusters][image1]


You will find `pcl_helper.py` file under `/sensor_stick/scripts`. This file contains various functions to help you build up your perception pipeline. 


# Object Recognition with Python, ROS and PCL (Ex. 3)

In this exercise, you will continue building up your perception pipeline in ROS.  Here you are provided with a very simple gazebo world, where you can extract color and shape features from the objects that were sitting on the table from Exercise-1 and Exercise-2, in order to train a classifier to detect them.


## Setup
* If you completed Exercises 1 and 2 you will already have a `sensor_stick` folder in your `~/catkin_ws/src` directory.  You should replace that folder with the `sensor_stick` folder contained in this repository and add the Python script you wrote for Exercise-2 to the `scripts` directory. 

* If you do not already have a `sensor_stick` directory, first copy/move the `sensor_stick` folder to the `~/catkin_ws/src` directory of your active ros workspace. 

* Make sure you have all the dependencies resolved by using the `rosdep install` tool and running `catkin_make`:  
 
```sh
$ cd ~/catkin_ws
$ rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y
$ catkin_make
```

* If it's not already there, add the following lines to your `.bashrc` file  

```
export GAZEBO_MODEL_PATH=~/catkin_ws/src/sensor_stick/models
source ~/catkin_ws/devel/setup.bash
```

## Preparing for training

Launch the `training.launch` file to bring up the Gazebo environment: 

```sh
$ roslaunch sensor_stick training.launch
```
You should see an empty scene in Gazebo with only the sensor stick robot.

## Capturing Features
Next, in a new terminal, run the `capture_features.py` script to capture and save features for each of the objects in the environment.  This script spawns each object in random orientations (default 5 orientations per object) and computes features based on the point clouds resulting from each of the random orientations.

```sh
$ rosrun sensor_stick capture_features.py
```

The features will now be captured and you can watch the objects being spawned in Gazebo. It should take 5-10 sec. for each random orientations (depending on your machine's resources) so with 7 objects total it takes awhile to complete. When it finishes running you should have a `training_set.sav` file.

## Training

Once your feature extraction has successfully completed, you're ready to train your model. First, however, if you don't already have them, you'll need to install the `sklearn` and `scipy` Python packages.  You can install these using `pip`:

```sh
pip install sklearn scipy
```

After that, you're ready to run the `train_svm.py` model to train an SVM classifier on your labeled set of features.

```sh
$ rosrun sensor_stick train_svm.py
```
**Note:  Running this exercise out of the box your classifier will have poor performance because the functions `compute_color_histograms()` and `compute_normal_histograms()` (within `features.py` in /sensor_stick/src/sensor_stick) are generating random junk.  Fix them in order to generate meaningful features and train your classifier!**

## Classifying Segmented Objects

If everything went well you now have a trained classifier and you're ready to do object recognition!  First you have to build out your node for segmenting your point cloud.  This is where you'll bring in your code from Exercises 1 and 2.

Make yourself a copy of the `template.py` file in the `sensor_stick/scripts/` directory and call it something like `object_recognition.py`.  Inside this file, you'll find all the TODO's from Exercises 1 and 2 and you can simply copy and paste your code in there from the previous exercises.  

The new code you need to add is listed under the Exercise-3 TODO's in the `pcl_callback()` function.  You'll also need to add some new publishers for outputting your detected object clouds and label markers.  For the step-by-step instructions on what to add in these Exercise-3 TODOs, see the [lesson in the classroom](https://classroom.udacity.com/nanodegrees/nd209/parts/586e8e81-fc68-4f71-9cab-98ccd4766cfe/modules/e5bfcfbd-3f7d-43fe-8248-0c65d910345a/lessons/81e87a26-bd41-4d30-bc8b-e747312102c6/concepts/dfab1b50-2efd-428d-bfd9-d1df0544541e).

## Object recognition
	roslaunch sensor_stick robot_spawn.launch
In another terminal:
	chmod +x object_recognition.py
	./object_recognition.py
![object recognition][image2]


