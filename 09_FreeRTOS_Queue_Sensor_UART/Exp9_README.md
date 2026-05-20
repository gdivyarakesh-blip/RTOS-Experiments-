# Experiment 9 — Inter-Task Data Transfer via FreeRTOS Message Queue

> Semaphores announce "an event occurred."
> Queues announce "an event occurred — and here is the associated data."

---

## Objective

Build a **Producer-Consumer** inter-task communication pipeline using a FreeRTOS **Message Queue**, where a `Sensor_Read` task produces distance measurements every second and a `Motion_Control` task consumes and processes them.

---

## Components Required

| Component | Qty |
|-----------|-----|
| STM32 Nucleo-F446RE | 1 |
| USB Type-A to Mini-B cable | 1 |
| (Real ultrasonic sensor optional — experiment uses simulated data) | — |

---

## The Producer-Consumer Data Pipeline

```
+------------------+          +---------------------+          +----------------------+
|                  |          |                     |          |                      |
|  SENSOR_READ     |--PUT-->  |   FreeRTOS QUEUE    |--GET-->  |  MOTION_CONTROL      |
|  (Producer)      |          |   [16 slots]        |          |  (Consumer)          |
|                  |          |   +-+-+-+-+-+-+     |          |                      |
|  dist += 1       |          |   |1|2|3|4| | |     |          |  receives distance   |
|  every 1000 ms   |          |   +-+-+-+-+-+-+     |          |  prints via ITM      |
|                  |          |   FIFO ordering     |          |  blocks when empty   |
+------------------+          +---------------------+          +----------------------+
        |                              |                                  |
    osDelay(1000)              Copy-by-Value:                    osMessageQueueGet()
    yields for 1 s             data duplicated INTO              blocks until data
                               queue's internal buffer           becomes available
```

---

## Why Queues Instead of Global Variables?

```c
// Global variable (unsafe in multi-task environment)
unsigned int dist = 0;  // Task A writes, Task B reads -- RACE CONDITION!
                        // Scheduler may context-switch mid-access

// FreeRTOS Queue (thread-safe)
osMessageQueuePut(queue, &dist, ...);  // Kernel-managed, atomic transfer
osMessageQueueGet(queue, &dist, ...);  // Blocks gracefully when empty
```

### Queue Blocking Behavior

```
Queue EMPTY --> Consumer blocks (zero CPU burn) until Producer deposits data
                   Consumer: BLOCKED ----------- READY --> RUNNING
                                       data arrives ^

Queue FULL  --> Producer blocks until Consumer reads and frees a slot
              (natural flow control -- prevents data loss)
```

---

## CubeMX Configuration

### Step 1 — System Settings

| Setting | Value |
|---------|-------|
| RCC | BYPASS |
| HCLK | 84 MHz |
| SYS → Debug | Trace Asynchronous Sw |
| SYS → Timebase | **TIM6** |

### Step 2 — FreeRTOS (CMSIS_V2)

**Tasks & Queues tab — Tasks:**

| Field | Task 1 | Task 2 |
|-------|--------|--------|
| Task Name | `Sensing_Task` | `Navigation_Task` |
| Entry Function | `Sensor_Read` | `Motion_Control` |
| Priority | `osPriorityNormal` | `osPriorityNormal` |

**Tasks & Queues tab — Queue:**

| Field | Value |
|-------|-------|
| Queue Name | `myQueue01` |
| Queue Size | `16` (number of slots) |
| Item Size | `sizeof(unsigned int)` = 4 bytes |

### Step 3 — Advanced Settings
- Enable **USE_NEWLIB_REENTRANT** (required for `printf` across multiple tasks)

### Step 4 — Printf and SWV Setup
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

### Confirm Queue Item Type
```c
/* Verify element type matches unsigned int -- update if different */
myQueue01Handle = osMessageQueueNew(16, sizeof(unsigned int), &myQueue01_attributes);
```

### Producer Task — `Sensor_Read`
```c
void Sensor_Read(void *argument) {
    /* USER CODE BEGIN 5 */
    unsigned int dist = 0;
    for (;;) {
        printf("Inside Data Producer Task\n");

        dist = dist + 1;  // Simulated distance reading (replace with real sensor code)

        /* Deposit value into the queue -- blocks if queue is full */
        osMessageQueuePut(myQueue01Handle, &dist, 0, osWaitForever);

        osDelay(1000);  // Generate a new reading every 1 second
    }
    /* USER CODE END 5 */
}
```

### Consumer Task — `Motion_Control`
```c
void Motion_Control(void *argument) {
    /* USER CODE BEGIN Motion_Control */
    unsigned int distance;
    for (;;) {
        printf("Inside Data Consumer Task\n");

        /* Suspend here until a value appears in the queue */
        osMessageQueueGet(myQueue01Handle, &distance, NULL, osWaitForever);

        printf("Distance is %u\n", distance);
        /* Insert downstream processing here: motor logic, display output, etc. */
    }
    /* USER CODE END Motion_Control */
}
```

---

## Running and Observing

```
1. Build --> Debug --> Switch to debug perspective
2. Window --> Show View --> SWV ITM Data Console
3. Port 0 --> Start Trace --> Resume

Expected SWV output:
   Inside Data Producer Task
   Inside Data Consumer Task
   Distance is 1
   Inside Data Consumer Task       <-- consumer blocks, waits
   Inside Data Producer Task       <-- 1 second later
   Inside Data Consumer Task
   Distance is 2
   ...
```

---

## Observation Table

| # | Query | Response |
|---|-------|----------|
| 1 | Priority of `Sensor_Read` task | |
| 1 | Priority of `Motion_Control` task | |
| 2 | Which task has higher priority? | Both equal / Sensor / Motion |
| 3 | Which task appears more frequently in SWV? | Sensor / Motion / Both alternate |
| 4 | Does the consumer receive every value produced? | Yes / No |

---

## Suggested Modifications

### Swap Simulated Data for Real HC-SR04 Readings
Place the ultrasonic distance computation code from Experiment 3 inside `Sensor_Read`, replacing the `dist = dist + 1` line with actual measured distance before the `osMessageQueuePut()` call.

### Trigger Queue-Full Behavior
Reduce queue size to `3`, retain `osDelay(1000)` in the producer, but remove the delay from the consumer — observe the producer stall when all slots are occupied.

### Trigger Queue-Empty Behavior
Add `osDelay(2000)` in the consumer and keep `osDelay(1000)` in the producer — observe the consumer block for 2 seconds at a time while waiting for new data.

---

## Reflection Questions

1. What does "thread-safe" mean and why does a FreeRTOS queue guarantee it while a global variable does not?
2. Implement and test a **fast producer, slow consumer** — what happens when the queue fills up?
3. Implement and test a **slow producer, fast consumer** — what does the consumer do while waiting for data?
4. Why is `osWaitForever` used as the timeout — what would using `0` (no wait) change about the behavior?

---

## Result

Inter-task communication was successfully demonstrated using a FreeRTOS Message Queue. The `Sensor_Read` producer task placed data into the kernel-managed queue every second, while `Motion_Control` remained efficiently suspended until new data arrived — producing a clean, race-condition-free Producer-Consumer pipeline.
