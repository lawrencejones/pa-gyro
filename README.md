# Inertial Measurement Unit

## Project Summary

The aim of this project was to create a prototype board to trial
the accuracy and reliability of current market sensors, specifically
gyroscopes and accelerometers. There are many uses for a device capable
of combining data from both types of sensor, as given accurate
information from both, one can determine your location in respect to
a given origin.

The solution contained within this repository is broken into a few
components. At the most basic, it consists of a hardware device (in
this case a circuit) that hosts the sensors, a driver component to
read the information from the hardware and a software layer that
performs dsp on the raw data.

## Current Project State

The implementation can be broken down into the following steps,
completion indicated by a tick.

- [X] Research suitable components
- [X] Design circuit board to host chosen sensors
- [X] Fabricate board and assemble
- [X] Implement drivers for board/individual sensors
- [X] Implement digital signal processing
- [ ] Implement link between driver data and dsp
- [ ] Implement web gui for data analysis

Below is a breakdown of each stage of the project, what files are
related to which section and what still needs to be done.

## 1. Research Suitable Components

Included in the repository is the inital report written to propose
the project to the partners. It explains the motivation behind the
project. The current state of the market shows that gyroscopes are
the weakest link in an  Inertial Measurement Unit and so our focus
was on improving gyroscope accuracy. As such, we selected our gyros
from the current market leader, Invensense.

Three different price levels were chosen.
  - MPU3300 ~ £60
  - ITG3050 ~ £13
  - L3G4200D ~ £2  (not Invensense)

Due to time constraints the project only covers implementation for
the two top price categories, though due to the relative poor performance
of the L3G4 this does not appear to be a hinderance in answering
the initial question about IMU potential.

## 2. Design Circuit Board and Assemble

Located in the hw submodule are the Eagle CAD files for the
production of the circuit. PCBTrain we're used in the production
of the circuit and for future use I'll outline the procedure below.

### Eagle & PCBTrain

Eagle is a CAD design package that is used for circuit production.
The steps required to produce the files found in the circuit folder
are...

  1. Open Eagle, select `File > New Project` to create a new project space
  2. Once a project is created, `File > New Schematic` will create a new
     schematic file, used to host the diagramatic circuit schematic
  3. Various tutorials exist online to explain the Eagle software, though
     as a quick start: 
       - draw wires using the Wire/Draw lines tool
       - pick a sensible grid spacing to begin, this will avoid configuration
         issues further down the process
       - if you can, source the components that you use from online. They are
         typically more accurate and save you the time to produce them yourself
       - should a component package not be available, then there is software
         available to generate component files from a pdf specification
       - keep a library saved inside your project folder that contains all
         your created components
  4. Design the schematic in a conventional manner and assuming the ERC
     passes then move on to the board layout
  5. Autorouting is not going to work for anything more than a very simple
     circuit, therefore begin board routing with sensible placement of
     components. Use a modular design structure and learn the command
     shortcuts for faster manipulation of the board layout
  6. Once components are placed, run a ratsnest to generate links between
     chip terminals etc. Then proceed to connect ratsnest suggested links
     using the `Route manually` tool. NB - This does take time, rushing will
     only lengthen the process as mistakes early on are costly. Think
     carefully about initial placement to avoid running out of space later.
  7. On completion of the routing, you can optionally render a polygon to
     act as a shared ground/other plane. This can simplify routing. Look
     online for further details.
  8. Once satisfied with the routing, run the PCBTrain design rule check
     to verify pad spacing, wire width etc. If this passes then the pcb
     should be ready to submit for production

Submission is performed on the PCBTrain website.

## 3. Fabricate Board and Assemble

Included in the hardware for this project is a fully assembled PCB following the design files given in this repo. It has been populated with the chosen chips and via the web interface tested for access and health of the gyros, of which all but one appear to be working correctly.

There are also extra boards to be used for redundancy/replacement, though it is important to note there is an error in this revision that has the Raspberry Pi pin connects shifted by 1 for the I2C lines. This has been fixed in the latest design file, though the header does not necessarily translate directly to the Rev.B Pi.

## 4. Implement Drivers for Board and Sensors

Drivers have been written for the MPU3300, the ITG3050 and the PCA9548a. These allow communication with the two most expensive gyros, and expose an interface to configure the sensors behaviour via key value pairs. This is done via the cli provided in the IMU binary file (build/imu/imu).

## 5. Implement Digital Signal Processing

DSP has been implemented in terms of Kalman Filters and Allan Variance, though the dsp source has not yet been integrated with the rest of the project. A header file (src/dsp.h) shows the suggested interface for this library though we did not have the time to configure the current source to match. However assuming the dsp library is improved to match the header interface, then the library should integrate correctly with the piping of device data to fifos.

## 6. Implement Link Between Driver Data and DSP

Our current suggestion of how drivers should output their data is via a system fifo, the implementation of which may be found in the `src/dev/shared/dev_pipe.c` file. This pipe function can be attached to the various structs representing sensors on the board, and called to create a system file to contain the data. Our future plans were to pipe data through the DSP library to allow the system to read processed data in real time, though the linking has not yet been complete.

## 7. Implement Web GUI for Data Analysis

This is the web server component of the project, where we wish to display our progress in real time. As so far we managed to allow debugging and health checking of the sensors to be displayed on the site which should allow for rapid testing of components, and simulated data feed examples can be seen in previous commits prior to the restructuring.

This step needs only to have the server create a web socket and feed the data fetched from an fifo buffer to the client in json format. A very similar system is used for the LeapMotion Javascript SDK, should an example be required.

