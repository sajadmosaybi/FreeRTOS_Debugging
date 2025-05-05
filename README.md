# 🔧 1. Traditional Debugging with IDE (e.g., STM32CubeIDE)
- Step into/step over tasks and ISRs.
- Breakpoints: Set breakpoints in tasks or ISRs.
- Watch variables: Monitor global or local variables.
- Call stack: View which functions/tasks are running.
- Memory view: Inspect heap, stack, and other memory.
#### 💡 STM32CubeIDE has FreeRTOS awareness, showing task list, states, stack usage, etc., in real time if enabled properly.

No code changes needed—just:

- Build and run in Debug mode.
- Set breakpoints in task functions.
- Use the “FreeRTOS Task List” tab during debug.
```
void StartTask1(void *argument) {
    for (;;) {
        int x = 123;  // <--- Set a breakpoint here
        osDelay(1000);
    }
}
```
# 🧠 2. FreeRTOS+Trace Tools
- Percepio Tracealyzer: Very powerful graphical debugger for FreeRTOS. It gives insight into:
- Task execution timeline
- Interrupt latency
- CPU usage
- Task interactions
- Requires adding trace macros in your FreeRTOSConfig.h and using vTraceEnable().

Percepio Tracealyzer is a real-time visualization tool for FreeRTOS and other RTOS-based applications. It shows what's happening inside your firmware: which tasks are running, when context switches occur, CPU load, task timing, and more. It is invaluable for debugging complex timing issues, unexpected behavior, or performance optimization.

## 🧠 What You Can Do with Tracealyzer

- Visualize task execution, state changes, and runtime.
- Analyze CPU usage and identify bottlenecks.
- Measure response times and interrupt latencies.
- Debug priority inversion, missed deadlines, or task starvation.
- Export execution data and logs.

## ✅ What You Need

- STM32 development board (e.g., STM32F4, STM32H7)
- STM32CubeIDE (or STM32CubeMX + IDE of your choice)
- Percepio Tracealyzer (Download from [Percepio.com](Percepio.com))
- Percepio TraceRecorder library (you’ll add it to your project)
- Optional: Serial (UART) or RTT for data streaming

### 📁 Step 1: Create a FreeRTOS Project

- Open STM32CubeIDE.
- Create new STM32 project (choose your MCU or board).
- In CubeMX:
- Enable FreeRTOS middleware.
- Create a few dummy tasks.
- Generate the project code.

### 📦 Step 2: Download & Add TraceRecorder
- Go to: https://percepio.com/tracealyzer/
- Download:
- Tracealyzer
- TraceRecorder source code
- Unzip TraceRecorder and copy the trace_recorder folder into your project (Core or Middlewares).
- In your project, add this path to includes:
  ```
  Core/trace_recorder/include
  ```
### ⚙️ Step 3: Edit FreeRTOSConfig.h
  
In FreeRTOSConfig.h, enable tracing macros:
```
#define configUSE_TRACE_FACILITY        1
#define configUSE_STATS_FORMATTING_FUNCTIONS 1
#define configGENERATE_RUN_TIME_STATS   1

#include "trcRecorder.h"   // Add this at the bottom

```
Also define these macros to redirect trace calls:
```
#define vPortDefines.h // Leave as is
#define INCLUDE_xTaskGetIdleTaskHandle  1

```
### 🧠 Step 4: Start Trace Recording
In main.c before osKernelStart():
```
vTraceEnable(TRC_START);  // Start recording
// or use TRC_INIT for delayed start

```
### 🧵 Step 5: Add Tasks (Example)
```
void StartTask1(void *argument) {
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        osDelay(1000);
    }
}

```
### 🧪 Step 6: Choose Your Trace Mode

Trace data can be:

- Stored in RAM (snapshot)
- Streamed via UART or RTT (recommended)
- To enable snapshot mode:
- In trcConfig.h (inside trace_recorder/include), set:
```
#define TRC_CFG_RECORDER_MODE TRC_RECORDER_MODE_SNAPSHOT
```
For streaming mode, set TRC_RECORDER_MODE_STREAMING and configure the port (e.g., UART or SEGGER RTT).

### 💻 Step 7: Build, Flash, and Run

- Build the project in STM32CubeIDE.
- Flash it to the board.
- The firmware now records trace data.

### 🧲 Step 8: View Trace in Tracealyzer

- Open Percepio Tracealyzer.
  
 Select:
