# Paulus v0.1

## Two-Wheel Self-Balancing Robot — Build and Bring-Up Manual

### 1. Objective

Paulus v0.1 is a two-wheel dynamically self-balancing mobile robot based on an ESP32.

Version 0.1 is intended to:

* balance autonomously on two wheels;
* recover from small disturbances;
* measure wheel rotation through quadrature encoders;
* drive forward and backward under closed-loop control;
* turn by differential wheel control;
* stop motor power through a physical emergency-stop circuit;
* expose a clean command interface for later Wi-Fi, Raspberry Pi, vision, speech and autonomous behavior.

Version 0.1 does **not** yet need a Raspberry Pi, camera, microphone or AI stack.

## The purchased hardware includes two JGB37-520 12 V encoder motors, a BNO085 IMU, two BTS7960 motor drivers, an 11.1 V 3S battery, an XL4016 regulator, emergency stop, automotive relay, 15 A fuse, aluminum 2020 extrusion, acrylic, wiring and related hardware.

# 2. Safety architecture

Before building anything else, establish one rule:

**The emergency stop must work without software.**

Do not implement the E-stop as an ESP32 GPIO that says `motor_pwm = 0`.

The motor-power path should be:

```text
3S battery
    │
    ▼
  XT60
    │
    ▼
15 A fuse
    │
    ├──────────────────────► logic power / XL4016
    │
    ▼
relay contact
    │
    ▼
BTS7960 motor power
    │
    ▼
motors
```

The relay coil circuit should be approximately:

```text
Battery +
    │
    ▼
E-STOP NC
    │
    ▼
relay coil
    │
    ▼
Battery -
```

Place the 1N4007 across the relay coil:

```text
       relay coil
   + ─────(   )───── -
   │                 │
   └──────|<|────────┘
          diode
```

The **cathode/striped side goes toward coil positive**.

The emergency button you bought is a locking, normally-closed 22 mm unit, and the relay is a 12 V automotive relay.
For the common four-pin automotive relay numbering, you will usually see:

```text
30  = common power contact
87  = normally-open output
85  = coil
86  = coil
```

Verify the diagram printed on **your relay** rather than blindly trusting those numbers.

The fuse belongs close to the battery, not beside the motor drivers.

The XT60 is your convenient master disconnect. The E-stop kills motor power quickly; disconnecting the XT60 makes the whole robot electrically dead.

---

# 3. Prepare the power system

Do this before mounting electronics to the chassis.

Do **not** connect the ESP32 yet.

Connect the battery to the input of the XL4016 regulator through temporary safe wiring.

Use a multimeter.

Adjust the XL4016 potentiometer until:

```text
Vout = approximately 5.0 V
```

Measure it twice.

Then remove battery power.

Do not adjust the regulator while the ESP32 is attached unless you have a specific reason to.

Later, the 5 V output can feed the ESP32 development board through its appropriate **5V/VIN input**, depending on your exact board.

Do not feed 5 V into the ESP32's 3.3 V pin.

The motor drivers receive battery voltage directly. The XL4016 is for the logic side.

Conceptually:

```text
              11.1–12.6 V
                  │
             ┌────┴─────┐
             │          │
             ▼          ▼
         motor rail    XL4016
             │          │
             │         5 V
             │          │
         BTS7960s     ESP32
```

All electronic subsystems ultimately need a common electrical ground:

```text
battery -
ESP32 GND
XL4016 GND
BTS7960 logic GND
encoder GND
IMU GND
```

Avoid sending heavy motor current through thin logic-ground wiring. Bring the power grounds back toward a common distribution point.

---

# 4. Bench-test one motor

Do not assemble the complete robot yet.

Take:

```text
1 JGB37 motor
1 BTS7960
ESP32
battery
```

Keep the wheel clear of the table.

The common IBT-2/BTS7960 module generally exposes:

```text
B+ / B-      motor supply
M+ / M-      motor
VCC          logic supply
GND          logic ground

RPWM
LPWM
R_EN
L_EN
```

Verify the labels printed on your actual board.

Connect:

```text
battery + → B+
battery - → B-

motor → M+, M-

5 V logic → VCC
logic GND → GND
```

