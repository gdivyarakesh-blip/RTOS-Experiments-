# Experiment 7 — FreeRTOS Dual Tasks and Priority-Based Preemptive Scheduling

> Two tasks. Same delay interval. Different priority levels. Completely different observable outcomes.

---

## Objective

Configure two FreeRTOS tasks with adjustable priority levels and analyze how the **preemptive scheduler** allocates CPU time between them — verified through LED blink behavior and SWV ITM trace console output.

---

## Components Required

| Component | Qty |
|-----------|-----|
| STM32 Nucleo-F446RE | 1 |
| External LED | 1 |
| 220 Ohm Resistor | 1 |
| Breadboard + connecting wires | — |
| USB Type-A to Mini-B cable | 1 |

### External LED Connection
```
PA6 ---- [220 Ohm] ---- LED(+) ---- LED(-) ---- GND
```

---

## FreeRTOS Priority Scheduling — How It Works

### Equal Priority — Round-Robin Time Sharing
```
Time:    0ms      500ms    1000ms   1500ms
         |         |         |         |
Task_1:  ##........##........##........   LED1 blinks @ 1 Hz
Task_2:  ..##........##........##......   LED2 blinks @ 1 Hz

Both tasks yield via osDelay(500) -- CPU time shared equally.
```

### Unequal Priority — Preemption in Action
```
Time:    0ms      500ms    1000ms
         |         |         |
Task_1:  ####################   (HIGH priority -- first in CPU queue)
         (during its blocking period, Task_2 gets occasional access)
Task_2:  .....#.....#.....#.   LED2 blinks slower / starves
```

### Extreme Priority Gap — Task Starvation
```
Task_HIGH (Realtime): ################################
Task_LOW  (Low):      ................................ (may never run!)
```

---

## CubeMX Configuration

### Step 1 — GPIO Setup

| Pin | Label | Mode |
|-----|-------|------|
| PA5 | LED_1 (onboard LD2) | GPIO Output |
| PA6 | LED_2 (external) | GPIO Output |

### Step 2 — System Settings

| Setting | Value |
|---------|-------|
| RCC | BYPASS Clock Source |
| HCLK | 84 MHz |
| SYS → Debug | Trace Asynchronous Sw |
| SYS → Timebase | **TIM6** |

### Step 3 — FreeRTOS (CMSIS_V2)
- **Tasks & Queues** tab → create **two tasks**:

| Field | Task 1 | Task 2 |
|-------|--------|--------|
| Task Name | `LED_1` | `LED_2` |
| Entry Function | `Task1_function` | `StartLED_2` |
| Priority | (adjust per observation row) | (adjust per observation row) |
| Stack Size | 128 words | 128 words |

### Step 4 — Printf and SWV Setup (same as Exp 6)
- Enable `printf`/`scanf` float formatting in C/C++ Build Settings
- Debugger → SWV enabled, Core Clock = 84 MHz

---

## Source Code — `main.c`

### Header Include + ITM Output Redirect
```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
/* USER CODE END Includes */

/* USER CODE BEGIN 0 */
int _write(int file, char *ptr, int len) {
    for (int i = 0; i < len; i++) {
        ITM_SendChar(*ptr++);
    }
    return len;
}
/* USER CODE END 0 */
```

### Task Function Bodies
```c
/* Task 1 -- drives LED on PA5 */
void Task1_function(void *argument) {
    /* USER CODE BEGIN 5 */
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        printf("Task_1 Executing for LED Toggle \n");
        osDelay(500);
    }
    /* USER CODE END 5 */
}

/* Task 2 -- drives LED on PA6 */
void StartLED_2(void *argument) {
    /* USER CODE BEGIN StartLED_2 */
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_6);
        printf("Task_2 Executing for LED Toggle \n");
        osDelay(500);
    }
    /* USER CODE END StartLED_2 */
}
```

---

## Observation Table — Repeat for Each Priority Combination

Adjust priorities in the CubeMX Tasks & Queues tab, regenerate code, then record behavior:

| S.No | Task_1 Priority | Task_2 Priority | LED_1 Rate | LED_2 Rate | SWV Console Pattern | Remarks |
|------|----------------|----------------|-----------|-----------|---------------------|---------|
| 1 | `osPriorityLow` | `osPriorityLow` | | | | Equal share |
| 2 | `osPriorityNormal` | `osPriorityLow` | | | | Minor imbalance |
| 3 | `osPriorityHigh` | `osPriorityNormal` | | | | Noticeable difference |
| 4 | `osPriorityRealtime` | `osPriorityNormal` | | | | Risk of starvation |
| 5 | `osPriorityError` | `osPriorityLow` | | | | Extreme scenario |

### Expected SWV Console Patterns

```
Equal priorities:
  Task_1 Executing for LED Toggle
  Task_2 Executing for LED Toggle
  Task_1 Executing for LED Toggle
  Task_2 Executing for LED Toggle   <-- regular alternation

High vs Low:
  Task_1 Executing for LED Toggle
  Task_1 Executing for LED Toggle
  Task_1 Executing for LED Toggle
  Task_2 Executing for LED Toggle   <-- Task_2 appears infrequently
```

---

## Running the Program

```
1. Set task priorities in CubeMX --> Regenerate Code
2. Re-add task function bodies (lost after regeneration)
3. Connect board --> Build --> Debug
4. Window --> Show View --> SWV ITM Data Console
5. Enable Port 0 --> Start Trace --> Resume
6. Monitor LED blink rates and console message frequency
7. Repeat this procedure for every row in the observation table
```

---

## Suggested Modifications

| Experiment | How to Implement |
|-----------|-----------------|
| Make Task_2 dominate Task_1 | Assign Task_2 a higher priority level |
| Demonstrate complete starvation | Set Task_1 to `osPriorityRealtime`, Task_2 to `osPriorityLow` — Task_2 LED should halt |
| Introduce a third task | Add `LED_3` on a new GPIO pin with an intermediate priority |
| Remove `osDelay` from Task_1 | Task_1 will spin endlessly and completely starve Task_2 |

---

## Reflection Questions

1. How does the scheduler divide CPU time between two equal-priority tasks both using `osDelay(500)`?
2. What happens to the LEDs when one task contains no `osDelay()` call at all?
3. Compare this dual-task RTOS setup with a super loop controlling two LEDs — what structural advantages does the RTOS provide?
4. How does SWV ITM tracing confirm correct RTOS scheduling behavior without occupying GPIO pins or UART bandwidth?

---

## Result

Two FreeRTOS tasks were created with configurable priorities using the CMSIS-RTOS v2 interface. Priority-based preemptive scheduling was clearly demonstrated through differential LED blink rates and SWV ITM console output, illustrating how a higher-priority task monopolizes CPU access and can lead to starvation of lower-priority tasks.
