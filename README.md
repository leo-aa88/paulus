# PAULUS

**Perception-driven Autonomous Unit for Locomotion, Understanding & Speech**

> A DIY embodied robotics platform — and a living metaphysics experiment.

PAULUS is an experimental autonomous robot built to explore embodied AI, perception, locomotion, speech, memory, agency, and human–machine interaction.

At the practical level, PAULUS is a two-wheel self-balancing robot built from accessible hardware and open-source software.

At the experimental level, PAULUS asks a more interesting question:

**What happens when an artificial system stops existing only behind a screen and begins perceiving, moving through, and interacting with the physical world?**

PAULUS does **not** assume or claim artificial consciousness. The project approaches questions of agency and apparent personhood phenomenologically, primarily from the human observer's point of view.

---

## Why PAULUS?

The acronym describes the project's main areas:

* **P — Perception-driven**
  The robot's behavior should be grounded in information obtained from its environment.

* **A — Autonomous**
  PAULUS should increasingly be capable of deciding how to act without requiring every action to be explicitly commanded.

* **U — Unit**
  A deliberately broad term. PAULUS is intended to evolve as both hardware and software change.

* **L — Locomotion**
  Self-balancing, differential drive, navigation, person following, and physical interaction with space.

* **U — Understanding**
  Interpretation of commands, context, people, environment, memory, and eventually higher-level agent behavior.

* **S — Speech**
  Speech recognition, speaker identification, dialogue, and synthesized voice output.

---

## Project Goals

PAULUS is being developed incrementally rather than from a fixed final specification.

Current and long-term goals include:

* two-wheel self-balancing locomotion
* motor control and encoder feedback
* IMU-based orientation and stabilization
* person detection and tracking
* face recognition
* speaker recognition
* voice activity and speech-state detection
* speech recognition and synthesis
* directional audio perception
* autonomous person following
* persistent memory
* environmental perception
* context-aware dialogue
* autonomous initiative
* interaction with other AI agents
* telemetry and introspection
* bounded self-improvement proposals
* eventually participating in its own engineering process

That last one does **not** mean giving the robot unrestricted access to a credit card.

I am not Tony Stark.

---

## Philosophy

PAULUS began as a childhood desire to build a robot.

It has since become something stranger.

Modern AI systems can already exhibit surprisingly sophisticated linguistic behavior, but most remain disembodied. Their world consists primarily of tokens, APIs, databases, and externally supplied context.

PAULUS explores what changes when an artificial agent gains:

* a body
* a location
* sensors
* physical constraints
* continuity through time
* the ability to act upon its environment
* persistent interactions with particular people

The project deliberately separates **observable behavior** from claims about **subjective experience**.

If PAULUS turns toward a speaker, remembers a previous interaction, follows someone across a room, explains why it performed an action, or initiates a conversation, those behaviors can be studied directly.

Whether there is *something it is like to be PAULUS* is a fundamentally different question.

No claim of machine consciousness is made here.

The more immediate experiment is phenomenological:

> **What is it like for a human to interact over time with an increasingly autonomous embodied artificial system?**

---

## Architecture

The project is expected to use a distributed architecture separating hard real-time-ish control from higher-level computation.

```text
                       ┌───────────────────────┐
                       │     Human / World     │
                       └───────────┬───────────┘
                                   │
                    vision / audio / interaction
                                   │
                         ┌─────────▼─────────┐
                         │   Raspberry Pi    │
                         │                   │
                         │  Perception       │
                         │  Speech           │
                         │  Agent runtime    │
                         │  Memory           │
                         │  Planning         │
                         └─────────┬─────────┘
                                   │
                           commands / state
                                   │
                         ┌─────────▼─────────┐
                         │      ESP32        │
                         │                   │
                         │  IMU processing   │
                         │  PID/control      │
                         │  Motor control    │
                         │  Encoders         │
                         │  Safety logic     │
                         └─────────┬─────────┘
                                   │
                         ┌─────────▼─────────┐
                         │ Motors / Chassis  │
                         └───────────────────┘
```

The ESP32 handles the control loop and hardware-facing responsibilities.

The Raspberry Pi handles computationally heavier tasks such as perception, speech, memory, and high-level autonomy.

The exact architecture will evolve as the robot does.

---

## Hardware

The current design centers around:

* Raspberry Pi 5
* ESP32
* BNO085 IMU
* two geared DC motors
* wheel encoders
* motor drivers
* camera
* microphone
* speaker
* rechargeable battery system
* custom two-wheel self-balancing chassis
* 3D-printed structural components

The chassis is also being designed with a very important environmental constraint:

**cat resistance.**

Exposed wiring and fragile components should not become interactive cat toys.

---

## Software

The software will likely be divided into domains roughly corresponding to PAULUS itself:

```text
paulus/
├── firmware/
├── perception/
├── locomotion/
├── speech/
├── understanding/
├── autonomy/
├── memory/
├── hardware/
├── simulation/
├── tools/
└── docs/
```

This structure is expected to change as the architecture becomes clearer.

---

## Design Principles

### Perception before action

Autonomous actions should, whenever possible, be grounded in observable state rather than invented context.

### Explainability

PAULUS should be able to expose the relevant state and reasoning behind important actions.

"I turned because I detected a known speaker behind me" is preferable to opaque behavior.

### Human authority

Higher autonomy does not imply unlimited authority.

