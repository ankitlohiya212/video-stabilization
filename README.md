# video-stabilization
Video Stabilization with Python and OpenCV

## How It Works :
## Step 1 : Set Input and Output Videos
First, let’s complete the setup for reading the input video and writing the output video. The comments in the code explain every line.

## Step 2: Read the first frame and convert it to grayscale
For video stabilization, we need to capture two frames of a video, estimate motion between the frames, and finally correct the motion.

## Step 3: Find motion between frames
This is the most crucial part of the algorithm. We will iterate over all the frames, and find the motion between the current frame and the previous frame. It is not necessary to know the motion of each and every pixel. The Euclidean motion model requires that we know the motion of only 2 points in the two frames. However, in practice, it is a good idea to find the motion of 50-100 points, and then use them to robustly estimate the motion model.

### 3.1 Good Features to Track
The question now is what points should we choose for tracking. Keep in mind that tracking algorithms use a small patch around a point to track it. Such tracking algorithms suffer from the aperture problem as explained in the video below

So, smooth regions are bad for tracking and textured regions with lots of corners are good. Fortunately, OpenCV has a fast feature detector that detects features that are ideal for tracking. It is called goodFeaturesToTrack

### 3.2 Lucas-Kanade Optical Flow
Once we have found good features in the previous frame, we can track them in the next frame using an algorithm called Lucas-Kanade Optical Flow named after the inventors of the algorithm.

It is implemented using the function calcOpticalFlowPyrLK in OpenCV. In the name calcOpticalFlowPyrLK, LK stands for Lucas-Kanade, and Pyr stands for the pyramid. An image pyramid in computer vision is used to process an image at different scales (resolutions).

calcOpticalFlowPyrLK may not be able to calculate the motion of all the points because of a variety of reasons. For example, the feature point in the current frame could get occluded by another object in the next frame. Fortunately, as you will see in the code below, the status flag in calcOpticalFlowPyrLK can be used to filter out these values.

### 3.3 Estimate Motion
To recap, in step 3.1, we found good features to track in the previous frame. In step 3.2, we used optical flow to track the features. In other words, we found the location of the features in the current frame, and we already knew the location of the features in the previous frame. So we can use these two sets of points to find the rigid (Euclidean) transformation that maps the previous frame to the current frame. This is done using the function estimateRigidTransform.

Once we have estimated the motion, we can decompose it into x and y translation and rotation (angle). We store these values in an array so we can change them smoothly.

## Step 4: Calculate smooth motion between frames
In the previous step, we estimated the motion between the frames and stored them in an array. We now need to find the trajectory of motion by cumulatively adding the differential motion estimated in the previous step.

### Step 4.1 : Calculate trajectory
In this step, we will add up the motion between the frames to calculate the trajectory. Our ultimate goal is to smooth out this trajectory
We also, define a function cumsum that takes in a vector of TransformParams and returns trajectory by performing the cumulative sum of differential motion dx, dy, and da (angle).

### Step 4.2 : Calculate smooth trajectory
In the previous step, we calculated the trajectory of motion. So we have three curves that show how the motion (x, y, and angle) changes over time.

In this step, we will show how to smooth these three curves.

The easiest way to smooth any curve is to use a moving average filter. As the name suggests, a moving average filter replaces the value of a function at the point by the average of its neighbors defined by a window.

### Step 4.3 : Calculate smooth transforms
So far we have obtained a smooth trajectory. In this step, we will use the smooth trajectory to obtain smooth transforms that can be applied to frames of the videos to stabilize it.

This is done by finding the difference between the smooth trajectory and the original trajectory and adding this difference back to the original transforms.

## Step 5: Apply smoothed camera motion to frames
We are almost done. All we need to do now is to loop over the frames and apply the transforms we just calculated.

### Step 5.1 : Fix border artifacts
When we stabilize a video, we may see some black boundary artifacts. This is expected because to stabilize the video, a frame may have to shrink in size.

We can mitigate the problem by scaling the video about its center by a small amount (e.g. 10%).

The function fixBorder below shows the implementation. We use getRotationMatrix2D because it scales and rotates the image without moving the center of the image. All we need to do is call this function with 0 rotation and scale 1.1 ( i.e. 10% upscale)
