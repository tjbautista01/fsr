# Fork details
This fork has the following modifications done to it:
- Fetching analog values using [ResponsiveAnalogRead library] (https://github.com/dxinteractive/ResponsiveAnalogRead) for less noisy reads
- Joystick inputs for Arduino instead of keyboard using [Arduino Joystick Library] (https://github.com/MHeironimus/ArduinoJoystickLibrary)
- PSX functionaly added using [veroxzik's PSX Library] (https://github.com/veroxzik/arduino-psx-controller)
- Modified React WebUI to support more panels

# Teejusb's FSR Guide
A complete software package for FSR dance pads.  
Join the [discord](https://discord.gg/RamvtwuEF2) for any questions/suggestions

## Features
- React web UI to customize sensitivity 
- Profiles & persistence
- Light support

## Screenshots
<img src="./img/fsr2.gif" width="550">

<img src="./img/fsr1.gif" width="550">


## Requirements
- A [Teensy](https://www.pjrc.com/store/index.html) or Arduino
  - uses native keyboard library for Arduino and Joystick library for Teensy
- Python 3.7+
    - virtualenv
- Node 12+
  - yarn

## Hardware setup
Follow a guide like [fsr-pad-guide](https://github.com/Sereni/fsr-pad-guide) or [fsr](https://github.com/vlnguyen/itg-fsr/tree/master/fsr) to setup your Arduino/Teensy with FSRs.

## Firmware setup
1. Install [Arduino IDE](https://www.arduino.cc/en/software) (skip this if you're using OSX as it's included in Teensyduino)
2. Install [Arduino Joystick Library] (https://github.com/MHeironimus/ArduinoJoystickLibrary) if using an Arduino Leonardo or related clones.
3. Install [Teensyduino](https://www.pjrc.com/teensy/td_download.html) and get it connected to your Teensy and able to push firmware via Arduino IDE

(The next three steps are for Teensy)
4. In Arduino IDE, set the `Tools` > `USB Type` to `Serial + Keyboard + Mouse + Joystick` (or `Serial + Keyboard + Mouse`)
5. In Arduino IDE, set the `Tools` > `Board` to your microcontroller (e.g. `Teensy 4.0`)
6. In Arduino IDE, set the `Tools` > `Port` to select the serial port for the plugged in microcontroller (e.g. `COM5` or `/dev/something`)

7. Load [fsr.ino](./fsr.ino) in Arduino IDE.
8. By default, A0-A7 are the pins used for the FSR sensors in this software, corresponding to 8 buttons on a DDR softpad. To add or remove more, [alter the Sensor array](./fsr/fsr.ino#L491-L500) and [number of buttons you will need](./fsr.ino#L41)
9. Refer to [the](./fsr.ino#L331) [following lines](./fsr/fsr.ino#L606-607) and [the ResponsiveAnalogRead library] (https://github.com/dxinteractive/ResponsiveAnalogRead) to adjust the smoothing of the analog readings.
10. Regarding [the Sensor array](./fsr/fsr.ino#L491-L500), if PSX functionality is not desired, set the second values to PS_INPUT::PS_NONE.
11. Push the code to the board
12. For attaching a PSX cable, refer to [veroxzik's PSX Library] (https://github.com/veroxzik/arduino-psx-controller). Note that if powering via the 7.6v Rumble line, you will need to use a Schottky diode (side with line leading to Arduino VIN pin, other side to PSX cable).

### Testing and using the serial monitor
1. Open `Tools` > `Serial Monitor` to open the Serial Monitor
2. Within the serial monitor, enter `t` to show current thresholds.
3. You can change a sensor threshold by entering numbers, where the first number is the sensor (0-indexed) followed by the threshold value. For example, `3 180` would set the 4th sensor to a threshold of 180.  You can change these more easily in the UI later.
4. Enter `v` to get the current sensor values.
5. Putting pressure on an FSR, you should notice the values change if you enter `v` again while maintaining pressure.


## UI setup
1. Install [Python](https://www.python.org/downloads/). On Linux you can install Python with your distribution's package manager. On some systems you might have to additionally install the python3 header files (usually called `python3-dev` or similar).
2. Install [Node](https://nodejs.org/en/download/)
    - Install [yarn](https://classic.yarnpkg.com/en/docs/install#windows-stable). A quick way to do this is with NPM: `npm install -g yarn`
3. Within [server.py](./webui/server/server.py), edit the `SERIAL_PORT` constant to match the serial port shown in the Arduino IDE (e.g. it might look like `"/dev/ttyACM0"` or `"COM1"`)
    - You also may need to [modify](https://github.com/teejusb/fsr/pull/1#discussion_r514585060) the `sensor_numbers` variable for the number of panels used.
4. Edit [this line in App.js](./webui/src/App.js#L25) according to number of sensors in panel. Note that you will need to run 'npm run build' within the webui folder for every change done (webserver can be kept running, just refresh page after building changes).
5. Open a command prompt (or terminal) and navigate to `./webui/server` with `cd webui/server`
6. Run `python -m venv venv` (you may need to replace `python` with `py` on Windows or potentially `python3` on Linux)
7. Run `venv\Scripts\activate` (on Linux you run `source venv/bin/activate`)
8. Run `pip install -r requirements.txt` to install dependencies (might need to use `pip3` instead of `pip` on Linux)
9. Then move to the `./webui` directory by doing `cd ..`
10. Run `yarn install && yarn build && yarn start-api`
    - On Linux, you'll also need to edit the `start-api` script in `./webui/package.json` to reference `venv/bin/python` instead of `venv/Scripts/python`

The UI should be up and running on http://localhost:5000 and you can use your device IP and the port to reach it from your phone (e.g. http://192.168.0.xxx:5000 )


## Troubleshooting 
- If you use localhost in your browser and if the UI looks choppy, try using your local IP instead
- If you see the following error, ensure the "Serial Monitor" isn't already open in Arduino IDE `serial.serialutil.SerialException: [Errno 16] could not open port /dev/cu.usbmodem83828101: [Errno 16] Resource busy: '/dev/cu.usbmodem83828101`
- If you notice that your input is delayed and perhaps that delay increases over time, you can sometimes rectify that by restarting the server. Close your `start-api` window and run it again.

## Tips
### Make a desktop shortcut (Windows)
Create a new text file called `start_fsrs.bat` and place it on your desktop.
```bat
start "" http://YOUR_PC_NAME_OR_IP:5000/
cd C:\Users\YourUser\path\to\fsr\webui
yarn start-api
```
Now you can just click on that file to open the UI and start the server.
