I’d generate the assembly sequence in about **14 visual steps**. Use the same master style prompt every time, then append the step-specific prompt.

### Master style prompt

Use this at the start of every image prompt:

> Create a clean black-and-white isometric technical assembly illustration for a DIY two-wheel self-balancing robot named “Paulus v0.1”. Style: professional mechanical assembly manual, like LEGO Technic / IKEA / industrial service documentation, white background, crisp black vector-like line art, very light gray shading only where necessary, no color, no photorealism, no decorative styling. Use a consistent 30-degree isometric perspective. Show exploded parts with dashed alignment guides and solid arrows indicating assembly direction. Use numbered circular callouts only; avoid long text inside the drawing. Components must look mechanically plausible and proportionally correct. Show screws, brackets, spacers, nuts, cable paths and connector directions clearly. Do not hide parts behind opaque bodywork. Do not invent components not specified. The robot uses 2020 aluminum extrusion, two JGB37-520 12 V encoder gearmotors with wheels, two BTS7960 motor-driver boards, ESP32 development board, BNO085 IMU breakout, XL4016 buck converter, 3S battery pack, automotive relay, 15 A inline fuse, XT60 connector, red mushroom emergency-stop switch, acrylic mounting plate, 16 AWG power wire and small signal wiring. No caster wheel; only the two drive wheels touch the ground.

Then append one of these.

---

## Step 1 — Lay out the aluminum chassis

> Show three 500 mm 2020 aluminum extrusion profiles before assembly. Show two vertical rails approximately 320–350 mm long and one horizontal lower crossmember approximately 200–250 mm wide, with cut marks indicated if the profiles need trimming. Present the final intended rectangular inverted-U chassis shape beside the exploded profiles. Use arrows to show where the profiles will meet. Call out the two vertical rails as 1 and 2, and the lower crossmember as 3. Include a small dimension-reference inset, but no dense text.

**Instruction:** establish the basic frame geometry first. Keep the two vertical members parallel and symmetric.

---

## Step 2 — Install the 2020 corner brackets

> Show the two vertical 2020 rails being joined to the lower horizontal crossmember using 90-degree aluminum corner brackets, M5 screws, and T-slot nuts. Use an exploded isometric view showing the T-nuts sliding into the extrusion channels, brackets aligning against the profiles, and screws entering through the bracket holes. Show four brackets total, two on each lower corner if space permits. Include tightening arrows but indicate final tightening should happen only after squaring the frame.

**Instruction:** loosely assemble first, square the frame, then tighten.

---

## Step 3 — Mount the motor brackets

> Show one JGB37 motor mounting bracket being installed at each end of the lower 2020 crossmember. The brackets must be symmetric and aligned so both motor shafts are horizontal, coaxial, and at the same height. Show the mounting screws and T-nuts in exploded position. Include a dashed centerline through both motor shaft positions to emphasize alignment.

**Instruction:** motor-shaft alignment matters. Both wheel axles should lie on the same horizontal line.

---

## Step 4 — Install the JGB37 motors

> Show two cylindrical JGB37-520 encoder gearmotors sliding into their mounting brackets from opposite sides of the frame. Show the gearbox bodies, rear encoder sections, 6 mm D-shafts facing outward, encoder cables facing inward or upward away from the wheels, and motor mounting screws. Use arrows to show insertion and tightening direction. Do not install wheels yet.

**Instruction:** orient the encoder wiring where it cannot touch the wheels.

---

## Step 5 — Install the wheels

> Show one wheel being fitted onto each 6 mm D-shaft. Show the flat side of the D-shaft aligned with the wheel hub or coupling, then show the retaining screw or hub fastener tightening against the shaft. Show both wheels installed symmetrically, with a dashed floor plane indicating that the two wheels are the robot’s only ground contact points.

**Instruction:** verify that both wheels spin freely without contacting the frame.

---

## Step 6 — Mount the battery

> Show a 3S2P 11.1 V battery pack positioned vertically between the two aluminum rails, approximately 100–150 mm above the wheel axle. Show two reusable Velcro battery straps passing around the battery and the chassis or battery support plate. Show vertical adjustment arrows indicating that the battery position can later be moved upward or downward for center-of-mass tuning. Keep the battery centered laterally.

**Instruction:** do not permanently fix the battery position yet.

---

## Step 7 — Install the acrylic electronics deck

> Show a rectangular section cut from a 3 mm transparent acrylic sheet being mounted horizontally or vertically between the 2020 rails using spacers, screws and T-slot nuts. Show four mounting points with washers. Include a drilling inset showing the acrylic supported underneath and a washer placed under the screw head. Leave sufficient clearance for wiring and access to screw terminals.

**Instruction:** acrylic supports electronics; the aluminum frame carries structural loads.

---

## Step 8 — Mount the motor drivers

