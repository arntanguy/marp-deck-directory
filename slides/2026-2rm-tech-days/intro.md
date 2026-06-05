---
# Metadata
title: Evolution of software frameworks and build-system of LIRMM/JRL-AIST and buildsystems
date: 2026-06-04
author: Arnaud Tanguy 
affiliation: https://github.com/arntanguy
lang: en-US

# Marp config
theme: custom-theme
paginate: false
# Transitions documentation: https://github.com/marp-team/marp-cli/blob/v4.3.1/docs/bespoke-transitions/README.md
transition: coverflow
header: Retrospective and evolution of LIRMM/JRL control software and build systems
footer: |
  Tech Days 2026 - Retrospective and evolution of LIRMM/JRL control software and build systems
---

<style scoped>
  h1 a {
    color: inherit;
    text-decoration: underline;
  }
</style>

<!--
_class: lead
_paginate: false
_header: ""
_footer: ""
-->

# Retrospective and evolution of LIRMM/JRL control software and build systems

## Arnaud Tanguy - Research Engineer - University of Montpellier

**LIRMM**: Laboratoire d'informatique et de robotique de l'université de Montpellier
**JRL-AIST**: Joint Robotics Laboratory - Advanced Institute of Science and Technology - *Tsukuba, Japan*

---

<!-- _transition: none -->
<!-- header: Who am I? -->

# Who am I?
## Arnaud Tanguy

- Research engineer since 2018 with focus on humanoid robotics and background in computer vision
- Empower researchers to succeed in their projects 
- Make science reprocucible
- Any problem should only be hard to solve once 

---

## Background