For initial testing, enable the driver according to the particular IBT-2 board's pinout.

Then send PWM to one direction at a time.

Never command both directions simultaneously at nonzero duty.

Conceptually:

```text
forward:
    RPWM = PWM
    LPWM = 0

reverse:
    RPWM = 0
    LPWM = PWM

stop:
    RPWM = 0
    LPWM = 0
```

Start at approximately **10% duty cycle**, not 100%.

The ESP32's LEDC peripheral is designed to generate hardware PWM and supports multiple PWM channels, so it is well suited to controlling the motor drivers.

A starting PWM frequency around:

```text
15–20 kHz
```

is reasonable for this application.

### Acceptance test

You should be able to command:

```text
STOP
FORWARD 10%
FORWARD 20%
STOP
REVERSE 10%
REVERSE 20%
STOP
```

without ESP32 resets, driver overheating, unexpected direction changes or strange noise.

Repeat the complete test with the second motor and second BTS7960.

---

# 5. Characterize each encoder

Do this before balancing.

Your JGB37 motors have quadrature encoders, but the Mercado Livre description does not clearly specify their exact CPR/PPR or encoder electrical output.

Therefore:

**Do not assume that the encoder outputs are 3.3 V-safe.**

Determine:

```text
encoder supply voltage
encoder A output voltage
encoder B output voltage
PPR/CPR
```

before permanently connecting them to the ESP32.

If the encoder outputs rise to approximately 5 V, do not connect them directly to ESP32 GPIOs. Use proper level shifting or resistor dividers.

Once electrically safe, connect each encoder's:

```text
A → ESP32 GPIO
B → ESP32 GPIO
GND → common GND
```

Use interrupts or the ESP32's appropriate pulse-counting mechanism.

Turn the wheel slowly by hand.

Print:

```text
left_count
right_count
```

Then determine the sign convention:

```text
wheel moving robot forward = positive ticks
wheel moving robot backward = negative ticks
```

If one motor mechanically faces the opposite direction—as it probably will—the raw encoder sign may naturally be reversed. Fix this in software so both wheels share the same robot-centric convention.

### Acceptance test

When you manually roll both wheels as though the robot were moving forward:

```text
left velocity  > 0
right velocity > 0
```

When moving backward:

```text
left velocity  < 0
right velocity < 0
```

Do not proceed until this is true.

---

# 6. Bring up the BNO085

The BNO085 contains accelerometer, gyroscope, magnetometer and onboard sensor-fusion processing.

There is one important caveat for this build:

**BNO08x I²C communication has known compatibility problems with ESP32-family chips.** Adafruit explicitly documents unreliable behavior between the BNO085's I²C implementation and ESP32/ESP32-S3 devices.

Therefore, treat communication mode as an implementation choice rather than assuming I²C will automatically be perfect.

If your generic breakout exposes the necessary UART/SPI configuration pins, UART or SPI may ultimately be preferable.

If you initially try I²C and it works reliably, fine.

If you see:

```text
random sensor hangs
timeouts
I²C bus resets
missing packets
controller freezes
```

do not spend three weeks tuning the PID.

Fix the IMU transport first.

The BNO085 UART-RVC mode is particularly simple and produces yaw, pitch, roll and acceleration reports at 100 Hz.

For a more sophisticated balance controller, using a mode that exposes both:

```text
orientation
angular velocity
```

is preferable.

For balance, you primarily care about:

```text
pitch angle θ
pitch angular velocity θ̇
```

You do **not** need magnetometer-based absolute heading to keep the robot upright.

In fact, avoiding magnetic heading in the inner balance loop is desirable because there are powerful electric motors and large current paths nearby.

Mount the BNO085 rigidly to the chassis.

Do not mount it on something flexible.

Keep it away from:

```text
motor bodies
motor power cables
relay
large loops carrying current
```

as much as practical.

### Acceptance test

With the robot motionless:

```text
pitch ≈ constant
```

Tilt it forward:

```text
pitch changes smoothly in one direction
```

Tilt it backward:

```text
pitch changes smoothly in the opposite direction
```

