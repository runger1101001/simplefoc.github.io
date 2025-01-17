---
layout: default
title: Stepper Driver 4PWM
nav_order: 1
permalink: /stepper_driver_4pwm
parent: StepperDriver
grand_parent: Driver code
grand_grand_parent: Writing the Code
grand_grand_grand_parent: Arduino <span class="simple">Simple<span class="foc">FOC</span>library</span>
---

# Stepper Driver - `StepperDriver4PWM`

This is the class which provides an abstraction layer of most of the common 4 PWM stepper drivers out there. Basically any stepper driver board that can be run using 4 PWM signals can be represented with this class.
Examples:
- L298N
- MX1508
- Shield R3 DC Motor Driver Module
- etc.


<img src="extras/Images/stepper4pwm.png" class="width60">

## Step 1. Hardware setup
To create the interface to the stepper driver you need to specify the 4 `pwm` pin numbers for each motor phase and optionally `enable` pin for each phase `en1` and `en2`.
```cpp
//  StepperDriver4PWM( int ph1A,int ph1B,int ph2A,int ph2B, int en1 (optional), int en2 (optional))
//  - ph1A, ph1B - phase 1 pwm pins
//  - ph2A, ph2B - phase 2 pwm pins
//  - en1, en2  - enable pins (optional input)
StepperDriver4PWM driver = StepperDriver4PWM(5, 6, 9, 10, 7,  8);
```

## Step 2.1 PWM Configuration
```cpp
// pwm frequency to be used [Hz]
// for atmega328 fixed to 32kHz
// esp32/stm32/teensy configurable
driver.pwm_frequency = 20000;
```
<blockquote class="warning">
⚠️ Arduino devices based on ATMega328 chips have fixed pwm frequency of 32kHz.
</blockquote>

Here is a list of different microcontrollers and their PWM frequency and resolution used with the  Arduino <span class="simple">Simple<span class="foc">FOC</span>library</span>.

MCU | default frequency | MAX frequency | PWM resolution | Center-aligned | Configurable freq
--- | --- | --- | --- | ---
Arduino UNO(Atmega328) | 32 kHz | 32 kHz | 8bit | yes | no
STM32 | 50kHz | 100kHz | 14bit | yes | yes
ESP32 | 40kHz | 100kHz | 10bit | yes | yes
Teensy | 50kHz | 100kHz | 8bit | yes | yes

All of these settings are defined in the `drivers/hardware_specific/x_mcu.cpp/h` of the library source. 


## Step 2.2 Voltages
Driver class is the one that handles setting the pwm duty cycles to the driver output pins and it is needs to know the DC power supply voltage it is plugged to.
Additionally driver class enables the user to set the absolute DC voltage limit the driver will be set to the output pins.  
```cpp
// power supply voltage [V]
driver.voltage_power_supply = 12;
// Max DC voltage allowed - default voltage_power_supply
driver.voltage_limit = 12;
```

<img src="extras/Images/stepper_limits.png" class="width60">

This parameter is used by the `StepperMotor` class as well. As shown on the figure above the once the voltage limit `driver.voltage_limit` is set, it will be communicated to the FOC algorithm in `StepperMotor` class and the phase voltages will be centered around the `driver.voltage_limit/2`.

Therefore this parameter is very important if there is concern of too high currents generated by the motor. In those cases this parameter can be used as a security feature. 

## Step 2.3 Initialisation
Once when all the necessary configuration parameters are set the driver function `init()` is called. This function uses the configuration parameters and configures all the necessary hardware and software for driver code execution.
```cpp
// driver init
driver.init();
```

This function is responsible for:
- determining and configuring the hardware timer for PWM generation
- verifying that all provided pins can be used to generate PWM
- configuring the PWM channels

If for some reason the driver configuration fails this function will return `0` if everything went well the function will return `1`. So we suggest you to check if the init function was executed successfully before continuing
```cpp
Serial.print("Driver init ");
// init driver
if (driver.init())  Serial.println("success!");
else{
  Serial.println("failed!");
  return;
}
```

## Step 3. Using encoder in real-time

BLDC driver class was developed to be used with the <span class="simple">Simple<span class="foc">FOC</span>library</span> and to provide the abstraction layer for FOC algorithm implemented in the `StepperMotor` class. But the `StepperDriver4PWM` class can used as a standalone class as well and once can choose to implement any other type of control algorithm using the BLDC driver.  

## FOC algorithm support
In the context of the FOC control all the driver usage is done internally by the motion control algorithm and all that is needed to enable is is just link the driver to the `StepperMotor` class.
```cpp
// linking the driver to the motor
motor.linkDriver(&driver)
```

## Standalone driver 
If you wish to use the BLDC driver as a standalone device and implement your-own logic around it this can be easily done. Here is an example code of a very simple standalone application.
```cpp
// Stepper driver standalone example
#include <SimpleFOC.h>

// Stepper driver instance
StepperDriver4PWM driver = StepperDriver4PWM(5, 6, 9,10, 7, 8);

void setup() {
  
  // pwm frequency to be used [Hz]
  driver.pwm_frequency = 20000;
  // power supply voltage [V]
  driver.voltage_power_supply = 12;
  // Max DC voltage allowed - default voltage_power_supply
  driver.voltage_limit = 12;
  
  // driver init
  driver.init();

  // enable driver
  driver.enable();

  _delay(1000);
}

void loop() {
    // setting pwm
    // phase 1: 3V, phase 2: 6V
    driver.setPwm(3,6);
}
```