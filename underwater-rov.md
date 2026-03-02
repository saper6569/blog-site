---
layout: default
title: Underwater ROV
---

# Underwater ROV


This project is being developed by me and two friends at Queen's University as a hands-on engineering initiative focused on building a practical underwater monitoring platform. Our goal is to design and prototype a tethered submarine capable of capturing real-time underwater video and environmental data while remaining modular, affordable, and field-deployable. Beyond being a technical challenge, this project is also an opportunity for us to apply concepts from embedded systems, electronics, and system design to a real-world environment.

Rivers and lakes present unique engineering challenges. Water clarity, current flow, debris, and limited accessibility make inspection and monitoring difficult. Traditional methods such as manual sampling or diver-based inspection can be time-consuming, expensive, and sometimes unsafe. Our project aims to create a compact remotely operated vehicle (ROV) that can be deployed from shore to visually inspect underwater environments, monitor debris accumulation, and serve as a foundation for future environmental sensing applications.

The system uses a tethered architecture to ensure stable communication and continuous power delivery. Wireless communication is unreliable underwater, so a physical tether provides a robust connection between the submarine and a shore-based control station. The tether carries power and data, enabling real-time video streaming and bidirectional control without relying entirely on onboard batteries. This design choice simplifies the vehicle, reduces the risk of power failure underwater, and allows for longer operation times during testing.

At the hardware level, the platform is built around two main processing components: an ESP32 microcontroller and a Raspberry Pi. The ESP32 handles low-level control tasks such as motor control and sensor interfacing, providing reliable real-time performance for thrusters and hardware management. The Raspberry Pi serves as a higher-level companion computer, responsible for video capture, encoding, and network communication with the shore station. Separating these responsibilities improves system stability and makes the platform easier to expand in the future.

For imaging, we are using a Raspberry Pi Camera module to stream live underwater video through the tether to a surface computer. The system is intentionally modular so that additional sensors, such as depth sensors, temperature sensors, or inertial measurement units, can be integrated as the project evolves.

The tether itself is based on Ethernet-style cabling (such as Cat6), allowing structured twisted-pair wiring for reliable data transmission and simplified integration with standard networking hardware. We are carefully considering voltage drop, waterproofing, mechanical strain relief, and buoyancy to ensure the system can operate reliably in real conditions.

Once the prototype is fully assembled and tested, we hope to deploy and test it in the lake in Kingston to evaluate performance in a real aquatic environment. Field testing will allow us to assess video quality, tether handling, maneuverability, and overall system robustness outside of a lab setting.

Overall, this project represents a collaborative effort to design, build, and test a functional underwater robotic platform from the ground up. It combines embedded systems, networking, power distribution, and mechanical design into a cohesive system—and serves as a stepping stone toward more advanced environmental monitoring and autonomous underwater technologies.
