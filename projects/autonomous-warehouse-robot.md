# Autonomous Warehouse Robot

**EGB320 Mechatronics Design | 2024**  
**Grade: 92/100**

[Back to Portfolio](../)

---
<div style="position:relative; padding-bottom:56.25%; height:0; overflow:hidden; margin-bottom:2rem;">
  <iframe 
    src="https://www.youtube.com/embed/3-7_UnPqpMY"
    style="position:absolute; top:0; left:0; width:100%; height:100%; border:0;"
    title="YouTube video player"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen>
  </iframe>
</div>

## Overview

<img src="/assets/img/robot_irl.png" alt="Robot in action" style="width:80%;">
*Final prototype - autonomous warehouse navigation robot*

A fully autonomous pick-and-place robot that navigates a 2m × 2m warehouse environment, retrieves requested items from shelves at three different heights, and delivers them to a packing bay—all while fitting within a 20cm × 20cm × 20cm box constraint.

**The Challenge**: Design and build a Technology Readiness Level 3 prototype that could autonomously complete warehouse tasks efficiently, without dropping objects or damaging the environment, within a $200 budget and a semester timeline.

**My Role**: Led complete hardware and electronics development for a 4-person team. Designed all mechanical systems in Fusion360, created custom PCB in Eagle CAD, selected and integrated motors, and coordinated system integration. When the team struggled with software integration days before the deadline, took over and redesigned the image segmentation algorithm 24 hours before final demonstration.

---

## System Requirements

<img src="/assets/img/functional_requirements.png" alt="Mobility subsystem requirements" style="width:80%;">
*Mobility subsystem functional requirements - constraints, inputs, and control logic*

The mobility subsystem had to satisfy multiple competing constraints:
- **Dimensional**: Fit within 20cm × 20cm × 20cm cube
- **Performance**: Navigate tight warehouse spaces, drive at variable speeds, climb inclines with payload
- **Safety**: Smooth turning and driving without damaging environment or objects
- **Integration**: Mechanical integration with item collection system, track system instead of wheels

---

## Demo Video

<video width="100%" controls>
  <source src="/assets/img/robot_video.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

*Watch the robot autonomously navigate, detect objects, and complete the warehouse task*

---

## Why This Project Matters

This wasn't a typical "follow the tutorial" robotics project. Success required understanding how mechanical constraints affect sensor performance, how power distribution impacts system stability, and how to make everything work together under competition pressure.

**Key achievement**: First student in course history to successfully implement 3D-printed TPU tank treads. This wasn't just innovation for its own sake—conventional wheels couldn't provide the traction and zero turning radius needed in the tight warehouse bays. The TPU treads solved both problems while staying under budget.

---

## Technical Deep Dive

### Mechanical Design

<img src="/assets/img/robot_orthographic.png" alt="Robot orthographic views" style="width:80%;">
*Orthographic projection showing track system dimensions and layout*

**Tank Track System**
- Custom 3D-printed TPU (flexible filament) tank treads—first successful implementation in the course
- Zero turning radius for navigating tight 2m × 2m warehouse space
- Precise wheel spacing (106mm track length, 26mm wheel diameter) to maintain proper track tension
- Added rubber grip tabs after discovering slippage issues in packing bay
- Low center of gravity (30mm track height) to prevent toppling when arm extended to full height

<img src="/assets/img/robot_render.png" alt="CAD render" style="width:80%;">
*Fusion360 render showing complete mechanical assembly*

**Chassis & Structure**
- Full mechanical design in Fusion360: chassis, motor mounts, four-bar linkage tower
- Chassis designed around item collection system—flat front for claw clearance, raised back tower for linkage mounting
- Multiple prototype iterations: started with 2WD wheels → tank tracks with 3rd tensioning wheel → final two-wheel system with reinforced mounts
- Upgraded wheel mounts to 4-bolt configuration to resist moment created by track tension

### Development Iterations

<img src="/assets/img/mobility_prototype_1.jpg" alt="Early mobility prototype" style="width:70%;">
*Prototype 1: Early tank tread design with 3rd tensioning wheel - discovered high internal friction*

<img src="/assets/img/mobility_prototype_2.jpg" alt="Final mobility system" style="width:70%;">
*Final mobility system: Two-wheel configuration with reinforced 4-bolt mounts and rubber grip tabs*

**Iteration Timeline**

**Prototype 1**: 2WD wheel configuration with basic chassis
- *Learning*: Turning radius too large for warehouse bays, insufficient traction

**Prototype 2**: First tank tread attempt with 3rd tensioning wheel
- *Learning*: High internal friction, track spacing calculation needed refinement

**Prototype 3**: Two-wheel tank system with calculated spacing
- *Learning*: Original mounts couldn't handle track tension, required structural upgrade

**Prototype 4 (Final)**: 4-bolt wheel mounts, rubber grip tabs, full system integration
- *Result*: 92/100, near-perfect demonstration, retrieved 3 objects from 3 heights successfully

### Electronics Design

<img src="/assets/img/electronics_dg.png" alt="Electronics system diagram" style="width:80%;">
*Electronics architecture - power distribution and control signal flow*

<img src="/assets/img/pcb_schematic.png" alt="Custom PCB schematic" style="width:80%;">
*Custom PCB schematic (Eagle CAD) - power distribution, voltage regulation, and LED control*

**Custom PCB** (Eagle CAD)
- Designed power distribution system splitting 7.4V battery to motor driver and voltage regulator
- Integrated variable voltage regulator for servo motors (prevents high servo current draw from crashing Raspberry Pi)
- LED control circuitry and testing headers
- Professional wiring harness with headers for clean cable management

**Motor Selection & Analysis**

