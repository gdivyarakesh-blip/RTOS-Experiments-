# STM32F446RE — Real-Time Embedded Systems and Peripheral Interfacing

This repository contains a collection of practical embedded systems projects developed using the STM32F446RE Nucleo Board. The projects begin with low-level GPIO control and gradually advance toward multitasking firmware applications using FreeRTOS, STM32CubeIDE, and STM32CubeMX.

Each directory demonstrates a standalone implementation focused on a particular peripheral interface, firmware concept, or RTOS mechanism on the STM32 platform.

---

## Key Learning Areas

- GPIO Output Configuration and LED Blinking
- Push Button Interfacing using Digital Inputs
- Ultrasonic Distance Measurement using HC-SR04
- PWM-Based Brightness and Signal Control
- Cooperative Super Loop Scheduling
- FreeRTOS Task Initialization and Scheduling
- Priority-Based Task Management in FreeRTOS
- Interrupt Handling with RTOS Synchronization
- Queue-Based Communication Between Tasks
- Shared Resource Protection using Counting Semaphores

---

## Repository Layout

| Folder | Experiment | Description |
|--------|------------|-------------|
| `01_GPIO_LED_Blink_SoftwareDelay` | LED Blinking using GPIO | Configure GPIO pins as outputs and implement LED blinking with software-generated delays |
| `02_PushButton_LED_Toggle` | Push Button Controlled LED | Read push button input states and toggle the onboard LED whenever the button is pressed |
| `03_HCSR04_Ultrasonic_LED_RangeIndicator` | Ultrasonic Sensor Interfacing | Measure object distance using the HC-SR04 ultrasonic module and indicate ranges using LEDs and UART |
| `04_PWM_LED_BrightnessControl` | PWM Signal Generation | Generate PWM signals through timers and vary LED brightness dynamically |
| `05_SuperLoop_LED_Button_Sensor` | Non-Blocking Super Loop Design | Create a super loop structure capable of handling LEDs, buttons, and sensors simultaneously without blocking execution |
| `06_FreeRTOS_SingleTask_LED_Blink` | FreeRTOS Single Task Example | Develop a basic FreeRTOS application with a single task performing periodic LED blinking |
| `07_FreeRTOS_DualTask_Priority_LED` | Task Priority Scheduling | Analyze task execution by assigning different priorities to multiple LED blinking tasks |
| `08_FreeRTOS_EXTI_Semaphore_LED` | External Interrupt with Semaphore | Trigger LED actions using external interrupts synchronized with FreeRTOS binary semaphores |
| `09_FreeRTOS_Queue_Sensor_UART` | Queue-Based Inter-Task Communication | Transfer sensor information between tasks using FreeRTOS queues and display results via UART |
| `10_FreeRTOS_CountingSemaphore_SharedResource` | Counting Semaphore Resource Sharing | Demonstrate concurrent resource access management using counting semaphores across multiple tasks |

---

## Tools and Technologies

| Component | Details |
|-----------|---------|
| **Microcontroller Board** | STM32F446RE Nucleo |
| **IDE** | STM32CubeIDE |
| **RTOS Framework** | FreeRTOS |
| **Programming Language** | Embedded C |
| **Programming & Debugging** | STM32CubeProgrammer / ST-Link Debugger |

---

## Project Flow

```text
GPIO Basics --> Button Interfacing --> Ultrasonic Sensor --> PWM Generation
        |
   Super Loop Architecture
        |
FreeRTOS Single Task --> Multi-Task Scheduling --> Interrupt Synchronization
        |
 Queue Communication --> Counting Semaphore Resource Control
