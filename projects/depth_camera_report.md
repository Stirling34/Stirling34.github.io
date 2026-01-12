# Stereo Vision Failure on Reflective Surfaces

**EGH437 Research Project | 2024**

[Back to Portfolio](../)

---

## Overview

<img src="/assets/img/setup.png" alt="Experimental setup - controlled lighting environment" style="width:80%;">

*Experimental setup: Intel RealSense D455 camera, controlled lighting array, glossy test surface at varying angles*

How badly do stereo vision cameras fail when looking at glossy objects? And does the viewing angle matter?

These aren't academic questions—they're safety-critical for autonomous robots navigating real environments full of car windows, wet roads, polished floors, and glossy packaging. If your robot's depth camera can't see a reflective obstacle, it's going to drive straight into it.

This project experimentally characterized pixel dropout on the Intel RealSense D455 (a popular robotics camera) when imaging reflective surfaces at different angles. Found a non-linear relationship with an unexpected "sweet spot" around 65°, and proposed a practical solution using image segmentation to flag unreliable depth regions before they cause collisions.

---

## The Problem

Stereo vision cameras are everywhere in robotics—they're cheap, high-resolution, and give you both RGB and depth data. They work by matching features between two camera views, like your eyes do.

But there's a fundamental assumption: surfaces reflect light diffusely (Lambertian reflection), so they look the same from both cameras. Glossy surfaces break this assumption completely. Specular reflections create bright spots that appear in different places in each camera, or disappear entirely depending on the angle. The feature matching fails, leaving "holes" in the depth image.

**Real-world impact**: Autonomous delivery robots miss parked cars with glossy paint. Warehouse robots can't detect reflective packaging. Even NASA's Perseverance rover uses stereo vision—these failure modes matter.

**The research gap**: We know stereo vision struggles with reflections, but nobody had experimentally measured *how* pixel dropout changes with viewing angle on a real camera with all its imperfections and limitations.

---

## Experimental Results

<img src="/assets/img/pixel_graph.png" alt="Pixel dropout vs viewing angle graph" style="width:80%;">

*Pixel dropout percentage across viewing angles—note the non-linear relationship and optimal angle around 65°*

### The Non-Linear Relationship

- **85-75°** (nearly perpendicular): ~55-58% dropout—intense localized reflections at top of surface
- **65°** (optimal angle): 43% dropout—lowest point, reflection more dispersed
- **55-45°**: Gradual increase to 48%
- **35-15°** (oblique angles): Dramatic rise to 76% dropout—reflection covers entire surface

This wasn't what I expected initially. The conventional wisdom is "more oblique = worse performance," but there's clearly an optimal viewing angle where dropout minimizes before climbing again.

---

## Dropout Patterns

<img src="/assets/img/heatmap.png" alt="Pixel dropout heatmaps across angles" style="width:80%;">

*Heatmaps showing pixel dropout consistency—brighter = more consistent failure. Angles: 15° to 85° (left to right, top to bottom)*

The heatmaps tell the story:
- **High angles (85-75°)**: Intense bright spot at top of surface where specular reflection aligns with camera
- **Optimal angle (65°)**: Dropout more diffused, less intense overall
- **Low angles (45-15°)**: Dropout spreads across entire surface, increasing in intensity

As the board angle decreased, the location of intense dropout shifted downward (following the fixed lighting geometry), then became more uniform and severe.

---

## Why This Happens

The results align with prior theoretical work on specular reflection and stereo correspondence:

**Too close to perpendicular** (85-75°): The viewing angle aligns with the direction of specular reflection. The reflected light saturates the camera—intensity maxes out, features become indistinguishable, stereo matching fails.

**The sweet spot** (65°): Viewing angle is near but not exactly aligned with specular reflection. Enough reflected light reaches the camera for feature detection, but not so much that it saturates. The IR projection pattern is visible but not overwhelmed.

**Too oblique** (35-15°): The viewing angle differs significantly from specular reflection direction. The projected IR pattern reflects *away* from the camera. Even though there's no saturation, there's nothing for the stereo algorithm to match between the two camera views.

This matches the previous simulation-based predictions done by various academics regarding optimal viewing angles for stereo vision on reflective surfaces—except now we have experimental validation on a real camera with real imperfections.

---

## Practical Implications

### For Robot Designers

**Camera placement matters**: If your robot will encounter reflective surfaces (and it will), positioning cameras to view objects around 60-70° gives you the best chance of maintaining depth perception. Closer to perpendicular or more oblique both degrade performance.

**Multi-camera arrays**: A single stereo camera at one angle will have blind spots on glossy objects. Multiple cameras at different angles could cover each other's dropout regions.

**Sensor fusion**: Don't rely solely on stereo vision. LiDAR doesn't care about reflections (different failure modes). Combining sensors reduces the risk of missing reflective obstacles entirely.

### The Collision Avoidance Problem

If a robot can't see depth on 40-75% of a glossy surface, it might treat that surface as empty space. Driving into a car, a window, or even a person wearing reflective clothing becomes a real possibility.

