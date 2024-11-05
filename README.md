# Real Time Driver State Detection 

Real time, webcam based, driver attention state detection and monitoring using Python with the OpenCV and mediapipe libraries.

## Installation

This projects runs on Python with the following libraries:

- numpy
- OpenCV (opencv-python)
- mediapipe

Or you can use poetry to automatically create a virtualenv with all the required packages:

```
pip install poetry #a global install of poetry is required
```

Then inside the repo directory:

```
poetry install
```

To activate the env to execute command lines:

```
poetry shell
```

Alternatively (not recommended), you can use the requirements.txt file provided in the repository using:
    
    pip install -r requirements.txt
    

## Usage

First navigate inside the driver state detection folder:
    
    cd driver_state_detection

The scripts can be used with all default options and parameters by calling it via command line:

    python main.py


**Note**:
This work is partially based on [this paper](https://www.researchgate.net/publication/327942674_Vision-Based_Driver%27s_Attention_Monitoring_System_for_Smart_Vehicles) for the scores and methods used.

### Mediapipe Features added:

- 478 face keypoints detection
- Direct iris keypoint detection for gaze score estimation
- Improved head pose estimation using the dynamical canonical face model
- Fixed euler angles function and wrong returned values
- Using time variables to make the code more modular and machine agnostic

## How Does It Work?

This script searches for the driver face, then use the mediapipe library to predict 478 face and iris keypoints.
The enumeration and location of all the face keypoints/landmarks can be seen [here](https://github.com/e-candeloro/Driver-State-Detection/blob/master/docs/5Mohl.jpg).

With those keypoints, the following scores are computed:

- **EAR**: Eye Aspect Ratio, it's the normalized average eyes aperture, and it's used to see how much the eyes are opened or closed
- **Gaze Score**: L2 Norm (Euclidean distance) between the center of the eye and the pupil, it's used to see if the driver is looking away or not
- **Head Pose**: Roll, Pitch and Yaw of the head of the driver. The angles are used to see if the driver is not looking straight ahead or doesn't have a straight head pose (is probably unconscious)
- **PERCLOS**: PERcentage of CLOSure eye time, used to see how much time the eyes are closed in a minute. A threshold of 0.2 is used in this case (20% of a minute) and the EAR score is used to estimate when the eyes are closed.

The driver states can be classified as:

- **Normal**: no messages are printed
- **Tired**: when the PERCLOS score is > 0.2, a warning message is printed on screen
- **Asleep**: when the eyes are closed (EAR < closure_threshold) for a certain amount of time, a warning message is printed on screen
- **Looking Away**: when the gaze score is higher than a certain threshold for a certain amount of time, a warning message is printed on screen
- **Distracted**: when the head pose score is higher than a certain threshold for a certain amount of time, a warning message is printed on screen


## The Scores Explained

### EAR

**Eye Aspect Ratio** is a normalized score that is useful to understand the rate of aperture of the eyes.
Using the mediapipe face mesh keypoints for each eye (six for each), the eye lenght and width are estimated and using this data the EAR score is computed as explained in the image below:
![EAR](https://user-images.githubusercontent.com/67196406/121489162-18210900-c9d4-11eb-9d2e-765f5ac42286.png)

**NOTE:** the average of the two eyes EAR score is computed

### Gaze Score Estimation

The gaze score gives information about how much the driver is looking away without turning his head.

To understand this, the distance between the eye center and the position of the pupil is computed. The result is then normalized by the eye width that can be different depending on the driver physionomy and distance from the camera.

The below image explains graphically how the Gaze Score for a single eye is computed:
![Gaze Score](https://user-images.githubusercontent.com/67196406/121489746-ab5a3e80-c9d4-11eb-8f33-d34afd0947b4.png)
**NOTE:** the average of the two eyes Gaze Score is computed

### Head Pose Estimation

For the head pose estimation, a standard 3d head model in world coordinates was considered, in combination of the respective face mesh keypoints in the image plane. 
In this way, using the solvePnP function of OpenCV, estimating the rotation and translation vector of the head in respect to the camera is possible.
Then the 3 Euler angles are computed.


