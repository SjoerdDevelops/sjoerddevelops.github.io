---
layout: project-right
title: "Robot Fleet API"
description: "Developing a Robot Fleet API using FastAPI, msgspec, and rosbridge_server"
date: "2026-10-21"
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

I was therefore tasked to develop a Robot Fleet API using [FastAPI](https://fastapi.tiangolo.com/) that talks to robots through `rosbridge_server` that web apps can hook into, whether it’s an app on a nurse’s phone or a tablet mounted on the robot itself.

## The Challenge

I wanted the whole chain to be strongly typed, from the raw websocket bytes all the way to the domain entities and API endpoints. But typing everything has a price. Validating a payload costs CPU on every message, which adds up for large and frequent streams such as camera images and point clouds. Every payload type also needs a model, which adds development cost.

As a team we pilot new robot concepts in hospital environments and in this context the robots' interfaces change regularly. Most of our messages are small, so decoding them costs almost nothing in performance. The real cost is development time: every interface change means updating a model, and that time is better spent on the robot concepts themselves.

The challenge was therefore to build a scalable API that supports rapid development cycles, while still allowing thorough validation of data traffic where it's needed.

## The Architecture at a Glance

The codebase is split into three modules, each with its own responsibility:

- **`rosbridge`** (transport layer) sends and receives raw JSON packets.
- **`core`** (application layer) routes requests and responses to their handlers based on envelope type.
- **`api`** (interface layer) exposes the robot fleet over HTTP.

Validation is split across the bottom two layers and handled by the [msgspec](https://jcristharif.com/msgspec/) library. The transport layer encodes and decodes packets into rosbridge envelopes, and the application layer optionally decodes the payloads those packets carry. For more on why I chose msgspec over Pydantic and how the two-stage decoding works, see [my blog post](https://sjoerddevelops.github.io/blog/2026-10-21-two-stage-decoding-of-rosbridge-messages/).

## Result

The API is typed end to end, but we only pay for the validation we actually use:

- **Every packet is checked at the edge.** Malformed packets and unknown operations are rejected in the transport layer, before they reach the application.
- **New robot interfaces work immediately.** Payloads are relayed as-is by default, so a changed or new interface on a robot doesn't require changes to the API.
- **Validation where it matters.** When the API needs to work with a payload, for example to transform it or hide ROS types from clients, we add a model for that topic, service or action and its payload is validated from then on.
- **Clear boundaries.** Each layer has one responsibility, so transport, routing and the HTTP interface can evolve independently.
- **Extensible through plugins.** New endpoints and domain models can be added as plugins, so supporting a new robot or use case doesn't require changes to the core codebase.

This lets us iterate quickly on new robot concepts, and harden the API as those concepts mature.

