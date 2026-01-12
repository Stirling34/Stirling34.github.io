# Joshua Stirling
**Mechatronics Engineering @ QUT | I love learning and building useful things**

[LinkedIn](https://au.linkedin.com/in/joshua-stirling-705975298) | **Email:** joshua.stirling34@gmail.com

![Joshua Stirling Profile Picture](./assets/header-bg.png)
---

## Intro
I'm most interested in projects that make me learn something I didn't know before. That usually means working across disciplines—electronics and software, hardware and AI, theory and implementation. I like building systems efficiently, but also creatively—challenging assumptions about how things "should" be done to find solutions that are simpler, faster, or more elegant. Currently finishing fourth year with focus on autonomous systems, computer vision, and control theory.

---

## Featured Projects

### Autonomous Warehouse Robot
**92/100 | EGB320 Mechatronics Design (2024)**
Multi-level warehouse navigation system that had to find objects on shelves at different heights using computer vision, grab them, and avoid obstacles. Led hardware development—designed the PCB in Eagle, built the chassis in Fusion360, 3D-printed custom TPU tank treads (which was a first for the course). When the software hit a wall 24 hours before deadline, rewrote the image segmentation algorithm to get object-specific detection working.

* **Why it matters:** Full integration challenge—had to make electronics, mechanics, and vision work together under time pressure.
* **Tech:** Custom PCB (Eagle CAD), Fusion360, Python/OpenCV, Arduino, motor control
* [View detailed project →](./projects/warehouse-robot)

### Depth Camera Failure Analysis
**Research Project | Computer Vision**
Characterized how stereo vision cameras fail on reflective surfaces. Ran controlled experiments measuring pixel dropout on an Intel RealSense D455 across different viewing angles (15-85°). Found a non-linear relationship—dropout spikes at extreme angles and near-perpendicular views, with an optimal "sweet spot" around 65°.

* **Why it matters:** Autonomous robots using stereo vision can completely miss glossy objects (cars, windows, wet roads). Understanding when and why the sensor fails is critical for building reliable perception systems. Proposed using mean-shift segmentation to flag suspicious dropout regions before a robot drives into them.
* **Tech:** Python, OpenCV, Intel RealSense SDK, experimental design, data analysis
* [Read the full report →](./projects/depth-camera-analysis)

### Traffic Flow Control
**Current Thesis Work | Control Systems**
Using predictive control to reduce traffic shockwaves—those phantom jams that propagate backward through traffic. Working with Dr. Guilherme Froes Silva on state-space modeling and advanced control theory (LQR, Kalman filtering) applied to real-world traffic flow.

* **Why it matters:** Exploring how control systems can be used in already existing infrastructure to produce amazing benefits.
* **Tech:** MATLAB, SUMO, control theory, state-space modeling
* [View repository →](https://github.com/Stirling34)

---

## Other Work

### Systematic Trading Algorithm (2023-2024)
12 months building a Python-based trading system using technical indicators. Achieved 15% geometric return over backtesting period. Not robotics, but taught me a lot about systematic problem-solving, quantitative analysis, and iterative optimization based on performance data.

### Formation Education (Co-Founder, 2024-Present)
Built an education startup from scratch with my business partner—60+ students in first term. We teach systematic learning skills (how to study, manage time, learn independently) through 8-week group programs. Handled everything: website development, curriculum design, operations, infrastructure (Docker on a VPS). Currently scaling.  
[formationeducation.cloud](https://formationeducation.cloud)

---

## Technical Background

### Hardware & Fabrication
Fusion360 (7+ years), Eagle CAD, 3D printing (owned printers since 2020, currently Bambu Lab P1S), PCB design & manufacturing, electronics assembly, soldering

### Software & AI
Python, C, MATLAB | TensorFlow, PyTorch, Keras | OpenCV (extensive), ROS (basics) | Git, Docker

### Robotics & Control
Computer vision pipelines, motor control (PID, servo, stepper), state-space control (LQR), Kalman filtering, autonomous navigation, sensor integration

---

## Academic Highlights
**GPA: 6.75/7.00 (Dean's Scholar)**

| Course | Subject | Grade |
| :--- | :--- | :--- |
| **CAB320** | Artificial Intelligence: Neural networks, computer vision, NLP—trained image recognition models with TensorFlow/PyTorch | **7.0** |
| **EGB320** | Robotics: Warehouse robot project, autonomous systems, PCB design | **7.0** |
| **EGH413** | Advanced Dynamics: 3D rigid-body kinematics, Lagrange mechanics, complex spatial motion of multi-link systems | **7.0** |
| **EGH445** | Control Systems: State-space control, LQR, Kalman filtering—magnetic levitation controller project | **6.0** |
| **EGH437** | Robot Anatomy: ROS fundamentals, robot kinematics | **6.0** |

---

## About Me
I've been tinkering with Raspberry Pis, Arduinos, and 3D printers for 5+ years. What I love about Mechatronics is that it's real - through combining multiple subsystems and the latest technologies (such as AI) it's possible to create new and exciting solutions that weren't possible before. I find learning new things exciting, and so every skill or knowledge base I pick up from a uni course or project feels like another tool on my belt.

Outside of robotics: I run a startup, I've built trading algorithms, I coach high school students in CAD and programming. In short - I like learning new things and figuring out how to build them.
