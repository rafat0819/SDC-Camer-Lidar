# Self Driving Car: Camera-lidar Fusion Tracking

In this project, we used real-world data from the Waymo Open Dataset and apply an extended Kalman filter for sensor fusion and tracking multiple vehicles.
The major tasks accomplished to complete the project:

1. Implement a kalman filter to track a object
2. Track management, initialize, update and delete tracks to manage tracking
3. Data association
4. Camera sensor fusion, on the basis of lidar fusion, add camera measurement fusion

To run the project, simply run the script loop_over_dataset.py.

## Step-1: Extended Kalman Filter

We use ekf to track objects. EKF is implemented in filter.py.

Implementing an ekf includes below work,

* Design system sate [x,y,z,vx,vy,vz]
* Design process model, consant velocity model
* implement predict step, with constant velociy model and process noise increasing with delta time
* implement upate step, with lidar measurement model

For the road segment specified in classroom, the following graph was plotted.
<p>
    <img src="img/Final Term/s1.png"/>
    <br>
    <em>Fig 1</em>
</p>

## Step-2: Track Management

Track management is implemented in trackmanagement.py.

Below are the key points of track management:
* Track is initialized after receiving an unassigned lidar measurement.
* A track contains scores. Its score will be increased if it's associated with a measurement, and decreased otherwise.
* A track contains state, which will be updated based on its score, 'confirmed' if the confirmed threshold is exceeded.
* A track will be deleted if its score is below a certain treshold.

For the road segment specified in classroom, we can see the track is initialized, confirmed, and then deleted. The following graph is plotted.

<p>
    <img src="img/Final Term/s2.png"/>
    <br>
    <em>Fig 2</em>
</p>

## Step-3: Data Association

Data association is implemented in association.py.

Below are the key steps invovled in data association,

* Create a mtraix for all available tracks and obervations.
* Calculate Mahalanobis distance (MHD) for each track measurement pairs.
* Use Chi Square hypotheisis test to reject unlikely track measurement pairs.
* Pick out the pair with smallest MHD, perform ekf update step, and then remove corresponding row and column in the association matrix.

For the road segment specified in classroom, we can see the track is initialized, confirmed, and then deleted. The following graph is plotted.

<p>
    <img src="img/Final Term/s3.png"/>
    <br>
    <em>Fig 3</em>
</p>

After implementing, we can see that the multiple measurements are propelry associated with multiple tracks.

## Step-4: Camera Sensor Fusion

In the step, we add camera measurements to the ekf fusion. It is implemented in the measurement.py.

Implementing camera measurement fusion consists of below major aspects,

* measurment model, the projection matrix which transforms points from 3d in space to 2d in image.
* Jocobian for the measurement model, the partial derivative of system state(x,y,z) with respect to measurement(u, v)
* measurement noise R for camera measurement
* check if a track's state is within camera's field of view 

## Evaluation

Tracking performance is evaluated using RMSE metrics

<p>
    <img src="img/Final Term/s4.png"/>
    <br>
    <em>Fig 4</em>
</p>

## Evaluation

The tracking movie is saved under the directory /result/movies.

<p>
    <img src="img/video.gif"/>
</p>

## Reflection

### Most Difficult Part

In my opinion, I faced the most difficulty while implementing the camera measurement model, projecting 3D points to 2D points. Waymo's camera coordinate frame deviates from the z-froward, x-right, and y down coordinate frame, so it is difficult to implement cameara axis transformation.

### Benifits of Camera-lidar Fusion Tracking over Lidar-only Tracking

Theoritically, camera-fusion tracking is said to improve the robustness and accuracy of the system.

However, in this project, the camera measurement model was not perfect. It tries projecting the centroid of 3d bounding box in space to the center of bounding box in image, in reality this assumption does not hold true. Thus, in this case, the camera-fusion tracking doesn't add much improvement to the system over lidar-only tracking.

### Real-life Challenges

A sensor-fusion system can face a number of challenges in real-life. This challenges also exist in this project.

* accurate association between mutiple tracks and multiple measures - It is difficult to set a good gating threshold for all scenarios.
* measurement noise configuration - Each measurement should come with its own noice variance rather than a fixed noise variance for the sensor.
* choice of process model - It is important to find a good balance between accuracy and computations.

### Improvement Opportunity

Since the current camera measurements model is very rough, adding camera-fusion tracking wont be beneficial to the system. Alternatively, we can match the actual 3d points in lidar point cloud to vehicle object pixels in image. A theorectially sound camera measurement model should help with the fusion result.