**Current approach**: Most systems just hope they don't encounter too many reflective surfaces, or rely on redundant sensors.

**My proposed solution** ↓

---

## A Practical Solution

<img src="/assets/img/segmentation.png" alt="Image segmentation solution for dropout detection" style="width:80%;">

*Top left: RGB image | Top right: Segmented regions | Bottom left: Depth image (white = no data) | Bottom right: Flagged dropout region (red)*

Rather than trying to fix the depth camera (you can't), flag suspicious regions *before* the robot acts on bad data:

1. **Segment the RGB image** using mean-shift segmentation to identify distinct objects/regions
2. **Check depth data** for each segmented region
3. **Flag regions with high pixel dropout** (>30-40% pixels reading zero depth)
4. **Treat flagged regions as obstacles** until verified by other sensors or a human operator

This doesn't solve the stereo vision problem, but it prevents the robot from confidently driving into something it can't see. The red overlay gives operators immediate visual feedback about where the camera is blind.

**Implementation**: Computationally cheap enough to run in real-time. Could be combined with dynamic IR projection intensity adjustment (if the camera supports it) or multi-frame averaging to reduce dropout before segmentation.

---

## Methodology

**What I Did**: Took an Intel RealSense D455 (active stereo—projects an IR pattern to add texture), mounted it on a tripod in a controlled indoor environment, and pointed it at a glossy white board. Rotated the board from 85° (nearly perpendicular) down to 15° (nearly parallel) in 10° increments, flooding it with light to emphasize specular reflections.

For each angle:
- Captured 100 frames of depth data
- Repeated 5 times for statistical validity
- Calculated pixel dropout percentage (pixels reading zero depth more than 30% of frames)
- Generated heatmaps showing *where* dropout occurred on the surface

Used Python with Intel RealSense SDK 2.0 and OpenCV. No post-processing—wanted to see how the camera actually performs in realistic conditions.

**Key decision**: Left auto-exposure and auto-gain enabled. Yes, this adds variability, but robots operating in the real world don't get to lock these settings. The experiment needed to reflect actual deployment conditions.

---

## Limitations & Future Work

**Single surface type**: Only tested glossy white paint. Different materials (chrome, wet asphalt, glass) will have different reflectivity properties. Pattern might hold, magnitudes will vary.

**Fixed lighting**: Real robots operate in changing conditions—sunlight, shadows, indoor/outdoor transitions. The optimal angle might shift with ambient IR levels (sunlight is ~50% infrared).

**Coarse angle increments**: 10° steps showed the trend, but the exact optimal angle could be anywhere from 60-70°. Finer resolution would pinpoint it more precisely.

**Exposure settings**: Left at default to simulate realistic operation, but this added ~20% measurement variability (up to 40% at extreme angles). Future work could explore how adaptive exposure tuning affects dropout.

**Next steps**: Test on multiple surface types, vary ambient lighting conditions, integrate the segmentation solution into an actual robot navigation stack and measure collision avoidance improvement.

---

## Why This Matters

Stereo vision cameras are cheap and ubiquitous in robotics. They're not going away. But if we don't understand how and when they fail, we're building robots that can't safely navigate real-world environments full of reflective surfaces.

This project demonstrated:
- **Angle matters non-linearly**: There's an optimal viewing angle, not just "avoid oblique angles"
- **Failure modes are predictable**: The heatmaps show consistent patterns that can be anticipated
- **Practical solutions exist**: You don't need perfect sensors, just awareness of where they're blind

For Physical AI and autonomous systems, the lesson is clear: reliable perception requires understanding your sensors' failure modes, not just their nominal specifications. 40-76% pixel dropout isn't an edge case—it's what happens when your stereo camera looks at a parked car on a sunny day.

---

## Technical Stack

**Hardware**
- Intel RealSense D455 (active stereo with IR projection)
- 4× LED floodlights + 1× halogen floodlight
- Glossy white painted plywood test surface
- 3D-printed adjustable angle stand

**Software**
- Python (main analysis and control)
- Intel RealSense SDK 2.0 (camera interface)
- OpenCV (image processing, segmentation, heatmap generation)
- No post-processing or filtering

**Methodology**
- 100 frames per trial, 5 trials per angle, 8 angles (85° to 15°)
- Pixel defined as "dropout" if zero depth >30% of frames
- Controlled indoor environment (no windows)
- Camera distance fixed at 50cm
- ROI manually defined per angle using polygon mask

**Uncertainty**
- Overall ~15-25% variability across trials
- Spikes to ~40% at extreme dropout conditions (75° and 15°)
- Likely caused by auto-exposure/gain adjustments and frame-to-frame variation in specular reflection intensity

---

## Project Files

- **[Full Technical Report](./EGH437_n11587725_Stirling.pdf)** - Complete methodology, literature review, and analysis
- **Code** - Python scripts for data collection, analysis, and visualization
- **Data** - Raw depth frames, processed dropout percentages, heatmap generation
- **Segmentation Demo** - Implementation of proposed collision avoidance solution

---

[Back to Portfolio](../)