Return it upright:

```text
pitch ≈ original value
```

Write down the sign convention.

For example:

```text
forward lean = positive pitch
backward lean = negative pitch
```

The exact sign does not matter.

Consistency does.

---

# 7. Build the chassis

Now build the mechanical structure.

Do not start by making it pretty.

Start by making it:

```text
rigid
symmetric
adjustable
serviceable
cat-resistant
```

The aluminum extrusion is the load-bearing structure. The 3 mm acrylic is mainly an electronics mounting deck, not the primary structural element.

You have three 500 mm lengths of 2020 extrusion.

A reasonable starting geometry is roughly:

```text
height:          300–400 mm
wheel spacing:   200–250 mm
width of body:   150–200 mm
```

Exact measurements depend on the supplied wheel and motor brackets.

A useful structure resembles:

```text
       ┌─────────────┐
       │ electronics │
       ├─────────────┤
       │             │
       │   battery   │
       │             │
       ├─────────────┤
       │             │
    O──┴─────────────┴──O
```

Prefer two vertical rails rather than one if possible because the resulting frame will resist twisting better.

Mount the motor brackets firmly at the bottom.

The motor shafts should be:

```text
parallel
horizontal
same height
square relative to chassis
```

Both wheels must rotate freely without touching the frame.

The wheel axle defines the robot's pivot line.

---

# 8. Position the battery

The battery is one of the heaviest components and therefore becomes a useful tuning parameter.

Use the straps you bought rather than permanently fixing it immediately.

Start with the battery approximately:

```text
100–150 mm above the wheel axle
```

and centered left-to-right.

The center of mass should be above the axle.

A somewhat elevated center of mass often makes the falling dynamics slower and easier to control than putting all mass immediately beside the axle.

Do not obsess over the perfect position yet.

That is why the mount is adjustable.

---

# 9. Install the acrylic electronics deck

Cut or drill the 30 × 30 cm, 3 mm acrylic sheet as needed.

Your replacement sheet is much more appropriately sized than the original 60 cm plate.

When drilling acrylic:

```text
support it underneath
drill slowly
do not force the bit
avoid holes extremely close to an edge
use washers
do not overtighten screws
```

Mount:

```text
ESP32
BNO085
BTS7960 #1
BTS7960 #2
XL4016
relay if appropriate
power distribution
```

Leave enough space to reach every terminal screw.

Do not bury the emergency-stop button behind wiring.

You should be able to hit it instantly.

---

# 10. Route the wiring

Separate the robot mentally into two wiring classes.

### High-current wiring

```text
battery
fuse
relay contacts
BTS7960 power
motor wires
```

Use the heavier 16 AWG wire here where practical.

### Signal wiring

```text
encoder A/B
IMU
PWM
driver enables
UART/I²C
```

Use the smaller Dupont/signal wiring.

Do not bundle encoder/IMU wires tightly against motor wires if you can avoid it.

Avoid dangling loops.

Use heat-shrink where appropriate.

Secure every cable so a moving wheel—and eventually a curious cat—cannot grab it.

The goal is:

```text
nothing loose
nothing capable of touching a wheel
nothing capable of shorting against aluminum
```

---

# 11. Implement the software state machine

Before implementing PID, create a robot state machine.

Use something conceptually like:

```text
BOOT
  ↓
DISARMED
  ↓
ARMED
  ↓
BALANCING

BALANCING → FALLEN
BALANCING → FAULT
BALANCING → DISARMED
```

The robot should start **DISARMED** every time it boots.

Do not automatically start the motors simply because power appeared.

Only enter BALANCING if:

```text
IMU is responding
pitch is plausible
encoder inputs are valid
E-stop allows motor power
robot is approximately upright
```

For example:

```text
abs(pitch) < 10°
```

before arming.

Immediately stop motor commands if:

```text
abs(pitch) > 30–40°
```

because at that point Paulus has fallen.

Also stop if:

```text
IMU data becomes stale
control loop misses deadlines repeatedly
sensor returns invalid values
software watchdog triggers
```

---

# 12. Perform the most important sign test

This test prevents the classic self-balancing-robot disaster.

