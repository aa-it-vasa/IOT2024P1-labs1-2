# System Architecture of IoT - Labs
## Goals and Overview

The emerging and existing IoT frameworks share some central IoT architectural
features. A central pattern seems to be about discovering, connecting,
monitoring, controlling and/or interacting with a system of _smart things_ or
_Things_ (with capital T), where a Thing is an abstract notion of a digitally
augmented physical object. Digital augmentation is one or more combination of
sensors, actuators, computation element or communication interfaces attached to
the physical object. So, a Thing can be a circuit package with one sensor, or
multiple sensors and actuator, or even a group of devices mimicking a physical
object.

[Three
patterns](https://www.w3.org/Submission/wot-model/#web-things-integration-patterns)
emerge when it comes to binding the Things together. Direct Connectivity refers to
a pattern where each Thing is directly connected to another Thing. Each Thing
has its own application program interface (API) and they communicate with each
other using a standardised API. However, this is indeed the most difficult
approach due to diverse communication standards, lack of processing and
operational power on the devices to support full-fledged networking stack, and
lack of standardisation of the API.
[W3C](https://www.w3.org/Submission/wot-model) is coming up with one such
standard and is yet to be ratified. 

Gateway-based connectivity is the second pattern, where a Thing cannot offer an
API directly and sits behind an intermediary. The gateway mimics the Thing and
provides an API on behalf of the Thing. The last common pattern is the
cloud-based connectivity. This is similar to a gateway approach, the cloud
service provides the API for the Things. The difference lies in the fact that
the gateway can be physically closer to the Things. The interaction between
gateway/cloud and the Things themselves could be orchestrated through a tightly
coupled, proprietary communication protocols, that is lighter on power and
computational load when compared to full internet stack.

In the lab, we will focus on the cloud and gateway pattern. We will use MQTT
protocol to directly interact with the devices first and then subsequently
conduct the interactions through a gateway. In process, the goal is to
recognise some good practices when it comes to deploying a Thing network and
this is demonstrated using Amazon's IoT and Greengrass product. We will cover
common security practices that involve authorisation and authentication issues.
We will also touch upon elastic edge computing, if time permits.

Concretely, the lab exercises are designed to achieve the following learning
goals:

1. Lab 1: Get a working understanding of the MQTT protocol. Setup a rudimentary
   publisher and subscriber using a openly available broker.
2. Lab 2: Establish a secure MQTT connection and understand various parts of
   the setup
3. Lab 3: Establish a MQTT connection that is safe from (unintended) snooping
   and is private using a gateway.
4. Lab 4: Setup an edge - a local computing platform on the gateway for your
   devices to offload its computation. Connect a device to the edge and perform
   some computations. 

## Grading and Expectations

### Lab reports
The lab work is graded based on a sumitted report (one report per lab/group) as well
as the code you have in your repository. The report is written in Markdown format in the files `Report_labN.md` files. 

Note that the last committed version on the `main`-branch at the deadline is used for grading.

Your own code should be placed in a subfolder named `code/Lab*` (where `*` is the lab number) 
in the main branch.

Each lab exercise has a set of questions in the end that is to be answered, with the
goal of demonstrating working knowledge of concepts involved. You are
encouraged to provide any pictures, graphs, screenshots, diagrams etc. that
help explain your solution. If you use content (picture, graphs, etc.)
from other sources, remember to properly cite and provide reference to the used
external sources. Refer to the code you have in your repository using links. 

### Learning reflections
You should also provide a reflection on what you learned during these labs in 
the file `lab_reflection.md`. This file should provide answers to the following questions:

1. Have you learned anything new?
2. Did anything surprise you?
3. Did you find anything challenging?
4. Did you find anything satisfying?


The report and reflections should contain full name of each group member.

The hard deadline for submission is **20.03.2023 23:55**. Note that each group will also need
to submit a link to your repo in Moodle before the deadline.

## Setup your computer 
To perform lab exercises, you need to setup your computer. No extra computers will be provided and it is your responsibility to setup your computer. It is assumed that you know some basic commands of Linux. 

For the Labs your will use a Virtualbox Ubuntu-based virtual machine (VM) as a development environment step. 
You should:

1. Download and install Virtualbox on your laptop: https://www.virtualbox.org/wiki/Downloads 
2. Download the VM from the course Team on Microsoft Teams and channel _Labs_.

3. Import the downloaded VM into Virtualbox by selecting the downloaded .ova file. The current VM has 4GB of RAM - Feel free to adjust it based on your own laptop configuration.

4. Check you can start the VM. 

5. Clone your repository on the VM. More information [here](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository). You might need to setup some authentication. 

- **The username and password are iotlabs/iotlabs.**
- Feel free to change the screen resolution in Ubuntu or to scale in/out the virtual screen. 
- Feel free to create a shared folder between your physical machine and VM (Settings->Shared folder). The following mount point can be used: /home/iotlabs/SharedFolder
- The VM has Atom and VS Code installed as a possible source code editor.

**If you have issues when importing the above VM,** you should the other version (also available in Teams) which uses the Open Virtualisation Format 2.0.

**If you have issue when booting the VM**, you should try to enable "3D acceleration" in Settings->Display before starting the imported VM. Also try increasing the amount of video memory and disabling audio support.

**If you have disk capacity issue,** you can increase the size of the VM disk using the Tools menu. Go to Tools->Virtual Media Manager->Select 'IoT-Labs 2023-Disk001.vdi'-> increase the disk size -> click on 'Apply'.
