# Experiment 2 — GPIO Digital Input: Push Button Controlled LED Toggle

## Objective
Interface a push button as a digital input device and demonstrate LED state control by toggling it on every valid button press.

---

## Components Required

| Component | Details |
|-----------|---------|
| Microcontroller Board | STM32 Nucleo-F446RE |
| Cable | USB Type-A to Mini-B |
| IDE | STM32CubeIDE |
| Configuration Tool | STM32CubeMX (integrated into CubeIDE) |

---

## Background Theory

This experiment extends the GPIO output concepts from Experiment 1 and introduces **GPIO input**, where the onboard user button is used to control the onboard LED.

### GPIO Configured as Input
When a GPIO pin is placed in **digital input** mode, the microcontroller passively observes the voltage on that pin and interprets it as logic `1` (HIGH) or `0` (LOW). Unlike output mode, the MCU does not drive the pin — it simply reads it.

### Board Pin Assignments

| Function | Arduino Label | MCU Pin | Default Logic Level |
|----------|---------------|---------|---------------------|
| User LED (LD2) | D13 | **PA5** | Output, Push-Pull |
| User Button (B1) | — | **PC13** | Input, Active LOW |

### Understanding Active-Low Button Logic
The onboard user button **B1** on the Nucleo-F446RE is connected to **PC13**, which has a hardware pull-up resistor already installed on the board. This results in:
- Button **not pressed** → PC13 reads **HIGH (1)**
- Button **pressed** → PC13 reads **LOW (0)**

This convention is referred to as **active-low** — the signal transitions low when the intended event (button press) takes place.

### What is Contact Bounce?
Mechanical push buttons do not produce a single clean transition. During a press, the metal contacts physically bounce several times before settling, generating a rapid burst of HIGH/LOW transitions. This is called **contact bounce**, and it can cause the MCU to register a single press as multiple events.

A simple **software debounce** is applied here — after a press is detected, a short `HAL_Delay(200)` pause allows the signal to stabilize before the next read.

### HAL Functions Utilized

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_ReadPin(GPIOx, GPIO_Pin)` | Returns the current logic level of an input pin (`0` or `1`) |
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current state of a GPIO output pin |
| `HAL_Delay(ms)` | Blocking millisecond pause — used here for software debounce |

---

## STM32CubeMX Setup

### Step 1 — MCU Selection
- Open STM32CubeMX → **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration

**PA5 — LED Output:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LED (optional) |

**PC13 — Push Button Input:**

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Input mode |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| User Label | BTN (optional) |

> The Nucleo board already provides a hardware pull-up on PC13, so no internal pull-up needs to be configured in CubeMX.

### Step 3 — Clock Source (RCC)
- Go to **System Core → RCC**
- High Speed Clock (HSE): **BYPASS Clock Source**

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL output | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Enter a **Project Name** (e.g., `Experiment_2`)
- Toolchain/IDE: **STM32CubeIDE**
- Click **Generate Code** → **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project → **Properties → C/C++ Build → Settings → MCU Post Build Outputs**
- Check **Convert to Binary File (.bin)**
- Check **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

## Source Code

Open `Core/Src/main.c` and insert the following code inside the `while(1)` loop between the user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    // Read PC13 — Active LOW: 0 means button is pressed
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == 0)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Flip LED state on PA5
        HAL_Delay(200);                           // Debounce pause
    }
    /* USER CODE END WHILE */
}
```

### Program Flow

```
Loop begins
    |
    v
Sample PC13
    |
    |-- HIGH (button released) --> do nothing --> loop again
    |
    +-- LOW (button held)
            |
            v
        Toggle PA5 (LED flips)
            |
            v
        Wait 200 ms  <- settles contact bounce
            |
            v
        Loop again
```

> Keep all user code strictly between the `USER CODE BEGIN` and `USER CODE END` markers to avoid it being overwritten on CubeMX regeneration.

---

## Build and Run

1. Connect the Nucleo board via USB
2. Click **Build** (hammer icon) → verify **0 errors**
3. Click **Debug** → click **Switch** on the perspective dialog
4. Click **Resume (▶)** to begin execution
5. Press the blue **B1 user button** on the board — the green LED **LD2** should toggle with each press

---

## Recorded Observations

### Button Press vs. LED State

| Trial No. | Button Action | PC13 State | LED State Before | LED State After | Remark |
|-----------|---------------|------------|------------------|-----------------|--------|
| 1 | No press (initial) | HIGH | OFF | OFF | No change |
| 2 | 1st Press | LOW | OFF | ON | LED turns ON |
| 3 | Release | HIGH | ON | ON | LED stays ON |
| 4 | 2nd Press | LOW | ON | OFF | LED turns OFF |
| 5 | 3rd Press | LOW | OFF | ON | LED turns ON again |
| 6 | Rapid Press | Bouncing | Varies | Toggles | LED flickers at high speed |

**Key observations:**
- The LED state is **latched** — it holds the new value even after the button is released.
- Every confirmed press triggers exactly one toggle.
- Rapid successive presses can cause flickering since the 200 ms window may not always be sufficient.

---

## Result

For every valid button press, the LED on PA5 toggled precisely once and retained its state after release. This confirms correct configuration of **PC13 as a digital input** and **PA5 as a digital output**, along with a functional software debounce implemented through `HAL_Delay()`.

---

## Key Takeaways

- A microcontroller has no inherent understanding of a "button press" as an event — it only reads binary logic levels at a moment in time. The programmer defines how to interpret signal transitions.
- **Active-low logic** is standard practice in embedded hardware design. Always refer to the board schematic to determine the idle-state polarity of an input signal.
- **Contact bounce** is a physical reality of mechanical switches. A single press can generate dozens of spurious transitions within microseconds. Delay-based debounce is the simplest approach, although state-machine or timer-based methods are generally more robust.
- The LED in this experiment is **stateful** — unlike the periodic blink in Experiment 1, it retains and holds its last condition. This forms the basis for latches and toggles in practical embedded control systems.
- Combining GPIO input and output in the same program creates a meaningful stimulus-response loop — the foundational building block of all embedded control systems.

---

## Project Layout

```
02_PushButton_LED_Toggle/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          <- User code resides here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_2.ioc        <- CubeMX pin and clock settings file
└── README.md               <- This document
```
