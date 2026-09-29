# ESP32 tutorial

Start with **firmware infrastructure**, not motor code. For now, connect only the ESP32 by USB.

Since you're on Windows/WSL, I’d do the first hardware bring-up with **VS Code running on Windows**, not inside WSL. USB passthrough into WSL is extra complexity with zero benefit at this stage.

1. **Install the ESP32 development environment.** In VS Code, install the official **Espressif IDF** extension. Then open the Command Palette and run **`ESP-IDF: Open ESP-IDF Installation Manager`**. Install the current stable ESP-IDF plus its toolchain. Espressif's current recommended workflow is exactly this: Installation Manager + VS Code extension. :chatgpt-content-reference{index="0"}

   After installation, run:

   ```text
   Ctrl+Shift+P
   ESP-IDF: Doctor Command
   ```

   You want everything important to come back healthy.

2. **Connect the ESP32 over USB.** Use a known-good **data** cable, not a charge-only cable. Windows should expose something like:

   ```text
   COM3
   COM4
   COM5
   ```

   Look at the metal module on the ESP32 board. If it says something like:

   ```text
   ESP32-WROOM-32
   ESP32-WROOM-32D
   ESP32-WROOM-32E
   ```

   your ESP-IDF target is the classic:

   ```text
   esp32
   ```

   If it says S3/C3/S2/etc., use the corresponding target instead.

3. **Create Paulus's firmware repo.** I’d call it:

   ```text
   paulus-fw/
   ```

   In VS Code:

   ```text
   Ctrl+Shift+P
   ESP-IDF: New Project
   ```

   Start from the `hello_world` example if offered. Choose your chip target and COM port.

   Your top-level project should initially look roughly like:

   ```text
   paulus-fw/
   ├── CMakeLists.txt
   ├── sdkconfig
   └── main/
       ├── CMakeLists.txt
       └── main.cpp
   ```

   Build it, flash it, and open the serial monitor. From the command line, the equivalent workflow is essentially:

   ```powershell
   idf.py set-target esp32
   idf.py build
   idf.py -p COM5 flash monitor
   ```

   Substitute your actual COM port.

   **Do not continue until you can flash repeatedly and see boot logs.**