Hold Paulus upright with the wheels touching the ground but **do not let go**.

Lean him slightly forward.

Ask:

> Which direction must the wheels move to save him?

Answer:

**Forward.**

The wheels must move underneath the falling center of mass.

Therefore:

```text
robot falls forward
        ↓
controller detects forward pitch
        ↓
wheels drive forward
```

Likewise:

```text
robot falls backward
        ↓
wheels drive backward
```

If your controller does the opposite:

**STOP.**

Do not tune gains.

Do not increase PWM.

Reverse the motor/controller sign.

A controller with the wrong feedback sign is positive feedback.

It will enthusiastically accelerate the failure.

---

# 13. Implement the inner balance controller

Start without encoder-based velocity control.

The simplest useful inner controller is essentially PD:

```text
error = target_angle - pitch

motor_command =
    Kp × error
  - Kd × pitch_rate
```

Initially:

```text
target_angle = 0°
Ki = 0
```

Do not start with a complicated PID.

Use:

```text
P = correct angular displacement
D = damp angular motion
```

The basic loop is:

```text
read IMU
    ↓
get pitch
get pitch rate
    ↓
calculate balance command
    ↓
apply same basic torque to both motors
```

Conceptually:

```text
u = Kp(θtarget - θ) - Kd θ̇

left  = u
right = u
```

Clamp the output.

For the first test, restrict maximum motor command aggressively:

```text
20–30% PWM
```

Increase it only when necessary.

---

# 14. Tune the balance controller

Put the robot somewhere with room to fall safely.

Not:

```text
beside a monitor
beside a glass table
beside the cat
beside your feet
```

Keep your hand near—but not on—the E-stop.

Start with small `Kp`.

The robot will probably simply fall.

Increase `Kp`.

Eventually the wheels will start meaningfully chasing the center of mass.

Too little `Kp`:

```text
slow response
falls without fighting hard enough
```

Too much `Kp`:

```text
violent rapid oscillation
jerky movement
```

Now introduce `Kd`.

Increase `Kd` until the oscillation becomes damped.

Too little `Kd`:

```text
oscillates repeatedly
```

Too much `Kd`:

```text
sluggish
noisy
buzzing/jittering
```

Do not add integral gain yet.

Your first meaningful milestone is:

```text
Paulus balances for 5 seconds.
```

Then:

```text
10 seconds.
```

Then:

```text
30 seconds.
```

Then:

```text
60 seconds.
```

Only when he can reliably balance should you proceed.

---

# 15. Find the real upright angle

Mechanical construction is never perfectly symmetric.

The actual equilibrium point may not be:

```text
0.000°
```

You may discover that the robot consistently creeps forward unless:

```text
target_angle = -0.8°
```

or similar.

That is normal.

Call this value:

```text
angle_trim
```

Do not use a huge integral term to compensate for a mechanical zero offset.

Calibrate the equilibrium instead.

---

# 16. Add encoder velocity measurement

Convert encoder ticks into wheel velocity.

Once CPR and wheel circumference are known:

```text
revolutions =
    ticks / counts_per_revolution
```

and:

```text
wheel_distance =
    revolutions × wheel_circumference
```

Velocity becomes approximately:

```text
distance_change / time_change
```

Calculate:

```text
v_left
v_right
```

Then the approximate robot translational velocity is:

```text
v = (v_left + v_right) / 2
```

Filter the measurement lightly if necessary.

Do not apply aggressive filtering that introduces large latency.

---

# 17. Add the outer velocity loop

A balancing robot moves forward by intentionally leaning forward.

Therefore the velocity controller does **not** directly command motor PWM.

Instead:

```text
desired velocity
      ↓
velocity controller
      ↓
desired pitch
      ↓
balance controller
      ↓
motor torque
```

Conceptually:

```text
velocity_error =
    target_velocity - measured_velocity
```

Then:

```text
target_angle =
    Kpv × velocity_error
  + Kiv × integral(velocity_error)
```

Clamp the target lean.

For initial operation, something on the order of only a few degrees is enough.

For example:

```text
-5° ≤ target_angle ≤ +5°
```

