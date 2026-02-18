# Robot Flask Server with Joystick Control

Web-based joystick controller for your Maestro servo-controlled robot. Access from any device on your network!

## 📁 Files Included

- `robot_server.py` - Main Flask server application
- `templates/index.html` - Web joystick interface
- `robotfuncs.py` - Robot control functions (from your existing code)
- `README.md` - This file

## 📋 Prerequisites

### Hardware
- Raspberry Pi (tested on Pi 3/4)
- Pololu Maestro servo controller (connected via USB)
- Robot with continuous rotation servos on channels 0 (rWheel) and 1 (lWheel)

### Software
```bash
# Install Python dependencies
pip3 install flask

# Make sure you have the maestro.py module
# (This should already be in your project)
```

## 🚀 Installation

1. **Upload all files to your Raspberry Pi:**
   ```bash
   # Create a directory for the project
   mkdir ~/robot_control
   cd ~/robot_control
   
   # Copy files here:
   # - robot_server.py
   # - robotfuncs.py
   # - maestro.py (your existing module)
   # - templates/index.html (must be in templates/ subdirectory)
   ```

2. **Verify file structure:**
   ```
   ~/robot_control/
   ├── robot_server.py
   ├── robotfuncs.py
   ├── maestro.py
   └── templates/
       └── index.html
   ```

3. **Make the server executable:**
   ```bash
   chmod +x robot_server.py
   ```

## 🎮 Running the Server

1. **Find your Raspberry Pi's IP address:**
   ```bash
   hostname -I
   ```
   Example output: `192.168.1.100`

2. **Start the server:**
   ```bash
   python3 robot_server.py
   ```

3. **Access the joystick interface:**
   - From the Pi itself: `http://localhost:5000`
   - From phone/laptop on same network: `http://192.168.1.100:5000`
     (Replace with your actual IP)

## 🕹️ Using the Joystick

### Controls
- **Joystick**: Drag to control robot movement
  - **Up/Down**: Forward/Reverse
  - **Left/Right**: Steering
  - **Center**: Stop
  
- **Stop Button**: Emergency stop (returns joystick to center)
- **Home Button**: Move all servos to center position

### How It Works

**Arcade Drive Mode:**
- Vertical axis controls forward/backward movement
- Horizontal axis controls left/right steering
- The joystick calculates differential steering automatically
  - Moving up-right = forward + gentle right turn
  - Moving left = spin left in place
  - Center = complete stop

**Technical Details:**
- Joystick position is normalized to -1.0 to 1.0 for both X and Y
- Values are mapped to servo range 5000-7000 (center = 6000)
- 0.1 deadzone prevents drift when near center
- Updates sent at 20Hz (50ms intervals)
- Left wheel and right wheel controlled independently

## 🔧 Configuration

### Adjust Control Sensitivity

Edit `robot_server.py`:

```python
# Line ~18: Deadzone (how close to center before stopping)
DEADZONE = 0.1  # Increase for larger deadzone (0.0-0.5)

# Line ~141: Update rate
time.sleep(0.05)  # Decrease for faster updates (minimum ~0.02)
```

### Adjust Servo Range

If your wheels need different values:

```python
# In map_joystick_to_servo() function (~30-45)
# Change the range from [5000, 7000] to your values
servo_value = int(6000 + (value * 1000))  # Adjust the 1000 multiplier
```

### Reverse Wheel Direction

If a wheel spins backwards:

```python
# In calculate_arcade_drive() function (~103-104)
# Change reversed_dir parameter:
left_servo = map_joystick_to_servo(left, reversed_dir=True)   # Change to True
right_servo = map_joystick_to_servo(right, reversed_dir=False) # Change to False
```

## 🐛 Troubleshooting

### Can't Connect to Server
1. Check Raspberry Pi is on the same WiFi network
2. Verify IP address: `hostname -I`
3. Check firewall: `sudo ufw allow 5000`
4. Try accessing from Pi itself first: `http://localhost:5000`

### Robot Not Moving
1. Check Maestro is connected: `lsusb` should show Pololu device
2. Verify servo channels in robotfuncs.py match your wiring
3. Check servo power supply
4. Look at terminal output for error messages

### Joystick Not Responding
1. Open browser console (F12) to check for JavaScript errors
2. Check network tab for failed API requests
3. Verify `/api/joystick` endpoint is receiving data

### Robot Moves Erratically
1. Increase DEADZONE value (line ~18)
2. Decrease update frequency (increase sleep time, line ~141)
3. Check servo wiring and power

## 📱 Mobile Usage Tips

- Add to home screen for app-like experience
- Use landscape mode for larger joystick
- Ensure good WiFi signal
- Keep browser tab active (background tabs may throttle updates)

## 🔒 Running on Startup (Optional)

To start the server automatically when Pi boots:

1. Create a systemd service:
   ```bash
   sudo nano /etc/systemd/system/robot-server.service
   ```

2. Add this content:
   ```ini
   [Unit]
   Description=Robot Joystick Server
   After=network.target

   [Service]
   Type=simple
   User=pi
   WorkingDirectory=/home/pi/robot_control
   ExecStart=/usr/bin/python3 /home/pi/robot_control/robot_server.py
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. Enable and start:
   ```bash
   sudo systemctl enable robot-server
   sudo systemctl start robot-server
   ```

4. Check status:
   ```bash
   sudo systemctl status robot-server
   ```

## 🎯 API Endpoints

For custom integrations:

- `POST /api/joystick` - Update joystick position
  ```json
  {"x": 0.5, "y": 0.75}
  ```

- `GET /api/status` - Get current status
  ```json
  {"connected": true, "x": 0.0, "y": 0.0, "running": true}
  ```

- `POST /api/stop` - Emergency stop

- `POST /api/home` - Move to home position

## 📊 Performance

- Update Rate: 20Hz (50ms)
- Latency: ~30-100ms depending on network
- Concurrent Users: Supports multiple viewers (only one should control)

## ⚠️ Safety Notes

- Always have clear space around robot when testing
- Keep STOP button accessible
- Start with slow movements to test
- Monitor servo temperatures during extended use
- Use appropriate power supply for servos

## 🤝 Credits

Created for robot control project using:
- Flask web framework
- Pololu Maestro servo controller
- Your existing robotfuncs.py module

## 📝 License

Free to use and modify for your project!