<img src="/assets/img/motor_graph.png" alt="Motor speed-torque analysis" style="width:70%;">
*Speed-torque analysis for 100:1 HP motor - only option meeting both speed requirements at required torque*

- Calculated speed-torque requirements for warehouse navigation and incline climbing
- Initial 30:1 micro metal gearmotors failed—insufficient torque at required speeds
- Created comparative analysis: 150:1 HP vs 100:1 HP motors
- **Selected 100:1 HP micro metal gearmotors**: Only option meeting both speed constraints (green and pink dashed lines) while maintaining sufficient torque (blue dashed line) above friction torque (red dashed line)
- 150:1 motors couldn't reach high-speed requirement; lower gear ratios couldn't provide torque for inclines

**Power & Control Architecture**
- Raspberry Pi 3B+ running computer vision and control logic
- RF-Robot motor driver HAT with built-in 5V regulator for Pi
- PWM motor control (6V to motors via driver HAT)
- Direct PWM servo control from Pi GPIO
- 11kg metal gear servo (arm lift), 9g metal gear servo (claw grip)
- 7.4V battery pack with custom power distribution

### Computer Vision & Navigation

**The Crisis Fix**: 24 hours before final demonstration, the image segmentation algorithm couldn't distinguish between similar objects. Diagnosis: algorithm treated all detected objects identically. Solution: Redesigned to implement object-specific detection with switching mechanism—different detection parameters for each object type.

**System Integration**: Camera mounted above claws for line-following during object collection. Vision pipeline in Python/OpenCV integrated with motor control for autonomous navigation and object detection.

---

## Testing & Validation

<img src="/assets/img/robot_tests.png" alt="Robot test sequence" style="width:80%;">
*Successful demonstration: Robot retrieving objects from three different shelf heights and delivering to packing bay*

### Design Validation

- Fit within 20cm × 20cm × 20cm cube: ✓
- Navigate tight warehouse spaces with zero turning radius: ✓
- Operate at required slow and fast speeds: ✓
- Drive up inclines with payload: ✓
- Under $200 budget: ✓ ($150 actual cost)
- TRL 3 (proof of concept in simulated environment): ✓

**Final demonstration**: Successfully retrieved 3 objects from 3 different heights and placed them in the packing bay. After completing the task flawlessly multiple times, identified additional failure modes and made improvements (replaced servo motors, mobility motors, tracks, and grippers to increase long-term reliability).

---

## Technical Stack

**Hardware & Fabrication**
- Fusion360 (complete mechanical design)
- Eagle CAD (custom PCB schematic and layout)
- 3D printing: Bambu Lab P1S (TPU tank treads, chassis, mounts)
- Electronics assembly & professional wiring

**Electronics**
- Raspberry Pi 3B+ (main controller)
- RF-Robot motor driver HAT
- 100:1 HP micro metal gearmotors (2×)
- 11kg metal gear servo (arm), 9g metal gear servo (claw)
- 7.4V battery pack with custom power distribution
- Variable voltage regulator (custom PCB)

**Software**
- Python (main control logic)
- OpenCV (computer vision pipeline)
- PWM motor control
- Real-time sensor fusion

---

## Key Lessons

**TPU is finicky but worth it**: Successfully printing flexible TPU treads required dialing in printer settings (slower speeds, higher temps, proper retraction). Multiple iterations to find the right tread pattern for traction without binding. Worth the effort—saved ~$50 vs commercial tracks and performed better.

**Power distribution is critical**: Initial design crashed the Pi whenever servos drew high current. Solution: separate power rails—motor driver HAT powers Pi, custom PCB voltage regulator powers servos. Small detail, huge impact on reliability.

**Integration reveals problems theory doesn't**: Motors that worked perfectly in isolation struggled when mounted. Tracks that tensioned correctly on the bench slipped under load. The camera angle that seemed obvious in CAD created blind spots in practice. Real-world testing revealed issues no simulation would catch.

**Crisis management under pressure**: When the vision system failed 24 hours before demo, panic wasn't an option. Systematic debugging: understand the entire pipeline → identify failure mode → implement targeted fix. Object-specific detection solved it, but only because I took time to understand *why* it was failing, not just *that* it was failing.

**Four-bolt mounts matter**: Original two-bolt wheel mounts couldn't resist the moment from track tension—tracks kept jumping off wheels. Redesigned for four bolts, problem solved. Small mechanical details have cascading effects.

---

## Why This Matters for Physical AI / Robotics

This project demonstrates the full integration challenge that defines real robotics work:

- **Hardware-software co-design**: The mechanical design (low center of gravity, track placement) was driven by software requirements (camera field of view, sensor placement). Can't optimize one without the other.

- **Constraint-driven innovation**: The 20cm cube constraint and budget forced creative solutions (3D-printed TPU instead of commercial tracks, custom PCB instead of off-the-shelf power distribution).

- **Reliability under real conditions**: The robot didn't just work once in ideal conditions—it performed consistently under competition pressure after multiple test runs, motor replacements, and iteration cycles.

- **Problem-solving when things break**: Real systems fail. Motors burn out. Tracks slip. Vision algorithms produce garbage. Success means diagnosing and fixing problems quickly, not avoiding them entirely.

The 92/100 grade and successful three-object demonstration validated that every subsystem—mechanical, electrical, software—worked reliably when it mattered.

---

## Project Files

- **[Technical Report](./EGB320_Report.pdf)** - Full design documentation and analysis
- CAD Files - Fusion360 files for chassis, tracks, mounts, linkage system
- PCB Design - Eagle CAD schematics and board layouts
- Code - Python vision pipeline, motor control, navigation logic
- Motor Analysis - Speed-torque calculations and selection rationale

---

[Back to Portfolio](../)