- **2011-2014**: Software Engineer with major in computer vision (Polytech'Nice-Sophia)
  - 1 year in Trinity College Dublin: master's in interactive entertainment technology
  - Project with Andrew Comport: visualizing a dense SLAM map with augmented reality headset
  - 6 months internship at TUM Munich in Daniel Cremer's team
    - Siamese Neural Network for loop closure detection of Visual SLAM

---

## Background

- **2014-2018**: **Thesis** - Visual SLAM for humanoid localization and closed-loop control 
  - *Supervisors: Andrew Comport and Abderrahmane Kheddar*
  - *Labs: LIRMM, JRL-AIST, I3S Sophia-Antipolis*
- **2019**: Research Engineer@LIRMM
- **2020-2022**: Research Engineer@JRL-AIST, Japan
- **2022-2026**: Research engineer@LIRMM

---

## Main projects

- **DARPA Robotics Challenge** - *Team AIST-NEDO* - *2015*
  - International challenge on disaster scenario response
- **COMANOID**: *2015-2019*
  - *Multi-Contact Collaborative Humanoids in Aircraft Manufacturing*
- **ANA Avatar XPrize** - *Team Janus* - *2023*
  - Humanoid avatar
- **Locomanipulation of large industrial objects** - *2020-2023*

---
<!-- header: Objective -->

## Objective

- Presentation of the software stack used across most projects
  - `mc_rtc` and its dependencies
- Evolution of build systems
  - `cmake` `jrl-cmakemodules` `cmake + shell` `mc-rtc-superbuild` `nix`
- Towards reproducibility of software and demos
  - `devcontainers`, `nix`, `ci`
- Perspectives: how can we do more together?
  - reduce amount of duplicated dependencies
  - converge on common software packaging practices

---

<!-- header: Software stack evolution -->

## Context 

- **JRL-AIST**: Joint research lab (since 2018) between
  - **LIRMM**: Interactive Digital Human Team (IDH)
    - Led by Abderrahmane Kheddar > Sofiane Ramadi
  - **AIST**: Humanoid Research Group (HRG)
    - Led by Fumio Kanehiro
- Shared robots:
  - HRP-4 in LIRMM
  - HRP robots family with HRG group (`HRP-4J`, `HRP4-CR`, `HRP-2Kai`, `HRP-5P`)
  - More recently Kawasaki's `RHPS1`

---

## Research topics (~2012)

- Multi-robot QP control methods
  - Karim Bouyarmane, Joris Vaillant, Abderrahmane Kheddar
  - [Thesis Joris Vaillant](https://theses.fr/2015MONTS065) (2015): *Programmation de mouvements de locomotion et manipulation pour robots humanoïdes et expérimentations*
- Multi-contact planning
  - Adrien Escande, Stanislas Brossette

---

## Related software development

- [`SpaceVecAlg`](https://github.com/jrl-umi3218/SpaceVecAlg) implementation of Roy Featherstone's spatial vector algebra
- [`RBDyn`](https://github.com/jrl-umi3218/RBDyn) rigid body dynamics algorithms
- [`Tasks`](https://github.com/jrl-umi3218/Tasks) multi-robot QP control

Implemented in `C++`, with `Python bindings`

- Multi-contact planning tools and algorihms
- Communication bridge with HRP robots 

---

## But no formalized ecosystem 

- Projects / demos / use-cases multiply
- No formalized methodology -> everyone implements ad-hoc python code
- Little to no code re-usability outside of the core dependencies (`SpaceVecAlg`, `RBDyn`, ...)
- Ad-hoc use of simulators
- No continuous integration

---

<!-- header: DARPA Robotics Challenge -->

## DARPA Robotics Challenge

Challenge following Fukushima nuclear reactor meltdown:
- First virtual round: June 2013 > AIST
- Finals: June 2015 > AIST and JRL
- **8 challenging tasks**:
  - *LIRMM/JRL*: `driving`, `egressing the car`
  - *HRG (AIST)*: `opening door`, `valve opening`, `hole drilling`, `cable unplugging`, `debris field crossing`, `stair climbing`

---

## DARPA Robotics Challenge

- *HRG* has `hrpsys` and `hmc`
  - Very good at walking, manipulation is difficult
  - Strong coupling with `choreonoid` and `openrtm-aist` (ros-like middleware)
  - Graph based, and hard to manage
- *LIRMM/JRL*
  - Very good real-time multi-contact control algorithms (`Tasks`)
  - No unified framework yet

---

## DARPA Robotics Challenge - Inception of `mc_rtc`

We (LIRMM/JRL) need a real-time control software with the following constraints:
- Control must be computed at `1000Hz`
- Can be run on the robot -> `openrtm component`
- Multi-contact planning and qp control -> `SpaceVecAlg`/`RBDyn`/`Tasks`
- Can run multiple control scenarios (ladder climbing, driving, etc)
- Finite State Machine

> Same challenge, two software stacks

---

## Finalist, finish 10/23 

TODO: image

---

## But... 

<center>
<video src="./assets/drc_hrp2_fail.mp4" autoplay loop muted height="100%"></video>
</center>

---

![bg left](./assets/hrp2_debris_fall.webp)

## Lessons learned

- Don't divide by zero!
- Lacking CI
- Manual deployement processes
- Perception is a challenge
- Control needs more reactivity


---
<!-- header: Control framework: mc_rtc -->

# Control framework: mc_rtc

![height:450px](./assets/mc_rtc_architecture.jpg)

---

<!-- header: Control framework: mc_rtc - Extensibility -->
## We love extensibility 

Everything is a plugin:
- Controllers
- Robots
- FSM States
- Plugins

Easily extensible:
- Robot and simulator interfaces

---

<!-- header: Control framework: mc_rtc - Logging -->
## We love [Logging](https://jrl.cnrs.fr/mc_rtc/tutorials/usage/logging.html)

```cpp
logger().addLogEntry("entry_name", [this]() { /* compute, say x; */ return x; });
```

- Efficient binary logging with [MessagePack](https://msgpack.org)
- Can log any type supported by the framework (numerical types, arrays, sva types, etc)
- Easy to extend to new types
- Custom GUI
- Somewhat redundant with [DataTamer](https://github.com/PickNikRobotics/data_tamer) / [PlotJuggler](https://github.com/PlotJuggler/PlotJuggler)

---

## We love [Logging](https://jrl.cnrs.fr/mc_rtc/tutorials/usage/logging.html)

![height:450px](./assets/log_stair_climbing.png)

---


<!-- header: Control framework: mc_rtc - GUI -->
![bg left](./assets/mc_rtc_gui.png)

## We love [GUI](https://jrl.cnrs.fr/mc_rtc/tutorials/usage/gui.html)

GUI is implemented as a client-server

### Adding elements to the GUI

```cpp
using namespace mc_rtc::gui;
gui().addElement(
  {"Category", "Sub-category"},
    NumberInput("gain",
      []() { return gain_; },
      [](double newGain) { 
        gain_ = newGain; 
      })
);
```
---


## We love [GUI](https://jrl.cnrs.fr/mc_rtc/tutorials/usage/gui.html)

<iframe width="560" height="315" src="https://www.youtube.com/embed/t7_CbzjKDQg?si=hlVMWRDDObxPDK4m" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

<!-- header: Control framework: mc_rtc - Robot Modules -->

<!-- TODO make background transparent -->
<!-- ![bg](./assets/robot_family_bg.png) -->

## We love [robots](https://jrl.cnrs.fr/mc_rtc/tutorials/advanced/new-robot.html)

### Robot module

`C++` or `YAML` description that defines:
- how to find the robot's model (urdf, ...)
- additional properties needed by simulators/real robots
- about 20 robots supported


---


<!-- header: Control framework: mc_rtc - FSM Controllers -->
## We love [FSM Controllers](https://jrl.cnrs.fr/mc_rtc/tutorials/recipes/fsm-example.html)

Getting started with
```sh
mc_rtc_new_fsm_controller my_first_fsm_controller FSMController
```

File | Description
---|---
`etc/FSMController.yaml`|Your controller's FSM configuration file.
`src/FSMController.[h,cpp]`|Controller implementatioin (C++ or Python)
`src/states/`|Defines the controller's states (C++, Python or YAML)
`src/states/data`|YAML configuration for states
...|...


---

<!-- header: Control framework: mc_rtc - An FSM Controller is: -->

# An [FSM Controllers](https://jrl.cnrs.fr/mc_rtc/tutorials/recipes/fsm-example.html) is
### A `yaml` configuration

```yaml
robots:
  ground:
    module: env/door
states:
  Door_Initial: {} 
  TurnHandle: {}
transitions: 
  - [Door_Initial, OpenDoor, TurnHandle]
```

`...`

---

### `C++` states 

```cpp
struct Door_Initial : public State
{
  void configure(mc_rtc::Configuration &) override;
  void start(Controller &) override;
  bool run(Controller &) override;
  void teardown(Controller &) override;
};
```

`...`

---

### `C++` states 

```cpp
#include <mc_control/fsm/Controller.h>
#include "Door_Initial.h"

void Door_Initial::start(mc_control::fsm::Controller & ctl)
{
  // Add GUI button to open door
  ctl.gui()->addElement({}, mc_rtc::gui::Button("Open door", [this]() { openDoor_ = true; }));
}

bool Door_Initial::run(mc_control::fsm::Controller &)
{
  if(openDoor_)
  {
    output("OpenDoor"); // transition map output
    return true; // can move to next state
  }
  return false;
}


void Door_Initial::configure(const mc_rtc::Configuration &) {}
void Door_Initial::teardown(mc_control::fsm::Controller &) {}
EXPORT_SINGLE_STATE("Door_Initial", Door_Initial)
```

---

### `YAML` State Configurations

```yaml
Door::ReachHandle: # state name
  base: MetaTasks # base c++ state or yaml state, this state adds tasks to a QP solver
  tasks:
    RightHandTrajectory:
      type: surfaceTransform
      surface: RightGripper
      weight: 1000
      stiffness: 5
      targetSurface: # Target relative to the door's handle surface
        robot: door
        surface: Handle
        offset_translation: [0, 0, -0.025]
      completion:
        AND:
          - eval: 0.05
          - speed: 1e-4
```

---

### A `transition map`

```yaml
Door::OpenDoorFSM:
  base: Meta
  transitions:
    - [Door_Initial, OpenDoor, Door::ReachHandle, Auto]
    - [Door::ReachHandle, OK, Door::MoveHandle, Auto]
    - [Door::MoveHandle, OK, Door::OpenDoor, Auto]
```

---

<!-- header: Control framework: mc_rtc - Plugins -->
## We love external tools

Inherit from `mc_control::GlobalPlugin` and implement:

Plugin function|Description
---|---
`init()`|called by mc_rtc when the controller is initialized
`reset()`|called by mc_rtc when the controller is changed
`before()`|called by mc_rtc at the beginning of the run function
`after()`|called by mc_rtc at the end of the run function

Link against any external library and do what you need.

---

<!-- header: Control framework: mc_rtc - State Observation -->
## We love knowing where we are
### State observation pipeline
- Simple observers: `Encoder`, `BodySensor`, etc
- Composed and configured with a `yaml` configuration:

```yaml
ObserverPipelines:
  name: "LIPMWalkingObserverPipeline"
  observers:
    - type: Encoder
    - type: Attitude
      required: false
    - type: KinematicInertial
      config:
        anchorFrame:
          maxAnchorFrameDiscontinuity: 0.02
```

---

<!-- header: Project videos -->

# Now we can do hard things *easily*™
## COMANOID (2020)

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/fW7lFUToMjc?si=DjatLceG4nIIgb7Q&amp;controls=0&amp;start=98" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>

---

## Fast torquening demo (2022)

<iframe width="560" height="315" src="https://www.youtube.com/embed/oyjTN__T1eA?si=NFhvSeI4_TK2UcPw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Locomanipulation (2024)

<iframe width="560" height="315" src="https://www.youtube.com/embed/YR7xZGwIdmE?si=AnNOpXMxix6FNyTx&amp;controls=0&amp;start=96" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## ANA Avatar XPrize (2024)

<iframe width="560" height="315" src="https://www.youtube.com/embed/wYCbwBSrcNw?si=w9GUQXEAMnyQflxl" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


---

<!-- header: Build systems -->
# But...
## How
## do we
## build and maintain all this?

---

<!-- header: Build systems: it started out small -->
## It started out small

- Each project is built with `cmake`
- Dependencies are handled manually
- To avoid duplication `...`
- Not sustainable

---

## Build script

- `mc_rtc` itself has
  - N direct dependencies: sva, rbdyn, tasks, mesh_sampling, ndcurves
  - a few system dependencies: eigen, boost, fmt, spdlog

- Build handled by a `build_and_install.sh` script
  - Sets up system dependencies (ubuntu only)
  - Clones and installs projects in the right order

*Ok-ish for a small dependency graph known ahead of time*

---

## Not enough for a full ecosystem

Nowadays, `mc_rtc` ecosystem consists of:
- Hundreds of controllers, a dozen large highly active ones
  - Ex: [BaseLineWalking](), [LIPMWalking](), [PolytopeController](), [PandaProsthesis]()...
- Dozens of robot modules: `HRP*`, `Panda`, `UR`, `Unitree G1/H1/GO1`, etc
- Dozens of interfaces with robots/simulators: `mc_udp`, `mc_mujoco`, `mc_openrtm`, `mc_rtc_ros_control`, etc

Each with their own dependencies:
- vision frameworks: `visp`, `pcl`, `orb-slam`...
- solvers: `casadi`

---

## `mc-rtc-superbuild`: an extensible `cmake` superproject

=> Ubuntu only, hard-ish to maintain

---

## Devcontainers 

---

## `Nix` to the rescue 

---

# Conclusion

**Questions?** 🎤
