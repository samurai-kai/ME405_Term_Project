# Controller Design
An agnostic PID controller class is used so It can be applied to a variety of 
inputs and outputs. This controller is primarily used to set effort values for 
each motor based on error. This error is either positional error dictated by the 
offset from the IR sensor centroid, or angular position error based on IMU 
heading angles.  

The documented low-level controller class can be found [here](controller.md). 

<br> 

![Controller diagram](./image/MotorControlBlockDiagram.png)