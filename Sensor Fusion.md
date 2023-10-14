For the drive IMU and Rotary encoder fusion we should use a ***custom Extended Kalman Filter (EKF)*** owing to the relatively low complexity of the arm and high error correction with this method.

**Extended Kalman Filter (EKF):**
- **Error Percentages:** EKFs are versatile and can handle both nonlinear and noisy systems, so they can effectively combine IMU and encoder data. With proper tuning and sensor calibration, error percentages can be minimized.
- **Computational Intensity:** EKFs can be computationally intensive, especially in higher-dimensional state spaces. However, their complexity can be managed by optimizing the implementation and leveraging ROS 2's real-time capabilities.

## Other Options for Sensor Fusion Algorithms:
1. **Sensor Fusion Libraries (e.g., Madgwick Filter, Mahony Filter):**
   - **Error Percentages:** These libraries provide quaternion-based sensor fusion methods that are well-suited for IMU data fusion. Error percentages can be relatively low if sensors are well-calibrated and motion is not too extreme.
   - **Computational Intensity:** Sensor fusion libraries like Madgwick or Mahony are efficient and optimized for embedded systems. They are suitable for real-time applications and can be integrated into a ROS 2 environment.

2. **Weighted Averaging:**
   - **Error Percentages:** Weighted averaging can be effective when you have a good estimate of the sensor accuracies. However, it may not handle complex dynamics or sensor errors as well as other techniques.
   - **Computational Intensity:** Weighted averaging is computationally straightforward and efficient. It involves basic mathematical operations and is suitable for real-time applications.

___
Reference:
https://www.sciencedirect.com/science/article/abs/pii/S0921889022002329