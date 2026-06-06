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
- To avoid duplication [jrl-cmakemodules](https://github.com/jrl-umi3218/jrl-cmakemodules)
  - factorize common patterns
  - simplify export of packages
  - shared between LIRMM (IDH), JRL, LAAS (gepetto), INRIA (willow)

> Ok for simple projects without many dependencies

---

## Build script

- `mc_rtc` itself has
  - `8` direct dependencies: `sch-core`, `sva`, `rbdyn`, `tasks`, `mesh_sampling`, `ndcurves`, `mc_rtc_data`
  - a few downstream projects: `mc_rtc_ros`, `mc-rtc-magnum`, robot interfaces
  - a few system dependencies: eigen, boost, fmt, spdlog

- Build mananged by a `build_and_install.sh` script
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

## Example: `BaseLineWalking`

Humanoid walking controller with many centroid trajectory generation methods

- Lots of dependencies: `QpSolverCollection`, `ForceControlCollection`, `TrajectoryCollection`, `NMPC`, `CentroidalControlCollection`, `BaselineFootstepPlanner`
  - Which themselves have lots of dependencies: `copra`
  - `QpSolverCollection`:   `eigen-qld`, `eigen-quadprog`, `jrl-qp`, `qpOASES`, `osqp-eigen`, `nasoq`, `hpipm`, `proxsuite`, `qpmad`
  - etc...

Too many to list, and a downstream project, we need a better solution.


---

## `mc-rtc-superbuild`: an extensible `cmake` superproject

Core idea:
- we already have cmake projects for everything
- cmake is good at handling a graph of targets and building targets in the right order
- cmake can run arbitrary commands

> - Write a meta cmake project that can fetch, build, install, update all projects and install their dependencies
> - Make it extensible with an extensions/ folder where users can add their own cmake scripts (submodules)

---

## `mc-rtc-superbuild` - example - `BaseLineWalking.cmake`

```cmake
include(${CMAKE_CURRENT_LIST_DIR}/../control/CentroidalControlCollection.cmake) # ...

set(BASELINE_WALKING_CONTROLLER_CMAKE_ARGS "")
if(WITH_ROS_SUPPORT)
  if($ENV{ROS_VERSION} EQUAL 2)
    set(BASELINE_WALKING_CONTROLLER_CMAKE_ARGS
      CMAKE_ARGS -DUSE_ROS2=ON)
  endif()
endif()

AddProject(BaseLineWalkingController
  GITHUB isri-aist/BaseLineWalkingController
  GIT_TAG origin/master
  DEPENDS CentroidalControlCollection BaseLineFootstepPlanner ForceControlCollection TrajectoryCollection ${BASELINE_WALKING_CONTROLLER_CMAKE_ARGS}
)
```

---

## `mc-rtc-superbuild` - example - `CentroidalControlCollection.cmake`

```cmake
include(${CMAKE_CURRENT_LIST_DIR}/ForceControlCollection.cmake)
include(${CMAKE_CURRENT_LIST_DIR}/NMPC.cmake)

AddProject(CentroidalControlCollection
  GITHUB isri-aist/CentroidalControlCollection
  GIT_TAG origin/master
  DEPENDS ForceControlCollection NMPC
)
```

---

## Supports ubuntu packages
```cmake
AddProject(
  mesh-sampling
  GITHUB jrl-umi3218/mesh_sampling
  GIT_TAG origin/master
  APT_PACKAGES libmesh-sampling-dev # if not building from source, install this apt package
  APT_DEPENDENCIES libgtest-dev libqhull-dev libassimp-dev # when buildng from source, depend on these apt packages
)
```

---

## Devcontainers

3 types of devcontainers can be automatically generated by CI from an `mc-rtc-superbuild` repository:
- `devcontainer`
  - pre-builds the whole superbuild environment: bootstrap, install all dependencies, store `ccache` output for fast recompilation within the container.
- `devel`
  - full image with all source, build and install output => can run the project as-is forever but still allow modification
- `relase`: only installed output => can execute the project

---

## mc-rtc-superbuild - lessons learned

### The good
- Easy to understand, most people already know cmake
- Can install APT, PIP dependencies, or any cmake project from source
- Worked great to manage complexity for a few years

### The bad
- Compilation heavy: most people don't release packages
- Only supports Ubuntu
- Hard to have project depend on different versions of a common dependency


---

## `Nix` to the rescue 

### What we really want is:

- Declarative source of truth
- Reproducible builds
- Reproducible depelopper environment 
- Custom dependencies for different projects 
- Binary cache

---

### Writing a derivation

```nix
{ stdenv, lib, cmake, pkg-config, jrl-cmakemodules, doxygen,
  eigen, boost, fetchFromGitHub, python3Packages }:
stdenv.mkDerivation {
  pname = "spacevecalg"; version = "1.2.10";
  src = fetchFromGitHub {
    owner = "jrl-umi3218"; repo = "SpaceVecAlg"; tag = "v1.2.10";
    hash = "sha256-fTKKj3m8cO4F46LlO7r8JeuWLhlyRcX7EblHroDYFkQ=";
  };
  nativeBuildInputs = [ cmake jrl-cmakemodules pkg-config doxygen ] 
    ++ (with python3Packages; [ cython python distutils pytest ]);
  propagatedBuildInputs = [ eigen boost ]
    ++ (with python3Packages; [ numpy eigen3-to-python ]);
  cmakeFlags = [ "-DINSTALL_DOCUMENTATION=OFF" ];
  meta = with lib; {
    description = "Spatial Vector Algebra with the Eigen library";
    homepage = "https://github.com/jrl-umi3218/SpaceVecAlg";
    license = licenses.bsd2;
    platforms = platforms.all;
  };
}
```

---

### Writing an overlay

```nix
{ ...}: final: prev:
{
  spacevecalg = prev.callPackage ./pkgs/spacevecalg { };
  rbdyn = prev.callPackage ./pkgs/rbdyn { };
  g1-mj-description = prev.callPackage ./pkgs/mc-rtc/mc-mujoco/robots/g1-mj-description.nix { };
  mc-mujoco-robots = prev.callPackage ./pkgs/mc-rtc/mc-mujoco/robots/default.nix { };
  # mc-mujoco with all public robots
  mc-mujoco-robots-public = prev.callPackage ./pkgs/mc-rtc/mc-mujoco/robots/default.nix {
    robots = [
      final.g1-mj-description
      final.h1-mj-description
      final.ur5e-mj-description
    ];
  };
  # ...
};
```

---

### [Flakoboros](https://github.com/Gepetto/flakoboros): circular packaging concept

- Developped by Guilhem Saurel @LAAS
- Idea:
  - One central repository that defines how to build packages we want:
    - [gepetto/nix](https://github.com/gepetto/nix), [mc-rtc/nixpkgs](https://github.com/mc-rtc/nixpkgs)
  - Each project defines a `flake.nix` that:
    - knows how to build the project from the latest source
    - provides a development shell with everything needed to work on the project
    - can override/add dependencies
  - CI builds, checks, and publish binary cache

---

### Example use: [PolytopeController](https://github.com/Hugo-L3174/polytopeController)

Hugo Lefevre's work on dynamic balance.

This project is a work-in-progress that depends on many other changes:
- A custom version of `tvm`
- A branch of `mc_rtc` with large-scale changes
- A branch of `state-observation` to adapt to `mc_rtc`
- A branch of `mc_dynamic_polytopes` 
- A branch of `mc_force_shoe_plugin`

---

### Example use: [PolytopeController](https://github.com/Hugo-L3174/polytopeController)

```nix
  inputs = {
    mc-rtc-nix.url = "github:mc-rtc/nixpkgs"; # global package repository
    flake-parts.follows = "mc-rtc-nix/flake-parts";
    systems.follows = "mc-rtc-nix/systems";

    # overrides
    mc-state-observation.url = "github:jrl-umi3218/mc_state_observation/pull/57/head";
    mc-state-observation.flake = false;
    dcm-vrptask.url = "github:Hugo-L3174/DCM_VRPTask/pull/1/head";
    dcm-vrptask.flake = false;
    mc-dynamic-polytopes.url = "github:Hugo-L3174/mc_dynamic_polytopes/pull/6/head";
    mc-dynamic-polytopes.flake = false;
    mc-force-shoe-plugin.url = "github:Hugo-L3174/mc_force_shoe_plugin/pull/16/head";
    mc-rtc.url = "github:jrl-umi3218/mc_rtc/pull/507/head";
    # mc-rtc.url = "path:/home/arnaud/devel/mc-rtc-nix/workspace/mc_rtc";
  };
```

---

### Example use: [PolytopeController](https://github.com/Hugo-L3174/polytopeController)

```nix

flakoboros = {
  overrideAttrs.mc-force-shoe-plugin = {
    src = inputs.mc-force-shoe-plugin;
  };

  overrideAttrs.polytopeController = {
    src = lib.cleanSource ./.;
  };
};

```

---

### Example use: [PolytopeController](https://github.com/Hugo-L3174/polytopeController)

```nix

flakoboros = {
  overrides.mc-rtc-superbuild = { pkgs-final, pkgs-prev, drv-final, drv-prev, ... }:
    let cfg-prev = drv-prev.superbuildArgs; in
    {
      superbuildArgs = cfg-prev // {
        pname = "mc-rtc-superbuild-hugo";
        robots = [ pkgs-final.mc-rhps1 ];
        controllers = [ pkgs-final.polytopeController ];
        configs = [ "${pkgs-final.polytopeController}/lib/mc_controller/etc/mc_rtc.yaml" ];
        plugins = [ pkgs-final.mc-force-shoe-plugin ];
        observers = [ pkgs-final.mc-state-observation ];
      };
    };
};
```

---

### Example use: [PolytopeController](https://github.com/Hugo-L3174/polytopeController)

```nix
  outputs = inputs: inputs.flake-parts.lib.mkFlake { inherit inputs; } ({ lib, ... }:
  {
    imports = [
      inputs.mc-rtc-nix.flakeModulePrivate
      {
        flakoboros = {
          # Override needed dependencies
          overrideAttrs.mc-force-shoe-plugin = {
            src = inputs.mc-force-shoe-plugin;
          };

          overrideAttrs.polytopeController = {
            src = lib.cleanSource ./.;
          };

          overrideAttrs.mc-state-observation = {
              src = inputs.mc-state-observation;
            };

          overrideAttrs.dcm-vrptask = {
            src = inputs.dcm-vrptask;
          };

          overrideAttrs.politopix = {
              src =
                builtins.trace "politopix is currently a private repository, ask I2S Bordeaux to make it public"
                  (
                    builtins.fetchGit {
                      url = "git@github.com:Hugo-L3174/politopix";
                      rev = "f625b42de4404eea16aabcf720f2cee19dfdc406";
                    }
                  );
            };

          overrideAttrs.mc-dynamic-polytopes = {
            src = inputs.mc-dynamic-polytopes;
          };

          overrideAttrs.tvm =
            { pkgs-final, ... }:
            {
              src = pkgs-final.fetchgit {
                # tvm pr 53
                url = "https://github.com/Hugo-L3174/tvm.git";
                rev = "4e6640660317dd9e311fc707de689c4cf984ee50";
                sha256 = "sha256-Mzx7J3yp9pcoOf5VMkma1sNs8uCqjgnCsZZYlHBGLE4=";
              };
            };

          overrideAttrs.mc-rtc =
            { ... }:
            {
              pname = "mc-rtc-hugo";
              src = inputs.mc-rtc;
            };

          overrides.mc-mujoco-robots =
            { pkgs-final, ... }:
            {
              robots = with pkgs-final; [
                hrp4-mj-description
                rhps1-mj-description
              ];
            };

          # overrides override package function arguments, while overrideAttrs overrides the attribute set
          overrides.mc-rtc-superbuild =
            { pkgs-final, pkgs-prev, ... }:
            let
              cfg-prev = pkgs-prev.mc-rtc-superbuild.superbuildArgs;
            in
            {
              superbuildArgs = cfg-prev // {
                pname = "mc-rtc-superbuild-hugo";
                robots = [ pkgs-final.mc-rhps1 ];
                controllers = [ pkgs-final.polytopeController ];
                configs = [ "${pkgs-final.polytopeController}/lib/mc_controller/etc/mc_rtc.yaml" ];
                plugins = [ pkgs-final.mc-force-shoe-plugin ];
                observers = [ pkgs-final.mc-state-observation ];
              };
            };

        };
      }
    ];
    perSystem =
      { pkgs, ... }:
      {
        devShells.default =
          (pkgs.callPackage "${inputs.mc-rtc-nix}/shell.nix" {
            inherit (pkgs) mc-rtc-superbuild;
          }).overrideAttrs
            (old: {
              shellHook = ''
                ${old.shellHook or ""}

                # Your custom shellHook commands
                export POLYTOPE_CONTROLLER="${pkgs.polytopeController}/lib/mc_controller/etc/mc_rtc.yaml"
                export POLYTOPE_CONTROLLER_MUJOCO="${pkgs.polytopeController}/lib/mc_controller/etc/mc_rtc_MuJoCo.yaml"
              '';
            });
      };
  }
```

---

#### What have we gained?

- Central `mc-rtc/nixpkgs` repository knows how to build most packages
  - Their dependency graph, how to build it, etc
- `PolytopeController`
  - `flake.nix` defines what changes are needed to build itself
  - Locks dependencies in a `flake.lock` file
  - Different dependency tree does not clash with other projects
  - CI can build, check, and deploy an up-to-date binary cache
- Running `nix develop`
  - Pulls all dependencies from the binary cache
  - Provides a development environment

---

#### Downsides?

- Very few people are familiar with `nix`
- Rebuilding a dependent package rebuilds everything that depends on it
  - Good: we ensure ABI is correct
  - Bad: takes time

---

## Taking it further: NixOS

### Repeatable, yet flexible OS declaration


---

# Conclusion

**Questions?** 🎤