Now commanding:

```text
target_velocity = +0.2 m/s
```

causes Paulus to lean forward slightly.

The balance controller then drives the wheels forward to maintain that lean.

When the desired velocity returns to zero, the outer loop makes him recover to stationary balance.

---

# 18. Add steering

Turning is differential.

Take the balance output:

```text
u
```

and a steering command:

```text
turn
```

Then:

```text
left_motor  = u - turn
right_motor = u + turn
```

The balance component remains common to both wheels.

The steering component opposes between wheels.

Now Paulus can:

```text
stand
drive forward
drive backward
turn left
turn right
rotate approximately in place
```

---

# 19. Recommended control architecture

The completed v0.1 control stack should look like:

```text
                    target velocity
                          │
                          ▼
                   velocity PI
                          │
                          ▼
                    target pitch
                          │
                          ▼
 BNO085 ───────────► balance PD
                          │
                          ▼
                     base torque
                      /       \
                     /         \
             - steering       + steering
                 │                 │
                 ▼                 ▼
              motor L           motor R
                 │                 │
                 ▼                 ▼
             encoder L         encoder R
                  \               /
                   \             /
                    measured v
```

The balance loop is the critical inner loop.

Everything else commands it indirectly.

---

# 20. Software timing

Do not write the main balance controller as:

```text
loop()
{
    delay(100);
    ...
}
```

Use deterministic timing.

A reasonable target is:

```text
100–200+ Hz balance update
```

depending on the actual stable IMU data rate and interface.

If you use BNO085 UART-RVC mode, its reports arrive at 100 Hz.

If you use a BNO085 mode/interface capable of giving reliable higher-rate orientation and gyro data, the inner controller can operate correspondingly faster.

Run slow operations separately:

```text
Serial printing
Wi-Fi
telemetry
debug logging
future AI communications
```

Do not let a giant `Serial.println()` storm delay the balance loop.

Eventually the software architecture should behave like:

```text
high priority:
    IMU
    balance
    motor output

medium:
    encoder velocity
    motion controller

low:
    telemetry
    Wi-Fi
    logging
```

---

# 21. First free-standing test procedure

Do this in order.

### Test A — electronics only

Robot lifted from floor.

Confirm:

```text
IMU values correct
encoders correct
motors correct
E-stop works
fall detection works
```

### Test B — motor feedback sign

Robot held upright.

Lean forward slightly.

Verify wheels try to move forward.

Lean backward.

Verify wheels try to move backward.

### Test C — restricted support

Place Paulus between your hands or inside a loose support frame where he can tilt but cannot launch across the room.

Tune P and D.

### Test D — free balance

Place on clear floor.

Arm.

Release gently.

Be ready to hit E-stop.

### Test E — disturbance

Once stable, gently push the chassis.

He should move the wheels underneath himself and recover.

### Test F — stationary drift

Let him balance for one minute.

Observe whether he:

```text
creeps forward
creeps backward
oscillates
slowly wanders
```

Correct angle trim and later use encoder velocity feedback.

---

# 22. Failure diagnosis

## Robot instantly accelerates away

Most likely:

```text
control sign reversed
```

Fix this before touching gains.

## Robot barely reacts and falls

Likely:

```text
Kp too low
PWM limit too low
motor wiring problem
battery weak
```

## Robot violently oscillates

Likely:

```text
Kp too high
Kd too low
latency too high
sensor noise
```

## Robot jitters while almost stationary

Possible:

```text
Kd too high
noisy angle/rate signal
motor deadband
gearbox backlash
PWM too aggressive
```

## ESP32 resets when motors move

Suspect:

```text
power noise
ground bounce
regulator problem
bad wiring
motor EMI
```

Keep motor power and logic routing separate.

If necessary later, add local decoupling and approximately 100 nF ceramic suppression capacitors directly at the brushed motor terminals.

## BNO085 randomly disappears

Do not immediately blame your controller.

The BNO08x family has documented I²C interoperability problems with ESP32 hardware.

Try a different supported transport such as UART or SPI if your breakout exposes the required pins.

---

# 23. Mechanical refinement

Once Paulus balances, improve the chassis.

