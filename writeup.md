## Project: Perception Pick & Place


---
[//]: # (Image References)
[image1]: ./misc/confusion_mat.png
[image2]: ./misc/objrecognition.png

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.
The filters in Exercise 1 were implemented. First of all, the PCL data were denoised using the statistical outlier filter. After some experiments, the number of neighboring points was set to 5 and the number of standard deviations for the threshold was set to 0.5. For the voxel grid downsampling, the leaf size was chosen as 0.004. The objects were passed through in the z direction between 0.6 and 1.1 meters. In addition, to remove the boxes from the filtered scene, another pass through filter was applied between 0.4 and 1 meters in the x direction. The maximal distance of the RANSAC plane fitting was set to 0.01, with which the inliers (table) and outliers (objects) were selected.

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  
To segment the objects, Euclidean clustering was implemented. The cluster tolerance was set to 0.02 and the cluster size was restricted to 40 to 4000 points. Each object was masked by a distinct color.

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
The objects were then classified based on the color and the normal histograms and a pre-trained SVM model. To train the SVM, features of the model objects in different angles were captured for 100 times and HSV color was used to compensate the brightness. The diagonal of the confusion matrix had values above 0.7 (Fig. 1).In Figure 2, the object recognition results are showed.

![alt text][image1]

Figure 1

![alt text][image2]

Figure 2

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

In this submission, the project_template.py is the main script for running the pick-place task. The three output .yaml files are included, together with the model.sav and training_set.sav files. The object_recognition.py script is for testing the parameters.

The logic of the pick-place process is as following:
First, the information about the list of objects that were supposed to be picked was read from the .yaml file using rospy.get_param function. I created a list of object names, as well a list of their corresponding groups (red or green).

Next, the information about the drop boxes was imported from the .yaml file to lists, including the names of the boxes (left or right), group (red or green) and position in x,y,z.

Then it looped through the detected object list. The label of each object was extracted and compared to the target list. If the detected object was on the list, the group information was identified, telling which box to drop at. However, if the object name was not on that list, it would perform a random guess about which box to drop at.

Next, the centroid of the object was calculated from the coordinate of the point cloud of the object for the robot arm to pick it up. The pick_pose was set to that position in x,y,z. In addition, the place_pose was set to the coordinate of the drop box.

Once the group of drop box was determined, the name of that drop box (left or right) was assigned to the arm_name.

A dictionary was created onces having all the information and was appended to dict_list, which would be saved in the output file. Notice that the test_scene_num was set manually.

The overall recognition result was good. It achieved 3/3 correct in world 1, 5/5 correct in world 2 and 7/8 correct in world 3. Except relatively large number of training iterations, I found the noise reduction in the first step very important. In addition, a small leaf size is crucial for recognizing tipping objects such as the glue. For example, a leaf size of 0.005 failed to recognize the glue while 0.004 did the job.

Although the recognition worked well, the picking up and dropping off created some errors. The grippers were not able to grab objects quite often. The dropped items sometimes also stayed on top of instead of inside the drop box. An object could also be knocked off by other objects in the way. These types of issues were not dealt with in this project.