- New Project > Import Snapshot.
- Point to memory dump (.bin) or serial stream.

Visualize:
- Task timeline
-CPU load
-Task-to-task communication
-Interrupts
-Errors or missed deadlines

### 🎯 OPTIONAL: Use RTT Streaming (Recommended for Real-Time)

If using SEGGER RTT for real-time streaming:

- Add SEGGER_RTT.c/h to your project.
- Set TRC_CFG_RECORDER_MODE to TRC_RECORDER_MODE_STREAMING.
- Set TRC_CFG_STREAM_PORT to TRC_STREAM_PORT_RTT.

# 📋 3. Serial Printf Debugging
- Print messages via UART using printf() or HAL_UART_Transmit().
- Use it in key parts of your code to trace task behavior.
- Less precise, but useful when you can't use a debugger.
  
⚠️ Avoid using printf() directly in tasks without care—it can block or affect real-time behavior. Use it wisely or redirect to a thread-safe queue.

```
void StartTask2(void *argument) {
    for (;;) {
        printf("Task2 running\r\n");
        HAL_UART_Transmit(&huart2, (uint8_t*)"Task2 OK\n", 9, HAL_MAX_DELAY);
        osDelay(1000);
    }
}
```
# 🔍 4. FreeRTOS Kernel Aware Debugging in STM32CubeIDE
- When FreeRTOS is integrated, STM32CubeIDE shows:
  - Task list
  - Stack usage per task
  - State (Ready, Running, Blocked)
  - Priority
- Make sure to enable the FreeRTOS Debug plug-in.

No code needed.
- Enable USE_OS and select CMSIS_V1 or V2.
- While debugging, check the "FreeRTOS" tab (shows task states, priorities, etc.).

# 🧰 5. SWV (Serial Wire Viewer) / ITM Debugging
- Use SWO pin and ITM printf() for fast debugging over a single pin.
- Allows logging without UART interference.
- Useful for timestamped logs.

Enable ITM/Trace in STM32CubeMX:
```
// Use ITM_SendChar to send data
ITM_SendChar('A');
```
Then configure SWV in STM32CubeIDE:

- Enable ITM in debug configuration.
- Use the SWV Console to view output.

# 📦 6. Monitoring Stack Overflow and Heap Usage

In FreeRTOSConfig.h:
```
#define configCHECK_FOR_STACK_OVERFLOW    2
#define configUSE_MALLOC_FAILED_HOOK      1

```
In main.c:
```
void vApplicationStackOverflowHook(TaskHandle_t xTask, char *pcTaskName) {
    printf("Stack overflow in %s\r\n", pcTaskName);
    while (1);
}

void vApplicationMallocFailedHook(void) {
    printf("Malloc failed!\r\n");
    while (1);
}

```
# 🧪 7. Task-Specific Debugging Aids

Monitor task states manually:
```
eTaskState state = eTaskGetState(TaskHandle_t task);
```
- Use uxTaskGetStackHighWaterMark() to check remaining stack.
  
```
eTaskState state = eTaskGetState(myTaskHandle);
printf("State: %d\r\n", state);

UBaseType_t highWaterMark = uxTaskGetStackHighWaterMark(NULL);
printf("Stack high water mark: %lu\r\n", highWaterMark);
```
# 📷 8. Oscilloscope or Logic Analyzer
- Toggle GPIO pins at key points in tasks or ISRs.
- Observe task switching or timing externally.
```
void StartTask3(void *argument) {
    for (;;) {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5); // Toggle LED
        osDelay(500);
    }
}
```
Connect oscilloscope or logic analyzer to the pin.

# 🧬 9. Unit Testing FreeRTOS Tasks (Advanced)
- Use mocks or hardware abstraction to test task logic.
- Run parts of your code on host PC in simulation (e.g., with Ceedling or Unity).

You can simulate task logic without FreeRTOS:
```
int compute(int a) {
    return a * 2;
}

// Unit Test
void test_compute() {
    assert(compute(3) == 6);
}
```
# 🗂️ 10. Watchdog Integration
- Use the Independent Watchdog (IWDG) to detect system hangs.
- Refresh watchdog in a known-good task to catch deadlocks or crashes.
```
// In one reliable task
void StartWatchdogTask(void *argument) {
    for (;;) {
        HAL_IWDG_Refresh(&hiwdg);  // Refresh watchdog
        osDelay(500);
    }
}
```
