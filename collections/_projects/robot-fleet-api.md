---
layout: project-right
title: "Robot Fleet API"
description: "Developing a Robot Fleet API using FastAPI, msgspec, and rosbridge_server"
date: 2026-10-21
weight: 1
thumbnail: "/assets/images/projects/robot-fleet-api/robot_transport_thumbnail.webp"
categories: ["Development"]
client: "Health Care"
role: "Robot Software Engineer"
gallery:
  - image: "/assets/images//projects/robot-fleet-api/robot_transport.webp"
    caption: "<strong>Above:</strong> Robot transporting items to the ward"
  - image: "/assets/images/projects/robot-fleet-api/robot_colab.webp"
    caption: "<strong>Above:</strong> Mobile manipulator and AGV collaborating on a task"
  - image: "/assets/images/projects/robot-fleet-api/robot_pick.webp"
    caption: "<strong>Above:</strong> Mobile manipulator unloading mobile platform"
gallery_limit: 3
---


Picture a robot with a hospital job. It shuttles lab samples down to the lab, hauls dirty linen to the laundry, and comes back with fresh sheets for the ward storage room. Every trip it makes is one less trek down a long corridor for a nurse, and more time for the patients who actually need them. 

There’s just one catch: a robot is only worth having if the people around it know what it’s up to and can tell it what to do. And nurses, doctors and ward staff don’t speak ROS2. And even if they did, nobody on a busy shift wants to SSH into a container and type commands into a blinking terminal just to find out where the clean towels went. They need apps that make the robot as easy to follow and direct as a colleague: tap a button, see where it is, send it on its way.

But who builds those apps? Not the robotics engineers. They’re great at making robots move, but app development and UX design aren’t their specialty. That’s a job for web developers and UX designers, and they don’t speak ROS2 either. They need an API they can ask simple questions, like “What’s this robot’s battery level?”, and get a plain answer back.

That’s why my team built an API that web apps can plug into, whether it’s an app on a nurse’s phone or a tablet mounted on the robot itself.

Read more about this in my recent blog post [here](https://sjoerddevelops.github.io/blog/2026-10-21-two-stage-decoding-of-rosbridge-messages/)
