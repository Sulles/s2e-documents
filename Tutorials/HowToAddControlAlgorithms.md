# How To Add Control Algorithms

## 1.  Overview
- In the [How To Make New Components](./HowToMakeNewComponents.md) tutorial, we have newly made components emulating codes in [s2e-core](https://github.com/ut-issl/s2e-core) and adding the new components into our simulation scenario.
- Now we can simulate the behavior of spacecraft **free motion** and emulate the behavior of sensors and actuators. 
- This tutorial explains how to add a **Control Algorithm** into the simulation scenario. 
- For a practical satellite project, we should implement the control algorithm as actual flight software like [C2A](https://github.com/ut-issl/c2a-core) into the S2E. However, using actual flight software is usually overdoing for use cases as research, the initial phase of satellite projects.
- So, we introduce the following three methods, and users can choose a suitable method.
  - Direct method: Directly control physical quantity without sensors, actuators, and their noises
    - For theoretical researches and preliminary analysis for satellite projects
  - Component method: Control using sensors and actuators without flight S/W framework
    - For engineering researches and preliminary analysis for satellite projects
  - Flight S/W method: Control using sensors and actuators with flight S/W framework
    - For actual satellite projects
- The supported version of this document
  - Please confirm that the version of the documents and s2e-core is compatible.

## 2. Direct method
- This chapter introduces how to add a control algorithm without sensors and actuators.
- This method directly measures the satellite's physical quantity and generates torque and force acting on the satellite.
- To do that, users need to edit the `Update` function in the `UserSat.cpp`.
  - A sample code is in `./Tutorials/SampleCodes/control_algorithm/direct_method/user_satellite.cpp`
- The `UserSatellite` class already has satellite attitude, orbit, and local environment information since it inherits the `Spacecraft` base class. So users can easily access these values.
- To measure physical quantities, users can use getter functions defined in the `Attitude`, `Orbit`, and `LocalEnvironment` classes as `dynamics_->GetAttitude().GetAngularVelocity_b_rad_s()`.
- To generate torque and force, users can use `dynamics_->AddTorque_b_Nm` and `dynamics_->AddForce_b_N`.
- The sample codes are in `SampleCodes/control_algorithm/direct_method/user_satellite.cpp`, and you can see very simple detumbling with the proportional control method.
- By using the sample code with initial angular velocity = [0.05, -0.03, 0.01] rad/s, the following results are given.
  - You need to edit the initialize file to set the initial angular velocity.
    
    <img src="./figs/ControlAlgorithm_DirectControl_result1.png" alt="CA_DC_1" style="zoom: 80%;" />  

    <img src="./figs/ControlAlgorithm_DirectControl_result2.png" alt="CA_DC_2" style="zoom: 80%;" />  
   
    <img src="./figs/ControlAlgorithm_DirectControl_result3.png" alt="CA_DC_3" style="zoom: 80%;" />  

## 3. Component method: Using ideal components

## 4. Component method: Using real components
- This chapter introduces a method to add a control algorithm using sensors and actuators.
- This method measures a satellite's physical quantity via sensors, generates torque and force via actuators, and executes control algorithms on OBC.
- This tutorial assumes the spacecraft has a three-axis gyro sensor, a reaction wheel, and an OBC.
- The sample codes are in `SampleCodes/ControlAlgorithm/ComponentMethod/`
- Firstly, users need to make the `UserOBC` class to emulate the OBC.
  - Copy the `UserObc` files to the `s2e-user/src/Components` from the `ComponentMethod/src/Components`, and add the `UserObc.cpp` to the `set(SOURCE_FILES)` in the `CMakeLists.txt` to compile it.
  - The `UserOBC` class has the `UserComponent` class as a member, and users can access all components to get sensing information or set the output of actuators.
  - In this tutorial, the angular velocity is measured by the gyro sensor. RW's output torque is calculated using the X-axis of the measured angular velocity, and the torque is set to RW.
- Next, users need to add the `UserObc` into the `UserComponent` class. You can copy the `UserComponents` files to the `s2e-user/src/Simulation/Spacecraft` from the `ComponentMethod/src/Simulation`.
- Finally, users need to add new source codes to the `CMakeLists.txt` to compile them.
  - You have to add `RwXxx.ini` to `s2e-user/data/ini/Components`
  - Refer to `SampleCodes/ControlAlgolithm/ComponentMethod/data/RwXxx.ini` if necessary.
- By using the sample code, the following results are given.
  - The X-axis angular velocity is controlled, but other axes are not controlled well since the satellite only has an RW on X-axis. The X-axis angular velocity has offset value since the gyro has offset noise.

    <img src="./figs/ControlAlgorithm_ComponentControl_result1.png" alt="CA_CC_1" style="zoom: 80%;" />  
    <img src="./figs/ControlAlgorithm_ComponentControl_result2.png" alt="CA_CC_2" style="zoom: 80%;" />  
    <img src="./figs/ControlAlgorithm_ComponentControl_result3.png" alt="CA_CC_3" style="zoom: 80%;" />  

  - The following two figure shows the observed angular velocity by gyro and the rotation speed of the RW. You can find the observed X-axis angular velocity reaches zero by the control.

    <img src="./figs/ControlAlgorithm_ComponentControl_result4.png" alt="CA_CC_4" style="zoom: 80%;" />  
    <img src="./figs/ControlAlgorithm_ComponentControl_result5.png" alt="CA_CC_5" style="zoom: 80%;" />  

## 5. FlightSW method: Control algorithm within C2A
- TBW
