# Robot Control System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    USER DEVICES                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Phone      │  │   Laptop     │  │   Tablet     │      │
│  │  (Browser)   │  │  (Browser)   │  │  (Browser)   │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │              │
│         └──────────────────┼──────────────────┘              │
│                            │                                 │
└────────────────────────────┼─────────────────────────────────┘
                             │
                    WiFi / Network
                             │
┌────────────────────────────┼─────────────────────────────────┐
│                  RASPBERRY PI                                 │
│                            │                                  │
│  ┌─────────────────────────▼──────────────────────────────┐  │
│  │              Flask Web Server                          │  │
│  │              (robot_server.py)                         │  │
│  │                                                         │  │
│  │  • Serves HTML joystick interface                      │  │
│  │  • Receives joystick positions (X, Y)                  │  │
│  │  • Runs control loop at 20Hz                           │  │
│  │  • Calculates arcade drive steering                    │  │
│  └──────────────────────┬──────────────────────────────────┘  │
│                         │                                     │
│  ┌──────────────────────▼──────────────────────────────────┐  │
│  │           Robot Control Functions                       │  │
│  │           (robotfuncs.py)                               │  │
│  │                                                          │  │
│  │  • move() - Sets servo targets                          │  │
│  │  • drive() - Controls left/right wheels                 │  │
│  │  • stop() - Emergency stop                              │  │
│  └──────────────────────┬───────────────────────────────────┘  │
│                         │                                     │
│  ┌──────────────────────▼───────────────────────────────────┐ │
│  │         Maestro Servo Controller                         │ │
│  │         (maestro.py)                                     │ │
│  │                                                          │ │
│  │  • USB communication with Maestro board                 │ │
│  │  • setTarget() - Position control                       │ │
│  │  • getPosition() - Read current position                │ │
│  └──────────────────────┬───────────────────────────────────┘ │
│                         │ USB                                 │
└─────────────────────────┼─────────────────────────────────────┘
                          │
            ┌─────────────▼────────────┐
            │  Pololu Maestro Board    │
            │  (Hardware Controller)   │
            └─────────────┬────────────┘
                          │ PWM Signals
            ┌─────────────┴────────────┐
            │                          │
    ┌───────▼────────┐        ┌───────▼────────┐
    │  Left Wheel    │        │  Right Wheel   │
    │  Servo (Ch 1)  │        │  Servo (Ch 0)  │
    └────────────────┘        └────────────────┘


JOYSTICK COORDINATE SYSTEM:
═══════════════════════════

         Y = +1.0 (Forward)
              ▲
              │
              │
X = -1.0 ◄────┼────► X = +1.0
(Left)        │        (Right)
              │
              │
              ▼
         Y = -1.0 (Reverse)

         Center: X=0, Y=0 (STOP)


ARCADE DRIVE ALGORITHM:
═══════════════════════

Input: Joystick position (x, y)

1. Apply deadzone (ignore if |x| < 0.1 or |y| < 0.1)

2. Calculate base speeds:
   forward = y     (vertical movement)
   turn = x        (horizontal movement)

3. Calculate differential steering:
   left_wheel = forward + turn
   right_wheel = forward - turn

4. Normalize if needed:
   if max(|left|, |right|) > 1.0:
       scale both down proportionally

5. Map to servo values:
   servo_value = 6000 + (speed × 1000)
   Range: 5000 (full reverse) to 7000 (full forward)
   Center: 6000 (stop)

Examples:
• Joystick straight up (0, 1): Both wheels = 7000 (forward)
• Joystick straight down (0, -1): Both wheels = 5000 (reverse)
• Joystick right (1, 0): Left=7000, Right=5000 (spin right)
• Joystick up-right (0.7, 0.7): Left=7700, Right=6300 (forward + right turn)


DATA FLOW:
══════════

1. User drags joystick on phone
   ↓
2. JavaScript calculates normalized X, Y (-1 to 1)
   ↓
3. POST to /api/joystick every 50ms
   ↓
4. Flask server updates global variables
   ↓
5. Control loop (background thread) reads variables
   ↓
6. Arcade drive algorithm calculates wheel speeds
   ↓
7. robotfuncs.move() called for each wheel
   ↓
8. maestro.setTarget() sends USB command
   ↓
9. Maestro board generates PWM signals
   ↓
10. Servos rotate wheels
```
