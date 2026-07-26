# Simulation-setup-for-UAV-object-tracking
Simulation setup (raspberry pi) for UAV object tracking 

## **Step 1: Simulating the Drone (Using SITL)**  

SITL (Software in the Loop) is like a pretend Pixhawk flight controller that runs on your computer. It lets you simulate how a drone flies, reads GPS, and reacts to commands.

### **1.1 Installing SITL**  
#### **For Windows:**
1. Download **Mission Planner** from [this link](https://ardupilot.org/planner/docs/mission-planner-installation.html).  
2. Install it. That’s it—you’re good to go for SITL on Windows.

#### **For Linux:**  
1. Open a terminal and run these commands:  
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip git -y
   git clone https://github.com/ArduPilot/ardupilot.git
   cd ardupilot
   git submodule update --init --recursive
   ./Tools/environment_install/install-prereqs-ubuntu.sh -y
   . ~/.profile
   ./waf configure --board sitl
   ./waf build
   ```
   This will download and set up SITL on your machine. It takes a little time, but don’t worry—this only happens once.  

### **1.2 Launch SITL**
To start the fake drone:  
```bash
sim_vehicle.py -v ArduCopter --map --console
```
- **`--map`**: Opens a live map that shows where the drone is flying.  
- **`--console`**: Opens a terminal where you can type commands for the drone.  

If it launches, congrats! You’ve got a simulated drone.

---

## **Step 2: Simulating the Raspberry Pi (Optional)**  

The Raspberry Pi part isn’t strictly necessary, but if you want to feel like you’re working with a Pi, you can set up a virtual Pi environment.

### **2.1 Using Docker for a Raspberry Pi Environment**  
Docker is like a sandbox where you can pretend you’re running a Raspberry Pi.  
1. Download Docker from [here](https://www.docker.com/).  
2. Set it up by running this:  
   ```bash
   docker pull balenalib/rpi-raspbian
   docker run -it balenalib/rpi-raspbian
   ```
   Boom—you’re inside a virtual Raspberry Pi!

---

## **Step 3: Setting Up YOLOv7 (Object Detection)**  

Here’s where things get exciting. YOLOv7 (You Only Look Once) is an object detection algorithm that can identify stuff in images or videos—like cars, tanks, or even people.

### **3.1 Installing YOLOv7**
1. Clone the YOLOv7 repository from GitHub:
   ```bash
   git clone https://github.com/WongKinYiu/yolov7.git
   cd yolov7
   ```
2. Install the required Python libraries:
   ```bash
   pip install -r requirements.txt
   ```

### **3.2 Running YOLOv7**
1. Download the YOLOv7 weights (a pretrained model file). You can find it in the repo or from the [YOLOv7 releases](https://github.com/WongKinYiu/yolov7).  
2. Use a sample video to test detection:
   ```bash
   python detect_and_track.py --source "test_video.mp4" --weights yolov7.pt --view-img
   ```
   - **`--source`**: Your video file (e.g., `"test_video.mp4"`).  
   - **`--weights`**: The YOLOv7 weight file (e.g., `yolov7.pt`).  
   - **`--view-img`**: Lets you see the results as the video plays.

If it shows bounding boxes on objects in your video, it’s working! 🎉  

---

## **Step 4: Making the Drone and Camera Talk**  

Now, we need to link the object detection system (YOLOv7) with your fake drone (SITL). We’ll use **Dronekit**, a Python library, to send commands to the drone.

### **4.1 Install Dronekit**
1. Install Dronekit and Dronekit SITL:
   ```bash
   pip install dronekit dronekit-sitl
   ```

### **4.2 Test the Drone Connection**
1. Use this Python script to check if the drone is listening:  
   ```python
   from dronekit import connect

   # Connect to your simulated drone (default SITL address)
   vehicle = connect('127.0.0.1:14550', wait_ready=True)

   # Print the GPS coordinates
   print("GPS Location:", vehicle.location.global_frame)
   ```
2. Run the script. If it prints the GPS data, your drone is connected!  

---

## **Step 5: Simulating the Camera Feed**  

We’ll use pre-recorded videos or even simulate a webcam feed. Here’s how:  

### **5.1 Pre-Recorded Videos**
This is the easiest way. Just load a video file into YOLOv7 as the source:  
```bash
python detect_and_track.py --source "test_video.mp4" --weights yolov7.pt --view-img
```

### **5.2 Simulate a Live Camera**
Want to make it feel like you’re using a real camera? Let’s set that up.  

#### For Linux:
1. Install **v4l2loopback**:
   ```bash
   sudo apt install v4l2loopback-dkms
   modprobe v4l2loopback
   ```
2. Stream a video to the virtual camera:
   ```bash
   ffmpeg -re -i test_video.mp4 -f v4l2 /dev/video0
   ```

---

## **Step 6: Setting Up a Geofence (Stay Inside the Lines)**  

To stop the drone from wandering too far, we’ll use a geofence. Here’s how to code it:  
```python
from geopy.distance import geodesic

# Center of the geofence (lat, lon)
geofence_center = (37.7749, -122.4194)  # Example: San Francisco
geofence_radius = 500  # Radius in meters

# Simulated target location (lat, lon)
target_location = (37.7750, -122.4180)

# Check if the target is within the geofence
distance = geodesic(geofence_center, target_location).meters
if distance <= geofence_radius:
    print("Target is inside the geofence.")
else:
    print("Target is outside the geofence.")
```

---

## **Step 7: Moving the Drone Toward a Target**  

Here’s how to send your drone to an object’s location:  
```python
from dronekit import LocationGlobal

# Target coordinates (lat, lon, altitude)
target_lat = 37.7750
target_lon = -122.4180
target_alt = 10  # Altitude in meters

# Command the drone to move
vehicle.simple_goto(LocationGlobal(target_lat, target_lon, target_alt))
print("Drone is moving to the target!")
```

---

## **Step 8: Emergency Stop (Return to Launch)**  

If the drone leaves the geofence or something goes wrong, make it return to its starting point:  
```python
from dronekit import VehicleMode

# Change mode to "RTL" (Return to Launch)
vehicle.mode = VehicleMode("RTL")
print("Drone is returning to launch!")
```

---