> Show two BTS7960 / IBT-2 motor-driver boards mounted side by side on the acrylic plate using M3 screws and nylon or metal standoffs. Orient the high-current screw terminals toward the lower part of the robot near the motors. Orient the logic headers upward toward the ESP32 position. Show air space around both heatsinks. Label the numbered callouts only as left driver and right driver through an external legend, not text printed on the boards.

**Instruction:** one BTS7960 controls the left motor; the other controls the right motor.

---

## Step 9 — Mount the regulator, ESP32 and BNO085

> Show three electronic modules being installed on the upper half of the acrylic deck: an XL4016 buck converter, an ESP32 development board and a small BNO085 IMU breakout. Mount each on standoffs. Place the BNO085 near the rigid central structure of the chassis, away from the motors, relay and high-current wiring. Show the IMU orientation axes with small X, Y and Z arrows. Place the ESP32 where its USB connector remains accessible.

**Instruction:** the BNO085 must be rigidly attached; flex or vibration at its mounting point directly hurts balancing.

---

## Step 10 — Install the emergency stop

> Show a 22 mm mushroom emergency-stop switch being mounted through a drilled hole in either a small upper acrylic panel or dedicated bracket near the top side of the robot. Show the button body passing through the hole, retaining nut tightening from behind, and the normally-closed contact block behind the panel. Make the button physically easy to reach from either side of the robot.

**Instruction:** the E-stop should be reachable instantly during tuning.

---

## Step 11 — Install fuse, relay and XT60

> Show the power-safety components mounted near the battery: XT60 battery connector, inline 15 A blade fuse holder, 12 V automotive relay and Faston terminals. Use a simplified exploded arrangement showing the physical order: battery connector, fuse, branch to logic regulator, relay contact, motor-driver power bus. Show the 1N4007 diode connected across the relay coil with the striped cathode toward positive. Use numbered callouts and cable-routing arrows rather than dense wiring text.

**Instruction:** put the fuse physically close to battery positive.

---

## Step 12 — Wire the high-current power path

> Show only the high-current wiring in an isometric cutaway-style diagram. Use thick black lines for 16 AWG wiring. Route battery positive through XT60 and the 15 A fuse. From there show one branch to the XL4016 input and one branch through the automotive relay contacts to the power inputs of both BTS7960 boards. Route battery negative directly to a common negative distribution point and from there to both BTS7960 boards and XL4016. Keep the drawing mechanically realistic and show insulated Faston terminals on the relay.

**Instruction:** keep the high-current paths short and mechanically secured.

---

## Step 13 — Wire the low-voltage control system

> Show only the signal and logic wiring. Use thin black lines. Show the XL4016 5 V output feeding the ESP32 development board and appropriate 5 V logic inputs of the BTS7960 modules. Show common ground connections. Show the BNO085 connected to the ESP32 through the selected communication interface, leaving the exact pins as numbered callouts rather than inventing pin numbers. Show each motor encoder’s A and B outputs routed separately to ESP32 GPIO inputs. Show RPWM, LPWM, R_EN and L_EN from the ESP32 to each BTS7960. Keep encoder and IMU wiring physically separated from motor-power cables.

**Instruction:** verify encoder voltage compatibility before connecting encoder outputs directly to ESP32 GPIO.

---

## Step 14 — Final Paulus v0.1 assembly

> Show the completed Paulus v0.1 self-balancing robot in a clean black-and-white isometric technical view. Two rubber wheels at the bottom, JGB37 motors on either side of a rigid 2020 aluminum chassis, battery vertically strapped near the lower-middle area, acrylic electronics plate with two BTS7960 drivers, ESP32, BNO085 and XL4016, inline fuse and automotive relay neatly mounted, prominent emergency-stop mushroom button near the upper side, tidy secured wiring, no loose cable loops, no caster wheel and no cosmetic shell. Include subtle numbered callouts corresponding to every major subsystem. Make it look like a functional engineering prototype rather than a consumer robot.

**Instruction:** this is the physical v0.1 configuration before control-loop tuning.

---

For the **control/tuning section**, I’d make another smaller sequence of schematic images rather than mechanical assembly drawings: **bench motor test → encoder-direction test → IMU pitch test → held-upright feedback-sign test → restricted balance test → free balancing → velocity control → differential steering**.

For example, the critical first-balancing image prompt would be:

> Create a black-and-white isometric technical instruction drawing of the completed Paulus v0.1 robot being lightly supported upright by two human hands while its wheels remain on the floor. Show the robot leaning approximately 5 degrees forward, with a curved pitch arrow indicating forward lean and large arrows on both wheels showing that the controller must rotate the wheels forward underneath the center of mass. Beside it show a second miniature diagram with the robot leaning backward and both wheels rotating backward. Add a large warning symbol beside an incorrect case where forward lean causes backward wheel motion. Minimal labels only: “CORRECT” and “WRONG”. Professional robotics-control manual style.

That particular illustration is probably more valuable than almost any other because it visually captures **the feedback sign that must be correct before you ever let Paulus go**.
