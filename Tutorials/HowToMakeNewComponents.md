# How To Make New Components

## 1.  Overview

- In the [How To Add Components](./HowToAddComponents.md) tutorial, we have added existing components to the simulation scenario.
- This tutorial explains how to make a new component into the s2e-user directory.
  - **Note**: You can move the source files for the new component into the [s2e-core](https://github.com/ut-issl/s2e-core) repository if the component is useful for all S2E users.
- The supported version of this document
  - Please confirm that the version of the documents and s2e-core is compatible.


## 2. Overview of component expression in S2E

- Source codes emulating components are stored in the `s2e-core/src/components` directory.
- All components need to inherit the base class `Component` for general functions of components, and most of the components also inherit the base class `ILoggable` for the log output function.
- `Component` class
  - The base class has an important virtual function `MainRoutine`, and subclasses need to define it in their codes.
    - When an instance of the component class is created, the `MainRoutine` function is registered in the `TickToComponent`, and it will be automatically executed in the `Spacecraft` class.
    - The main features of the components such as observation, generate force, noise addition, communication, etc... should be written in this function.
  - `PowerOffRoutine` is also important especially for actuators. This function is called when the component power line is turned off. Users can stop force and torque generation and initialize the component states.
- Power related functions
  - `SetPowerState`, `GetCurrent` are power related functions. If you want to emulate power consumption and switch control, you need to use the functions.
- `ILoggable` class
  - This base class has two important virtual functions `GetLogHeader` and `GetLogValue`, for CSV log output.
  - These functions are registered into the log output list when the components are added in `UserComponents::LogSetUp` 
- Communication with OBC
  - If users want to emulate the communication(telemetry and command) between the components and the OBC, they can use the base class `UartCommunicationWithObc` or `I2cTargetCommunicationWithObc`.
  - These base classes also support a feature to execute HILS function.


## 3. Make a simple clock sensor

- This chapter explains how to make a simple clock sensor, which observes the simulation elapsed time with a bias noise.

1. Copy the following files in the directory `./Tutorials/SampleCode/ClockSensor` to the directory `s2e-user/src/Components`.
   - `ClockSensor.cpp`
   - `ClockSensor.hpp`

2. Edit `./s2e-user/CMakeList.txt` to add target source files for the compiler. Please add the following description in `set(SOURCE_FILES)`

   ```
   src/Components/ClockSensor.cpp
   ```

3. Build `s2e-user` and check there is no error.

4. Edit `UserComponent.hpp` and `UserComponent.cpp` as referring [How To Add Components](./HowToAddComponents.md)

   - The constructor of the `ClockSensor` requires arguments as `prescaler`, `clock_gen`, `sim_time`, and `bias_sec`.
   - `prescaler` and `bias_sec` are user setting parameters for the sensor, and you can set these values.
   - `clock_gen` is an argument for the `ComponentBase` class.
   - `sim_time` is a specific argument for the clock sensor to get time information. `SimTime` class is managed in the `GlobalEnvironment`, and the `GlobalEnvironment` is instantiated in the `SimulationCase` class.
   - You need to edit the `UserComponents.cpp` as follows.
     - Instantiate the `ClockSensor` in the constructor.
     ```c++
     clock_sensor_ = new ClockSensor(10,clock_gen,&glo_env->GetSimTime(),0.001);
     ```
     - Delete the `clock_sensor_` in the destructor.
     ```c++
     delete clock_sensor_;
     ```
     - Add log set up into the `CompoLogSetUp` function.
     ```c++
     logger.AddLoggable(clock_sensor_);
     ```

5. Build `s2e-user` and execute it

6. Check the log file to confirm the output of the `clock_sensor`
   - The output of the clock sensor has an offset error, and the update frequency is decided by the `prescaler` and the `CompoUpdateIntervalSec` in the base ini file.

## 4. Make an initialize file for the clock sensor

- Usually, we want to change the parameters of components without rebuilding such as noise properties, mounting coordinates, and so on. So this section explains how to make an initialize file for the `ClockSensor`.

1. Copy the following files in the directory `./Tutorials/SampleCode/ClockSensor` to the directory `s2e-user/src/Components`.
   - `InitClockSensor.cpp`
   - `InitClockSensor.hpp`

2. Edit `./s2e-user/CMakeList.txt` to add target source files for the compiler. Please add the following description in `set(SOURCE_FILES)`

   ```
   src/Components/InitClockSensor.cpp
   ```

4. Edit the `UserComponents.cpp` as follows
   - Add include files
     ```c++
     #include "../../Components/InitClockSensor.hpp"
     ```
   - Edit making instance of the `ClockSensor` at the constructor
     ```c++
     std::string clock_sensor_ini_path = iniAccess.ReadString("COMPONENTS_FILE", "clock_sensor_file");
     clock_sensor_ = new ClockSensor(InitClockSensor(clock_gen, glo_env->GetSimTime(), clock_sensor_ini_path));
     ```

6. Copy `ClockSensor.ini` into `s2e-user/data/ini/components` from `./Tutorial/SampleCodes/ClockSensor`

7. Edit `UserSat.ini` to add the following line at the [COMPONENTS_FILE] section of the file

   ```c++
   clock_sensor_file = ../../data/ini/components/ClockSensor.ini
   ```

8. Build `s2e-user` and execute it

9. Check the log file 

10. Edit the `ClockSensor.ini` and rerun the `s2e-user` to confirm the initialize file can affect the result.