4. **Replace Hello World with Paulus's first actual firmware.** Use this as `main/main.cpp`:

   ```cpp
   #include <cstdint>

   #include "esp_log.h"
   #include "esp_timer.h"
   #include "freertos/FreeRTOS.h"
   #include "freertos/task.h"

   static const char* TAG = "paulus";

   static constexpr int64_t CONTROL_PERIOD_US = 5000; // 200 Hz

   enum class RobotState : uint8_t {
       BOOT,
       DISARMED,
       ARMED,
       BALANCING,
       FALLEN,
       FAULT,
   };

   struct Robot {
       RobotState state = RobotState::BOOT;

       float pitch_deg = 0.0f;
       float pitch_rate_dps = 0.0f;

       float left_command = 0.0f;
       float right_command = 0.0f;

       uint64_t control_cycles = 0;
       uint64_t missed_ticks = 0;
   };

   static Robot robot;

   static TaskHandle_t control_task_handle = nullptr;
   static esp_timer_handle_t control_timer = nullptr;

   static const char* state_name(RobotState state)
   {
       switch (state) {
           case RobotState::BOOT:      return "BOOT";
           case RobotState::DISARMED:  return "DISARMED";
           case RobotState::ARMED:     return "ARMED";
           case RobotState::BALANCING: return "BALANCING";
           case RobotState::FALLEN:    return "FALLEN";
           case RobotState::FAULT:     return "FAULT";
           default:                    return "UNKNOWN";
       }
   }

   static void control_timer_callback(void*)
   {
       if (control_task_handle != nullptr) {
           xTaskNotifyGive(control_task_handle);
       }
   }

   static void control_task(void*)
   {
       int64_t previous_time = esp_timer_get_time();

       while (true) {
           const uint32_t ticks =
               ulTaskNotifyTake(pdTRUE, portMAX_DELAY);

           if (ticks > 1) {
               robot.missed_ticks += ticks - 1;
           }

           const int64_t now = esp_timer_get_time();
           const float dt =
               static_cast<float>(now - previous_time) / 1'000'000.0f;

           previous_time = now;

           // -------------------------------------------------
           // Future control pipeline:
           //
           // read_imu();
           // read_encoders();
           // update_safety();
           // update_balance_controller();
           // write_motors();
           // -------------------------------------------------

           // Safety invariant:
           // DISARMED means motors produce ZERO command.
           if (robot.state == RobotState::DISARMED) {
               robot.left_command = 0.0f;
               robot.right_command = 0.0f;
           }

           robot.control_cycles++;

           // Report once per second.
           if ((robot.control_cycles % 200) == 0) {
               ESP_LOGI(
                   TAG,
                   "state=%s dt=%.4f ms cycles=%llu missed=%llu",
                   state_name(robot.state),
                   dt * 1000.0f,
                   robot.control_cycles,
                   robot.missed_ticks
               );
           }
       }
   }

   extern "C" void app_main(void)
   {
       ESP_LOGI(TAG, "Paulus v0.1 booting");

       // Critical invariant:
       // every boot begins DISARMED.
       robot.state = RobotState::DISARMED;

       BaseType_t task_result = xTaskCreate(
           control_task,
           "control",
           4096,
           nullptr,
           10,
           &control_task_handle
       );

       if (task_result != pdPASS) {
           ESP_LOGE(TAG, "Failed to create control task");
           return;
       }

       const esp_timer_create_args_t timer_args = {
           .callback = &control_timer_callback,
           .arg = nullptr,
           .dispatch_method = ESP_TIMER_TASK,
           .name = "control_tick",
           .skip_unhandled_events = true,
       };

       ESP_ERROR_CHECK(
           esp_timer_create(&timer_args, &control_timer)
       );

       ESP_ERROR_CHECK(
           esp_timer_start_periodic(
               control_timer,
               CONTROL_PERIOD_US
           )
       );

       ESP_LOGI(
           TAG,
           "Control loop started at 200 Hz; state=%s",
           state_name(robot.state)
       );
   }
   ```

   And make `main/CMakeLists.txt`:

   ```cmake
   idf_component_register(
       SRCS "main.cpp"
       INCLUDE_DIRS "."
       REQUIRES esp_timer
   )
   ```

   `esp_timer` is a good fit here because it provides microsecond-resolution periodic timers; Espressif explicitly recommends keeping its callbacks short, which is why our callback merely wakes a control task rather than doing the controller calculation itself. :chatgpt-content-reference{index="1"}

5. **Build, flash and observe Paulus's heartbeat.** You should see something roughly like:

   ```text
   I (...) paulus: Paulus v0.1 booting
   I (...) paulus: Control loop started at 200 Hz; state=DISARMED

   I (...) paulus: state=DISARMED dt=5.0010 ms cycles=200 missed=0
   I (...) paulus: state=DISARMED dt=4.9990 ms cycles=400 missed=0
   I (...) paulus: state=DISARMED dt=5.0030 ms cycles=600 missed=0
   ```

   The exact `dt` will jitter slightly.

   What matters is:

   ```text
   ~5 ms per iteration
   missed=0
   state=DISARMED
   ```

   Leave it running for a few minutes.

   Congratulations: this seemingly boring thing is already the real-time heartbeat that will eventually keep Paulus from smashing his face against the floor.

6. **Add the balance controller, but feed it imaginary physics.** Add:

   ```cpp
   struct BalanceController {
       float kp = 0.08f;
       float kd = 0.015f;
       float target_pitch_deg = 0.0f;

       float update(float pitch_deg, float pitch_rate_dps)
       {
           const float error =
               target_pitch_deg - pitch_deg;

           float output =
               kp * error -
               kd * pitch_rate_dps;

           if (output > 1.0f)
               output = 1.0f;

           if (output < -1.0f)
               output = -1.0f;

           return output;
       }
   };

   static BalanceController balance;
   ```

   These gains are **not real tuning values**. They're only for exercising the software.

   Temporarily put:

   ```cpp
   robot.pitch_deg = 5.0f;
   robot.pitch_rate_dps = 3.0f;
   ```

   into the controller path and calculate:

   ```cpp
   const float u = balance.update(
       robot.pitch_deg,
       robot.pitch_rate_dps
   );
   ```

   Then log `u`.

   Change the simulated input:

   ```text
   +5° forward lean
   0°
   -5° backward lean
   ```

   and verify that the output changes sign.

