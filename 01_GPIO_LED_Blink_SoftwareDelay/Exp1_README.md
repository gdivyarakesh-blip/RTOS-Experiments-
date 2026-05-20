# Experiment 1 — GPIO Digital Output: LED Blink Using Software Delay

## Objective
Configure a GPIO pin on the **STM32F446RE** microcontroller as a digital output and confirm LED blinking behavior using software-based delay routines.

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

The **STM32F446RE** microcontroller provides multiple general-purpose I/O (GPIO) ports — **GPIOA through GPIOH** — each of which can be individually assigned to one of four operating modes:

| Mode | Description |
|------|-------------|
| **Input** | Reads an incoming digital signal |
| **Output** | Drives a digital HIGH or LOW signal |
| **Alternate Function** | Delegated to on-chip peripherals (UART, SPI, I2C, etc.) |
| **Analog** | Connected to internal ADC/DAC circuits |

### Why Pin PA5?
On the **Nucleo-F446RE** development board, the green user LED (**LD2**) is physically wired to **Arduino header pin D13**, which maps to microcontroller pin **PA5**. No external LED or additional wiring is necessary — configuring PA5 as an output directly controls LD2.

### Push-Pull Output Mode
PA5 is configured in **push-pull output** mode. In this arrangement, the driver circuit can:
- Actively drive the pin **HIGH** (~3.3 V) — LED turns **ON**
- Actively drive the pin **LOW** (0 V) — LED turns **OFF**

This differs from open-drain mode, which can only pull the signal low and relies on an external resistor to bring it high.

### HAL Functions Utilized

| Function | Purpose |
|----------|---------|
| `HAL_GPIO_TogglePin(GPIOx, GPIO_Pin)` | Inverts the current logic state of the specified GPIO pin |
| `HAL_Delay(ms)` | Halts execution for the given number of milliseconds using the SysTick timer |

---

## STM32CubeMX Setup

### Step 1 — MCU Selection
- Launch STM32CubeMX → **ACCESS TO MCU SELECTOR**
- Search for and select: **STM32F446RETx**
- Click **Start Project**

### Step 2 — GPIO Configuration
- Navigate to **System Core → GPIO**
- Right-click pin **PA5** → Select **GPIO_Output**
- Apply the following GPIO settings:

| Parameter | Value |
|-----------|-------|
| GPIO Mode | Output Push Pull |
| GPIO Pull-up/Pull-down | No pull-up and no pull-down |
| Maximum Output Speed | Low |
| User Label | LD2 (optional) |

### Step 3 — Clock Source (RCC)
- Go to **System Core → RCC**
- High Speed Clock (HSE): **BYPASS Clock Source**
- This enables the board to use the external clock signal provided by the ST-Link interface

### Step 4 — Clock Configuration

| Clock Domain | Prescaler | Frequency |
|--------------|-----------|-----------|
| PLL Source | HSE | — |
| SYSCLK | PLL multiplier | 180 MHz |
| AHB (HCLK) | /1 | 180 MHz |
| APB1 | /4 | 45 MHz |
| APB2 | /2 | 90 MHz |

### Step 5 — Project Manager
- Provide a **Project Name** (e.g., `Experiment_1`)
- Toolchain/IDE: **STM32CubeIDE**
- Click **Generate Code** → **Open Project**

### Step 6 — Post-Build Output Settings
- Right-click the project → **Properties → C/C++ Build → Settings → MCU Post Build Outputs**
- Check **Convert to Binary File (.bin)**
- Check **Convert to Intel Hex File (.hex)**
- Click **Apply and Close**

---

## Source Code

After code generation, open `Core/Src/main.c` and insert the following lines inside the `while(1)` loop between the designated user code markers:

```c
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);   // Flip LED LD2 state on PA5
    HAL_Delay(500);                           // Hold for 500 ms

    /* USER CODE END WHILE */
}
```

> Always place your code between `USER CODE BEGIN` and `USER CODE END` markers. Any code written outside these blocks will be **removed** during CubeMX code regeneration.

---

## Build and Run

1. Connect the Nucleo board via USB
2. Click **Build** (hammer icon) → verify **0 errors, 0 warnings**
3. Click **Debug** → click **Switch** when the perspective-switch dialog appears
4. Click **Resume (▶)** to begin execution
5. The onboard LED LD2 should begin blinking at the configured rate

---

## Recorded Observations

The delay value was varied across multiple trials to observe its effect on blink speed:

| S. No. | GPIO Pin | Mode / Pull | Delay (ms) | Approx. Blink Period (s) | Comment |
|--------|----------|-------------|------------|--------------------------|---------|
| 1 | PA5 | Output, PP, No PU/PD | 100 | 0.1 s | Very rapid — barely noticeable |
| 2 | PA5 | Output, PP, No PU/PD | 200 | 0.3 s | Moderate and clearly visible |
| 3 | PA5 | Output, PP, No PU/PD | 500 | 1 s | Steady, easily observable |
| 4 | PA5 | Output, PP, No PU/PD | 1000 | 2 s | Slow — ON and OFF phases are well-separated |

> **Note:** The total blink period is roughly 2 × the delay value, since both the ON and OFF phases each consume one complete delay interval.

---

## Result

GPIO pin **PA5** on the STM32F446RE Nucleo board was successfully configured as a **push-pull digital output** through STM32CubeIDE. The onboard user LED **LD2** blinked correctly at every programmed rate. Changing the delay value produced a proportional and clearly visible shift in blink frequency, confirming correct GPIO output behavior and system clock configuration.

---

## Key Takeaways

- GPIO pins are multi-functional — the same physical pin may act as input, output, alternate function, or analog, determined entirely through software configuration.
- **Push-pull mode** actively manages both the high and low states of the pin, making it well-suited for directly driving an LED without additional components.
- `HAL_Delay()` is a **blocking call** driven by the SysTick timer. While it waits, the CPU is completely halted and cannot perform any other task — this is a fundamental constraint of bare-metal super loop programs.
- The **accuracy of `HAL_Delay()`** depends on a correctly configured SYSCLK. An incorrect clock setup will yield inaccurate delays.
- The **super loop (`while(1)`)** is the most elementary program structure in embedded software — linear, single-task, and running without interruption.
- Blink period = 2 × delay, because one complete ON-OFF cycle requires two delay durations.

---

## Project Layout

```
01_GPIO_LED_Blink_SoftwareDelay/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          <- User code resides here (while loop)
├── Drivers/
│   └── STM32F4xx_HAL_Driver/
├── Experiment_1.ioc        <- CubeMX pin and clock settings file
└── README.md               <- This document
```