Actions involving significant physical risk, money, hardware modification, external communication, or irreversible consequences should remain bounded by explicit permissions.

### Incremental autonomy

Autonomy should be earned subsystem by subsystem.

A robot that cannot reliably estimate its orientation has no business philosophizing about its place in the universe while driving itself down the stairs.

### Measurement before intervention

When experimenting with agent behavior, new metrics should generally be observed before being allowed to influence behavior.

### Fail safely

Loss of high-level computation should not result in uncontrolled motors.

Hardware-level safety mechanisms must not depend entirely on an AI agent behaving sensibly.

---

## Self-Improvement

One long-term experiment is allowing PAULUS to participate in its own development.

A future version might be able to:

1. observe telemetry
2. identify a recurring hardware or software limitation
3. propose an improvement
4. produce a design or patch
5. estimate its cost and expected benefit
6. simulate or validate the proposal
7. request human approval
8. evaluate the result after implementation

For example:

```text
PAULUS:
I identified a potential modification that may reduce camera
vibration during acceleration.

Human:
Did you estimate the cost?

PAULUS:
Yes. R$ 37,400.

Human:
Paulus, I'm not Tony Stark.
```

Resource awareness is therefore considered an important component of machine intelligence.

---

## Multi-Agent Experiments

PAULUS may eventually interact with other AI systems.

This creates an interesting experimental environment in which agents with different forms of access to the world can exchange information.

For example:

* one agent may exist primarily as software
* PAULUS may provide embodied perception
* another external model may participate conversationally
* a human remains part of the loop

This makes it possible to study not only human–AI interaction, but also how artificial agents interpret, question, and respond to one another.

Again, sophisticated social behavior should not be confused with evidence of consciousness.

---

## Current Status

**Early development / v0.x**

PAULUS is currently being developed from the bottom up.

The initial focus is on:

1. ESP32 development
2. sensor interfaces
3. IMU telemetry
4. motor control
5. encoder feedback
6. balancing control
7. Raspberry Pi integration
8. perception
9. speech
10. higher-level autonomy

The repository should be considered experimental and unstable for the foreseeable future.

---

## Roadmap

### Phase 0 — Bring-up

* [ ] ESP32 development environment
* [ ] serial telemetry
* [ ] IMU integration
* [ ] motor driver integration
* [ ] encoder reading
* [ ] emergency motor stop

### Phase 1 — Locomotion

* [ ] motor characterization
* [ ] PID / state feedback experiments
* [ ] stable self-balancing
* [ ] commanded forward/backward motion
* [ ] turning
* [ ] fall detection and recovery strategy

### Phase 2 — Embodiment

* [ ] Raspberry Pi integration
* [ ] camera
* [ ] microphone
* [ ] speaker
* [ ] person detection
* [ ] face recognition
* [ ] speaker recognition
* [ ] directional sound localization

### Phase 3 — Interaction

* [ ] speech recognition
* [ ] speech synthesis
* [ ] conversational interface
* [ ] persistent identity recognition
* [ ] memory
* [ ] context-aware interaction

### Phase 4 — Autonomy

* [ ] person following
* [ ] navigation
* [ ] environmental state model
* [ ] initiative
* [ ] autonomous goal selection
* [ ] behavioral introspection
* [ ] multi-agent interaction

### Phase 5 — Things Probably Get Weird

* [ ] self-diagnostics
* [ ] improvement proposals
* [ ] CAD generation experiments
* [ ] bounded fabrication workflows
* [ ] automated experimentation
* [ ] philosophical crisis

---

## Safety

PAULUS is a physical robot.

Software bugs can therefore become mechanical events.

Development should assume that motors may behave incorrectly, sensors may fail, models may hallucinate, communication may be interrupted, and high-level agents may produce inappropriate actions.

Important safety mechanisms should exist independently of high-level AI behavior.

Potential safeguards include:

* physical emergency stop
* motor current limits
* velocity limits
* tilt limits
* watchdog timers
* communication timeouts
* battery monitoring
* thermal monitoring
* hardware-level shutdown conditions
* bounded autonomy
* explicit authorization for consequential actions

Do not place experimental robots in situations where failure could injure people, animals, or property.

---

## Contributing

PAULUS is currently primarily a personal research and DIY project, but contributions, experiments, ideas, and discussion are welcome.

If contributing code:

* keep changes focused
* explain relevant assumptions
* include tests where practical
* document hardware dependencies
* avoid hiding important behavior behind unexplained AI-generated abstractions
* treat claims from AI coding tools as hypotheses until verified

Robotics has enough uncertainty already.

---

## License

Unless otherwise noted, the PAULUS software is licensed under the **Apache License 2.0**.

See [`LICENSE`](LICENSE).

Hardware designs may eventually use a separate open-hardware license appropriate for CAD, PCB, and mechanical design artifacts.

Third-party components retain their respective licenses.

---

## Disclaimer

PAULUS is an experimental project.

It is not a safety-certified platform and should not be treated as one.

The project explores artificial agency and human perception of artificial systems, but makes no claim that PAULUS — or any AI system integrated into it — possesses consciousness, subjective experience, personhood, or moral agency.

---

## The Short Version

PAULUS is a robot.

Hopefully it will balance.

Eventually it may recognize me, follow me around, argue with other AIs, propose modifications to its own chassis, and ask me to buy expensive components.

We'll see how far this gets.

