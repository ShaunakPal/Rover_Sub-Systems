Drive System: 
	Base Station: 
		Launch file 1: for Drive System
			1. Velocity publisher node 
				1. Imports Joystick mapping class
				2. publisher sending array of 8 integers 
			3. FPV feed
				1. CPP script running opencv
			4. IP cam
				1. CPP script running opencv
	Jetson:
		1. Motor Control node
			1. Subscribes to Velocity publisher node
			2. Sends data to STM
		2. Sensor data node
			1. Publishes GPS data
			2. Publishes IMU data
		3. Odometry Publisher node 
			1. Subscribes to Sensor data node
			2. Publishes odometry data

Science System:
	Base Station: 
		Launch file 2: for Science System
			1. Velocity publisher node 
				1. Imports Joystick mapping class
				2. publisher sending array of 12 values 
			2. Data visualisation and management node
				1. Subscribes to Sensor data node
				2. Converts data to csv
				3. Plots the csv data using matplotlib
			3. FPV feed
				1. CPP script running opencv
			4. IP cam
				1. CPP script running opencv
	Jetson:
		1. Motor Control node
			1. Subscribes to Velocity publisher node
			2. Sends data to STM
		2. Sensor data node
			1. Publishes Sensor data
		3. Data management node
			1. Subscribes to sensor data
			2. Converts to csv and logs

Relevant code snippets:
1. Sample publisher 

```python
	import rclpy
	from std_msgs.msg import Int32MultiArray
	import time
	
	def main():
	rclpy.init()
	node = rclpy.create_node('array_publisher')
	
	# Create a publisher for the Int32MultiArray
	publisher = node.create_publisher(Int32MultiArray, '/Drive/motorpwm', 10)
	
	# Create an array to hold the 8 integers
	array_data = [0, 0, 0, 0, 0, 0, 0, 0]
	
	# Create a message of type Int32MultiArray
	msg = Int32MultiArray(data=array_data)
	
	try:
		while True:
			# Publish the message
			publisher.publish(msg)
			node.get_logger().info('Publishing: %s' % msg.data)
			time.sleep(1)  # Adjust the sleep interval as needed
	except KeyboardInterrupt:
		pass
	
	# Shutdown the node and release resources when done
	node.destroy_node()
	rclpy.shutdown()
	
	if __name__ == '__main__':
	main()
	```

IP CAM:
```cpp
int main(int, char**) {
    cv::VideoCapture vcap;
    cv::Mat image;

    // This works on a D-Link CDS-932L
    const std::string videoStreamAddress = "http://ID:PASSWORD@IPADDRESS:PORTNO/mjpeg.cgi?user=ID&password=ID:PASSWORD&channel=0&.mjpg";
       //open the video stream and make sure it's opened
    if(!vcap.open(videoStreamAddress)) {
        std::cout << "Error opening video stream or file" << std::endl;
        return -1;
    }

    for(;;) {
        if(!vcap.read(image)) {
            std::cout << "No frame" << std::endl;
            cv::waitKey();
        }
        cv::imshow("Output Window", image);

        if(cv::waitKey(1) >= 0) break;
    }   

}
```


Science System Template
```python
import rclpy
from your_package_name.msg import ScienceOut  # Import your custom message type
from std_msgs.msg import Int32MultiArray

def main():
    rclpy.init()
    node = rclpy.create_node('science_out_publisher')

    # Create a publisher for the Int32MultiArray
    publisher = node.create_publisher(Int32MultiArray, '/science_out_array', 10)

    try:
        while True:
            # Create an array of ScienceOut messages
            science_out_array = []
            
            # Fill in each ScienceOut message with example values
            for _ in range(8):
                msg = ScienceOut()
                msg.drill_dir = 1
                msg.drill_speed = 2
                msg.drill_distance = 3
                msg.auger_dir = 4
                msg.auger_speed = 5
                msg.carousel_dir = 6
                msg.carousel_speed = 7
                msg.reagent_direction = 8
                msg.reagent_speed = 9
                msg.npk_servo = 10
                msg.micro_dir = 11
                msg.micro_speed = 12
                
                science_out_array.append(msg)

            # Publish the array of ScienceOut messages
            int32_array = Int32MultiArray()
            int32_array.data = [field for msg in science_out_array for field in [
                msg.drill_dir,
                msg.drill_speed,
                msg.drill_distance,
                msg.auger_dir,
                msg.auger_speed,
                msg.carousel_dir,
                msg.carousel_speed,
                msg.reagent_direction,
                msg.reagent_speed,
                msg.npk_servo,
                msg.micro_dir,
                msg.micro_speed
            ]]
            publisher.publish(int32_array)
            node.get_logger().info('Publishing: %s' % int32_array.data)
            rclpy.spin_once(node)
    except KeyboardInterrupt:
        pass

    # Shutdown the node and release resources when done
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## FPV servo driver node:

- Uses PCA9685 driver
- Subscribes to /Base/`<no.>`/cmd_vel and updates to the Int32MultiArray containing exactly 16 values corresponding to the 16 PWM channels.`{data: [32767, 0, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1]}`
- Sends the pwn values to the driver via i2c

ROS package: https://github.com/dheera/ros-pwm-pca9685
Dependency: https://github.com/Reinbert/pca9685/tree/master
___
## IP camera move node:
- Input: drive joystick
- Publishes on: /Base/IPout
	- message
		- int32 motor_dir
		- int32 motor_speed
___
## GPS publisher node:
- publishes GPS data on the topic `/Base/gps_raw`
### Example code:
gps_publisher_node.py

```
import rclpy
from sensor_msgs.msg import NavSatFix
import serial

