- Runs <u>only once</u> when arm is started up.
- Finds the angle of both segments of the arm relative to base IMU using trigonometry on acceleration due to gravity given by IMU data.
- Publishes the initial angle of each joint to /Arm/Odometry
- Moves the arm to pre-defined default position