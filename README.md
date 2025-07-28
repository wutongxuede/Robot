# Robot
#Focus on discussions and sharing about Robot-Wall making or other DIY projects.

#The source code of the software is from the content published by Simon Bluett chillibasket on GitHub (Click to jump to GitHub for viewing: https://github.com/chillibasket/walle-replica)
## Basic introduction of the Wall-E model
1. Each eye can be independently controlled to rise and fall by a servo.

2. Each eye has space for a small camera. (We haven't done this part yet, so if you are interested, you can try it first)

3. The head can be turned left and right using a servo. (Due to the heavy weight of the head, it is recommended to use a hollow printing for the head)

4. Both joints of the neck are driven by servos, which allows the head to look up/down and raise/lower.

5. Each arm has a servo at the shoulder to move the arm up and down.

6. The arms are composed of pressure-fit joints, hands and fingers, and can be posed manually.

7. The tracks are fully 3D printed and can be driven using two 12V DC reduction motors.
**Flashing the program**
1. Please write the program to the Raspberry Pi and Arduino first, test it, and then install the circuit board into the robot.
2. The programming of the robot can be divided into two main parts; the Arduino code, and the web server on the Raspberry Pi.
Download/clone the folder wall-e from the online repository. (To add a voice recognition module, use our modified wall-e.ino main program, in the walle-replica-3.0 - modified version program folder)
Open wall-e.ino in the Arduino IDE;. In addition, the MotorController.hpp and Queue.hpp files will also automatically open on different tabs in the IDE.
Install the Adafruit_PWMServoDriver.h library
Go to Sketch -> Include Library -> Manage Libraries
Search for Adafruit PWM Servo Driver.
Install the latest version of the library
Connect the computer and the controller with a USB data cable. Make sure to select the correct board and port in the "Tools" menu.
Upload the wall-e.ino main program
**Test**
Upload the main program to the Arduino. While the Arduino is still connected to the computer, connect a 12V (3S) battery and power it on.
Open the serial port detection (the button on the upper right of the Arduino IDE), set the baud rate to 115200.
Now test the movement of Wally, send characters "w", "a", "s" or "d", which respectively represent Wally moving forward, left, backward or right. Sending 'q' can stop the movement.
Now test the head movement of Wally, send letters "j", "l", "i" or "k" to tilt the head left or right, and raise or lower the eyes. At this stage, the joints controlled by the servo motors may have a wider range than the actual due to the lack of setting for the stroke. This problem will be solved by performing the following servo calibration steps.
Servo Calibration
Download/clone the "wall-e_calibration" folder from the online repository.
Open wall-e_calibration.ino in the Arduino IDE.
Upload the program to the Arduino, open the serial monitor, and set the baud rate to 115200.
This program is used to calibrate the movement range required to move each servo motor to the corresponding position, the maximum and minimum PWM pulse widths. The standard LOW and HIGH positions of each servo are as follows.
Before installing the servo motors, it is necessary to ensure that the angles of the servo motors are correct (the range can be referred to the following figure), and then install the servo arms. Since the servo motors can only rotate 180 degrees, if they are fixed at the wrong angle, they will not be able to control the joints correctly.

After starting the program, open the serial monitor. After 2-3 seconds, a message should appear indicating that the LOW position of the first servo motor (head rotation) is ready for calibration.
Send the characters "a" and "d" to control the robot's movement forward and backward, within the range of -10 to +10. If more precise control is needed, use the characters "z" and "c" to move the robot, within the range of -1 to +1.
Once the servo is positioned at the correct position (as shown in the figure below), send the character "n" to continue the calibration steps. It will move to the HIGH position of the same servo, and then the 7 servos in the robot will repeat this process.
When all the joints have been calibrated, the program will output an array containing the calibration values to the serial monitor.
Copy this array and paste it into lines 144 to 150 of the wall-e - ino program. The array should look similar to the following:

int preset[][2] =  {{410,120},  // head rotation
                    {532,178},  // neck top
                    {120,310},  // neck bottom
                    {465,271},  // eye right
                    {278,479},  // eye left
                    {340,135},  // arm left
                    {150,360}}; // arm right
                    
**Battery level detection (optional)**
Note: When using batteries to power the robot, it is necessary to detect the battery level in real time. Over-discharge may cause battery damage, and insufficient power supply may damage the SD card of the Raspberry Pi. 
When using the battery level detection function on the Arduino, please connect the following resistors as shown in the following diagram and make the connections.

The resistor (voltage divider) reduces the 12V voltage to below 5V, allowing the Arduino to measure the voltage using its analog pins. The recommended resistor values are R1 = 100kΩ and R2 = 47kΩ.
In Arduino, remove line 54 of the main program wall-e.ino.
If you use different resistor values, please change the value of the voltage divider gain factor in line 54 of the program according to the formula: POT_DIV = R2 / (R1 + R2).
The program should now automatically check the battery level every 10 seconds, and this level will be displayed in the "Status" section of the Raspberry Pi network interface.
OLED display (optional) (Contributed by hpkevertje on GitHub) Notes for Attention

A 1.3-inch small OLED display can be integrated, and the battery level of the Wally robot can be displayed on the battery indicator panel. This function requires enabling the aforementioned battery detection circuit. The screen will be updated each time the battery level is calculated. 
This function uses the u8g2 display library in the page mode. 
On the Arduino UNO, you might receive a warning about high memory usage, but this warning can be ignored. 
To use the oLed display function on Arduino, connect the i2c oLed display (as shown in the picture) to the i2c bus of the servo motor module. 
Input image description 
Install the U8g2 library in the Arduino Library Manager:
In Sketch -> Include Library -> Manage Libraries
Search for U8gt (creator: oliver)
Install the latest version of the library
Define the OLED in the Arduino main program wall-e.ino.
If you are using another display method supported by the library, you can change the constructor on line 78 according to the documentation on the library reference page. The default is for the sh1106_128x64_name display.
Add custom servo actions (optional)
The source code comes with two animations that can be copied to replicate the scenes in the movie; 
The eye movements of robot Wally when it is activated, as well as a series of actions when robot Wally curiously looks around.

Starting from version 2.7, users can now add custom actions more easily. 
Open the "animations.ino" file, which is located in the same folder as the main Arduino program.
Each action command contains the position you want all the servos to move to, as well as the time the animation should wait before moving to the next instruction.
Actions can be added by inserting an additional case in the switch statement. Place the additional code above the default section in the empty space. For example: case 3:
// --- Title of your new motion sequence ---
//          time,head,necT,necB,eyeR,eyeL,armL,armR
queue.push({  12,  48,  40,   0,  35,  45,  60,  59});
queue.push({1500,  48,  40,  20, 100,   0,  80,  80});
// Add as many additional movements here as you need to complete the animation
// queue.push({time, head rotation, neck top, neck bottom, eye right, eye left, arm left, arm right})
break;
The time should be measured in milliseconds (for example, 3.5 seconds = 3500).
The position command for the servo needs to be an integer between 0 and 100, where 0 = LOW, 100 = HIGH, and the servo position calibrated in wall-e_calibration is the same.
If you want to disable a specific servo, you can set its value to -1.
Raspberry Pi Settings
Hardware Settings
Connect the power cable of the Raspberry Pi to the USB power output port of the 12V to 5V converter.
Connect the Arduino to the Raspberry Pi using a USB connection.
If you have a Raspberry Pi camera, connect the cable to the CSI camera connector.
For convenience in setting up and installing, you can insert the display into the HDMI port and connect a USB keyboard and mouse. Or connect and configure the Raspberry Pi from another computer via SSH.
Basic Settings
Install the Raspberry Pi to run the latest version of the Raspberry Pi OS Desktop. Installation instructions can be found on the Raspberry Pi website.
Ensure the Raspberry Pi is connected to the internet.
Open "Terminal" on the Raspberry Pi.
Clone the repository to the root directory of the Raspberry Pi:
cd ~

git clone https://github.com/chillibasket/walle-replica.git

The web interface settings can be configured by editing the "config.py" file:
Open the configuration file: nano ~/wall-replica/web_interface/config.py
On line 14, you can change the password for the web interface. The default password is "walle".
On line 15, you can set the default serial port for connecting to the Arduino. You can use the command "dmseg | grep tty" to find a list of all connected serial ports.
On lines 16 and 17, you can configure whether the Arduino and camera should automatically connect when starting the web server.
After completing the editing of the configuration, run the installation script and set all necessary libraries (it may take some time): cd ~/walle-replica

sudo chmod +x ./raspi-setup.sh

sudo ./raspi-setup.sh
Using the Web Server
If the installation is successful, when the Raspberry Pi is powered on, the web server will automatically start. This is accomplished using the Systemd service.
On the Raspberry Pi, you can view the web interface at http://localhost:5000
To view the interface from another computer on the same WiFi network, first you need to use the command hostname -I to determine the current IP address of the Raspberry Pi on the network
To access the web interface, open a browser on any computer/device in the same network and enter the IP address of the Raspberry Pi, followed by :5000. For example, 192.168.1.10:5000
To start controlling the robot, first ensure that the serial communication with Arduino has been initiated. To do this, go to the Settings tab in the web interface, select the correct serial port from the drop-down list, and then press the Reconnect button. If the configuration settings are correct, the device will automatically start.
Here are some commands for controlling the web interface: 
Stop the automatic web interface service: sudo systemd stop wall.service 
Disable startup on boot: sudo systemctl disable wall.service 
Re-enable the startup at boot time: sudo systemd enable wall.service 
Use the Blocky control (contributed by dkrey on GitHub) 
Starting from version 3.0, there is a new option in the web interface that allows robots to be controlled using graphical programming with drag-and-drop scripting languages. Just drag the operations you want to execute from the left sidebar and drop them into the editor area. For example, you can control Wally, rotate the motor, and play audio.
For commands to drive the motor, you may need to adjust the parameters from lines 25 to 28 at the bottom of the "config.py" file to ensure the correct speed and turning amount.
Using Blocky control (contributed by: dkrey on GitHub)
Starting from version 3.0, a new option has been added to the web interface, allowing robots to be controlled through graphical programming with drag-and-drop scripting languages. Just drag the operations you want to execute from the left sidebar and drop them into the editor area. For example, you can control Wally, rotate the motor, and play audio.
For commands to drive the motor, you may need to adjust the parameters from lines 25 to 28 at the bottom of the "config.py" file to ensure the correct speed and turning amount.
Adding video stream (optional)
The web server automatically supports any CSI connector connected to the Raspberry Pi and a ribbon cable camera. (Currently, USB network cameras are not supported.) 
Add new sound (optional)
By default, the Raspberry should automatically choose whether to output audio to the HDMI port or the headphone jack. However, the following command can be used to ensure that it always uses the headphone jack: amixer cset numid= 31
Make sure that all the sound files you want to use are of the *.wav format. Most music editors should be able to convert sound files to this format.
Modify the file name to have the format [group name][file name][sound duration (milliseconds)]. For example: voice_eva_1200.wav. In the web interface, audio files will be grouped by "group name" and sorted alphabetically.
Upload the sound files to the following folder on the Raspberry Pi: ~/wall -replica/web_interface/static/sounds/
When reloading the page, all the files should appear in the web interface. If the files do not appear, it may be necessary to change the permissions required to access this folder: sudo chmod -R 755 ~/wall -replica/web_interface/static/sounds

Set Raspberry Pi as a WiFi hotspot (optional)
The Raspberry Pi can be configured as a WiFi hotspot network. The computer/mobile phone/tablet controlling the robot can directly connect to this network.
To set up the WiFi hotspot, we will use the RaspAP project, which is responsible for all the configurations and tools to make the system work.
The following operations are based on the quick installation guide:
Update Raspian, kernel and firmware (and then restart): sudo apt-get update

sudo apt-get dist-upgrade

sudo reboot now

Make sure to set the correct WiFi country in the localization options of raspi-config: sudo raspi-config
Run the quick installation program: curl -sL https://install.raspap.com | bash
For the first few yes/no prompts during the installation process, enter "y" (yes) to accept all recommended settings.
The last two prompts (advertising interception and next) are not necessary, so you can enter "n" (no).
Restart the Raspberry Pi: sudo Reboot
Now the Raspberry Pi should start the WiFi hotspot network in the following way:
SSID (wifi name): raspi-webgui
Password: ChangeMe
After connecting to the WiFi network via a computer, mobile phone or tablet, enter the following address in the browser to open the Wally's network interface: http://10.3.141.1:5000
It is recommended to enter the WiFi configuration page at http://10.3.141.1 to modify the WiFi name and password. The default username is "admin" and the password is "secret".
Click on the "Hotspot" item in the left column. In the "Basic" tab, you can change the WiFi network name, and in the "Security" tab, you can change the WiFi password.
To change the admin password of the WiFi settings management interface, click the "admin" icon in the upper right corner of the interface.