7. **Create the hardware abstraction now.** Don't let PID code eventually become full of GPIO calls.

   Move toward:

   ```cpp
   class Motor {
   public:
       virtual ~Motor() = default;

       virtual void set(float command) = 0;
       virtual void stop() = 0;
   };
   ```

   Then create:

   ```cpp
   class MockMotor : public Motor {
   public:
       void set(float command) override
       {
           last_command_ = command;
       }

       void stop() override
       {
           last_command_ = 0.0f;
       }

       float command() const
       {
           return last_command_;
       }

   private:
       float last_command_ = 0.0f;
   };
   ```

   Now the controller operates on:

   ```text
   BalanceController
           │
           ▼
        Motor
           │
      ┌────┴────┐
      ▼         ▼
   MockMotor  Bts7960Motor
     today       later
   ```

   Do the same eventually for:

   ```cpp
   Imu
   Encoder
   Motor
   ```

   This will make hardware bring-up dramatically less painful.

8. **Next, make Paulus's safety state machine real.** The important invariants should be software architecture, not scattered `if` statements:

   ```text
   BOOT
     ↓
   DISARMED
     ↓
   ARMED
     ↓
   BALANCING
   ```

   and:

   ```text
   BALANCING ── excessive tilt ──► FALLEN

   ANY STATE ── sensor failure ──► FAULT

   ANY STATE ── disarm ──────────► DISARMED
   ```

   In `DISARMED`, `FALLEN`, and `FAULT`:

   ```cpp
   left_motor.stop();
   right_motor.stop();
   ```

   Always.

   Later, the physical relay/E-stop adds an entirely independent safety layer beneath this.

9. **Once a BTS7960 arrives, replace `MockMotor` with real PWM.** ESP-IDF exposes the ESP32's LEDC hardware for PWM. The normal setup is timer configuration → channel configuration → duty updates. :chatgpt-content-reference{index="2"}

   We'll use four PWM signals eventually:

   ```text
   LEFT_RPWM
   LEFT_LPWM

   RIGHT_RPWM
   RIGHT_LPWM
   ```

   and a signed command:

   ```text
   +0.40
   ```

   becomes:

   ```text
   RPWM = 40%
   LPWM = 0%
   ```

   while:

   ```text
   -0.40
   ```

   becomes:

   ```text
   RPWM = 0%
   LPWM = 40%
   ```

   **Don't choose GPIO numbers yet** unless we know the exact ESP32 board/model you have. That avoids accidentally assigning a bootstrapping, flash, input-only, or otherwise inconvenient pin.

10. **Stop at this milestone before adding Wi-Fi.** Your first ESP32-only target should be:

   ```text
   ✓ reproducible build
   ✓ reliable flashing
   ✓ serial logging
   ✓ 200 Hz control heartbeat
   ✓ zero missed deadlines
   ✓ DISARMED boot invariant
   ✓ robot state machine
   ✓ mock IMU
   ✓ mock motors
   ✓ balance-controller implementation
   ✓ controller unit-testable independently of hardware
   ```

   Then, as actual packages arrive:

   ```text
   MockMotor   → BTS7960
   MockEncoder → JGB37 encoder
   MockIMU     → BNO085
   ```

   The control layer stays essentially unchanged.

The key idea is that **you don't need Paulus's body to begin building Paulus**. Right now you can make the ESP32 into a deterministic, safety-oriented real-time controller whose physical inputs and outputs are simply mocked.

Once you've successfully flashed the first 200 Hz version and it prints stable `~5 ms / missed=0`, the next concrete step should be **the `Motor` abstraction + BTS7960 driver**, or the **BNO085 driver** if the IMU happens to arrive first.