ser = serial.Serial('/dev/ttyS0', 9600)  # Adjust the serial port and baud rate as needed

def get_gps_data():
	line = ser.readline().decode('utf-8')
    if line.startswith('$GPGGA'):
        data = line.split(',')
        if len(data) >= 10:
            latitude = float(data[2])
            longitude = float(data[4])
            altitude = float(data[9])
    
    return [latitude,longitude,altitude]

def gps_publisher_node():
    rclpy.init()
    node = rclpy.create_node('GPS_publisher')

    publisher = node.create_publisher(NavSatFix, '/Base/gps_raw', 10)

    while rclpy.ok():
	    latitude, longitude, altitude = get_gps_data()
        msg = NavSatFix()
        msg.latitude = latitude
        msg.longitude = longitude
        msg.altitude = altitude
        publisher.publish(msg)
        node.get_logger().info('Publishing GPS data')
        rclpy.spin_once(node)

    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    gps_publisher_node()
```
___
## Science Sensor data management and visualisation

### Live plotter:
```
import matplotlib.pyplot as plt
import numpy as np
import random
from itertools import count
from matplotlib.animation import FuncAnimation

# Initialize an empty plot
x_data = []
y_data = []
index = count()

# Set up the plot (e.g., labels, titles)
fig, ax = plt.subplots()
ax.set_xlabel('Time')
ax.set_ylabel('Value')
ax.set_title('Live Data Plot')

# Create an empty line for updating the data
line, = ax.plot(x_data, y_data, 'b-') # 'b-' for a blue line

def update_data(i):
	new_x = next(index)
	new_y = random.random() # Replace with your live data source
	x_data.append(new_x)
	y_data.append(new_y)
	
	# Limit the number of data points to keep the plot from getting too crowded
	max_data_points = 100
	if len(x_data) > max_data_points:
		x_data.pop(0)
		y_data.pop(0)

	# Update the data of the line plot
	line.set_data(x_data, y_data)

# Automatically adjust the x-axis limits to show the most recent data
ax.relim()
ax.autoscale_view()

# Create an animation that calls update_data every 100 milliseconds (adjust as needed)
ani = FuncAnimation(fig, update_data, interval=100)

# Display the live plot
plt.show()
```

### Live Data logger:
```
import numpy as np
import random
from itertools import count
import csv
import time

# Initialize empty lists for data
x_data = []
y_data = []
index = count()

# Create an empty CSV file for data logging
csv_filename = 'live_data.csv'
with open(csv_filename, 'w', newline='') as csv_file:
csv_writer = csv.writer(csv_file)
csv_writer.writerow(['Time', 'Value'])

# Simulate live data updates and save to CSV
while True:
	new_x = next(index)
	new_y = random.random() # Replace with your live data source
	
	x_data.append(new_x)
	y_data.append(new_y)
	
	max_data_points = 100
	if len(x_data) > max_data_points:
		x_data.pop(0)
		y_data.pop(0)
	
	# Append the data to the CSV file
	with open(csv_filename, 'a', newline='') as csv_file:
	csv_writer = csv.writer(csv_file)
	csv_writer.writerow([new_x, new_y])

	time.sleep(1) # Adjust the sleep interval as needed
```