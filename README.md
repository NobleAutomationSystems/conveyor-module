# Modular Conveyor Belt (MVP)

Welcome to the **Modular Conveyor Belt MVP** — a core component of the open-source manufacturing automation platform. Think of it as "Factorio meets LEGO, but real." This project uses cheap commodity hardware and self-discovers on a local network to enable visual, drag-and-drop automation without customized engineering logic for every deployment.

## The Vision
- **Modular**: The conveyor belt is one piece of the puzzle. Other modules (arms, sensors, applicators) will be added to the ecosystem.
- **Cheap & General Purpose**: Built on affordable ESP32 hardware and commodity components.
- **Plug-and-Play**: Connect to WiFi, and the central "brain" finds it, making it immediately available in the UI.

## Repository Structure
- `/hardware` — Bills of materials (BOM), STL models, wiring diagrams, and research notes.
- `/firmware` — ESP32 firmware running the conveyor, handling network discovery, and driving motors/sensors.
- `/brain` — Future location for integrating this module with the central controller service.
- `/docs` — Assembly guides, architecture decisions, and troubleshooting logs.

## Getting Started
See the [Issues](https://github.com/NobleAutomationSystems/conveyor-module/issues) tab for active research tickets regarding hardware selection.