Check:

```text
motor mounts
wheel alignment
loose extrusion screws
battery movement
acrylic flex
wire strain
connector security
```

Mark screws that repeatedly loosen.

Consider medium-strength removable threadlocker where appropriate.

Move the battery vertically and retune.

You may discover a noticeably easier center-of-mass position.

Only then should you start optimizing aesthetics.

Version 0.1 is allowed to look terrible.

Its job is to stand.

---

# 24. Cat resistance

Even the prototype should follow several rules:

```text
no dangling wires
no accessible spinning shafts
no loose cable loops
no exposed high-current terminals
no components a paw can easily rip out
battery firmly strapped
robot cannot fall onto an animal during unattended testing
```

Do not leave Paulus balancing unattended around a cat.

A balancing controller cannot distinguish:

```text
external disturbance
```

from:

```text
cat has decided to conduct peer review
```

yet.

---

# 25. Acceptance criteria for Paulus v0.1

Do not declare v0.1 complete merely because he balances for three seconds once.

A proper milestone is:

```text
✓ Powers safely
✓ E-stop reliably removes motor power
✓ No exposed short-circuit hazards
✓ Both motors operate bidirectionally
✓ Both encoders report direction and speed
✓ IMU runs without communication failures
✓ Arms only when upright
✓ Automatically disarms after falling
✓ Balances autonomously for ≥ 60 seconds
✓ Recovers from gentle pushes
✓ Holds approximately zero velocity
✓ Drives forward under velocity command
✓ Drives backward under velocity command
✓ Turns left and right
✓ Stops and returns to stationary balance
```

At that point, Paulus is no longer an experiment in whether an inverted pendulum can be stabilized.

He is a **mobile robot platform**.

---

# 26. What comes after v0.1

Do not add these before the base is reliable.

## Paulus v0.2

Add:

```text
Wi-Fi command interface
telemetry
remote target velocity
remote steering
battery monitoring
```

Laptop becomes high-level controller:

```text
Laptop
   │
 Wi-Fi
   │
   ▼
 ESP32
   │
   ▼
balance robot
```

## Paulus v0.3

Add:

```text
Raspberry Pi 5
camera
ToF sensors
```

Architecture:

```text
camera
   │
   ▼
Raspberry Pi
person tracking
navigation
   │
target v / ω
   │
   ▼
ESP32
balance
   │
   ▼
motors
```

The Pi never balances the robot.

The ESP32 remains responsible for real-time physical control.

If Linux crashes:

```text
target velocity → 0
```

and the ESP32 should keep Paulus upright.

## Paulus v0.4+

Then add:

```text
microphones
speaker
speech recognition
face recognition
person following
memory
higher-level planning
agent architecture
```

Only at that point does:

> “Paulus, follow me.”

become a meaningful command.

---

# 27. The first boot sequence

When all the parts arrive, resist the urge to assemble everything immediately.

The correct sequence is:

```text
1. Set XL4016 to 5.0 V.
2. Build the hardware E-stop circuit.
3. Test the relay/E-stop.
4. Test motor L on the bench.
5. Test motor R on the bench.
6. Characterize encoder L.
7. Characterize encoder R.
8. Bring up the BNO085.
9. Verify pitch orientation and sign.
10. Assemble chassis.
11. Install battery.
12. Install electronics.
13. Route and secure wiring.
14. Verify power rails again.
15. Verify E-stop again.
16. Implement DISARMED/FALLEN states.
17. Verify motor feedback sign.
18. Implement balance PD.
19. Tune balance.
20. Achieve 60-second stationary balance.
21. Add encoder velocity loop.
22. Add forward/backward motion.
23. Add steering.
24. Only then begin Paulus v0.2.
```

The key engineering rule is:

> **Never debug mechanics, power electronics, sensors and control theory simultaneously.**

Make each layer boring before adding the next one.

The very first meaningful achievement for Paulus is therefore not speech, AI or even movement.

It is simply:

```text
pitch_target = 0

Paulus falls forward.
Paulus notices.
Paulus moves forward.
Paulus does not fall.

Paulus stands.
```

That is birth.
