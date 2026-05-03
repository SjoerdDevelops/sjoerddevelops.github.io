---
layout: project-top-custom
title: "Orbital Head-Mounted Display"
description: "An interface for viewpoint control during robot teleoperation"
introduction: "Robots are incredibly capable, but lack the ability to adapt to unexpected scenarios. This is where robotic teleoperation comes in: a powerful combination of the physical qualities of robots and the creativity and adaptability of humans. While robots handle the physical tasks, it’s the human operator who makes nuanced decisions that machines simply can’t replicate.<br><br>

But there’s a catch. For teleoperation to truly work, operators need to feel connected to the remote environment without becoming overwhelmed. Too much information can be just as problematic as too little, and an overload of input options can quickly make simple tasks cumbersome. Designing interfaces that enhance situational awareness while keeping cognitive load manageable is one of the biggest challenges in this field.<br><br>

That’s exactly the problem this research set out to tackle."
date: "2022-03-29"
weight: 1
thumbnail: "/assets/images/gen/projects/project-ohmd-1-thumbnail.webp"
client: "TU Delft"
categories: ["Research"]
role: "Researcher"
technologies: ["Unity", "VR"]
gallery:

- image: "/assets/images/gen/projects/project-ohmd-1.webp"
- image: "/assets/images/gen/projects/project-4-4.webp"
- image: "/assets/images/gen/projects/project-4-7.webp"
gallery_limit: 2

---
## The Teleoperation Visual Feedback Challenge

Traditional teleoperation setups often rely on arrays of fixed camera views shown on monitors. While this approach works, it comes with notable limitations. The available viewpoints are limited and often suboptimal due to camera placement constraints. Furthermore, operators must mentally piece together a 3D understanding of the scene from separate 2D images.

Some of these drawbacks can be avoided by introducing a dynamic viewpoint. In two extreme cases, this viewpoint is either autonomously or manually controlled. However, both approaches introduce new challenges. Autonomous camera control may present views that don't align with the operator's preferences, while manual control requires the operator to manage an additional six degrees of freedom.

The introduction of Stereoscopic head-mounted displays (i.e. Virtual Reality headsets) offers another improvement by providing depth information and more intuitive controls. Even so, the operator's viewpoint remains limited by their physical movement range.

## A New Way to See and Control

Orbital head-mounted display (OHMD) combines a third-person camera model - often used in video games - with a VR headset. It uses the motion sensors of the head-mounted display to pan and tilt the camera around a fixed point at the robot's end-effector whilst maintaining a constant view distance. This distance can be adjusted via a handheld joystick such that operators can change their field of view. This creates an intuitive camera control interface that is not constrained by the operator's physical workspace.

## Putting It to the Test

To assess how well OHMD performs, a human factors study was performed comparing three different interfaces:

- A traditional setup with multiple fixed camera views (Array)
- A standard head-mounted display (HMD)
- Orbital head-mounted display (OHMD)

Participants were asked to complete robotic welding tasks in a simulator. They task performance, perceived workload, and the overall user acceptance of the interfaces were recorded.

## What the Results Showed

The results were clear: both VR-based systems — OHMD and the standard HMD — outperformed the traditional multi-camera setup. Operators using these interfaces completed tasks more effectively, felt less mentally strained, and reported a more positive experience overall.

<strong>But OHMD brought something extra to the table.</strong>

Participants consistently favored OHMD for its flexibility and support. The ability to freely adjust viewpoints using natural head movements made it easier to understand the environment and maintain awareness. Many users felt it gave them greater control and confidence during operation.

Interestingly, though, the standard HMD still had an edge when it came to performing simple robotic tasks. Operators using the traditional HMD completed them noticeably faster than those using an OHMD. In this scenario, the stereoscopic view already provides sufficient contextual and depth information, making it unnecessary to change viewpoint. This, int turn, limits the usability benefits offered by the OHMD interface.

## Orbital Head-Mounted Display in 5 Minutes

<div class="container-832">{% include framework/shortcodes/youtube.html id="i6nopCUrqE0" title="IROS 2023 - Orbital Head-Mounted Display (OHMD)" %} </div>

<!-- ## Striking the Right Balance -->
## Links

- [Orbital Head-Mounted Display: A Novel Interface for Viewpoint Control during Robot Teleoperation in Cluttered Environments](https://ieeexplore.ieee.org/document/10341733)
  - Published in: [2023 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)](https://ieeexplore.ieee.org/xpl/conhome/10341341/proceeding)
  - DOI: [10.1109/IROS55552.2023.10341733](https://doi.org/10.1109/IROS55552.2023.10341733)
- [Youtube: IROS 2023 - Orbital Head-Mounted Display (OHMD)](https://www.youtube.com/watch?v=i6nopCUrqE0)
