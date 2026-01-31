---
name: embedded-cpp
description: Embedded C/C++ patterns for resource-constrained systems, real-time requirements, interrupt handling, and hardware abstraction layers.
---

# Embedded C/C++ Development Patterns

Comprehensive guide to embedded systems development with C/C++ for MCUs and resource-constrained environments.

## When to Activate

- Writing firmware for microcontrollers
- Developing real-time systems
- Working with bare-metal or RTOS
- Interfacing with hardware peripherals
- Optimizing for memory/power constraints

## Memory Management

### Static Allocation

```cpp
// Prefer static allocation over dynamic
class SensorManager {
public:
    static constexpr size_t MAX_SENSORS = 8;
    
private:
    Sensor sensors_[MAX_SENSORS];  // Fixed array
    size_t sensorCount_ = 0;
    
public:
    bool addSensor(const Sensor& sensor) {
        if (sensorCount_ >= MAX_SENSORS) return false;
        sensors_[sensorCount_++] = sensor;
        return true;
    }
};

// Static buffer pools
template<typename T, size_t N>
class StaticPool {
    std::array<T, N> pool_;
    std::array<bool, N> used_{};
    
public:
    T* allocate() {
        for (size_t i = 0; i < N; ++i) {
            if (!used_[i]) {
                used_[i] = true;
                return &pool_[i];
            }
        }
        return nullptr;  // Pool exhausted
    }
    
    void deallocate(T* ptr) {
        size_t index = ptr - pool_.data();
        if (index < N) used_[index] = false;
    }
};
```

### Stack Management

```cpp
// Know your stack limits
// In linker script: _Min_Stack_Size = 0x400; /* 1KB */

// Avoid large stack allocations
void badFunction() {
    uint8_t buffer[4096];  // May overflow stack!
}

// Use static or heap for large buffers
static uint8_t buffer[4096];  // In .bss section

// Check stack usage at compile time (GCC)
// -fstack-usage generates .su files

// Runtime stack monitoring
#define STACK_CANARY 0xDEADBEEF
extern uint32_t _estack;
void checkStackOverflow() {
    uint32_t* stackBase = (uint32_t*)(&_estack - STACK_SIZE);
    if (*stackBase != STACK_CANARY) {
        errorHandler();
    }
}
```

### Memory Sections

```cpp
// Place data in specific memory regions
__attribute__((section(".ccmram")))
static uint32_t fastBuffer[256];  // Core-Coupled Memory

__attribute__((section(".noinit")))
static uint32_t persistentData;  // Survives soft reset

__attribute__((section(".dma_buffer")))
__attribute__((aligned(4)))
static uint8_t dmaBuffer[512];  // DMA-accessible memory

// Const data in Flash
static const uint8_t LOOKUP_TABLE[] __attribute__((section(".rodata"))) = {
    0, 1, 4, 9, 16, 25, 36, 49
};
```

## Interrupt Handling

### ISR Best Practices

```cpp
// Keep ISRs short and fast
volatile bool dataReady = false;
volatile uint8_t rxBuffer[64];
volatile size_t rxIndex = 0;

extern "C" void USART1_IRQHandler(void) {
    if (USART1->SR & USART_SR_RXNE) {
        uint8_t data = USART1->DR;
        if (rxIndex < sizeof(rxBuffer)) {
            rxBuffer[rxIndex++] = data;
        }
        if (data == '\n') {
            dataReady = true;
        }
    }
}

// Process in main loop
void mainLoop() {
    if (dataReady) {
        __disable_irq();
        // Copy volatile data to local buffer
        uint8_t localBuffer[64];
        size_t len = rxIndex;
        memcpy(localBuffer, (void*)rxBuffer, len);
        rxIndex = 0;
        dataReady = false;
        __enable_irq();
        
        processData(localBuffer, len);
    }
}
```

### Volatile Usage

```cpp
// Hardware registers must be volatile
volatile uint32_t* const GPIO_ODR = 
    reinterpret_cast<volatile uint32_t*>(0x40020014);

// Shared variables between ISR and main must be volatile
volatile bool flag = false;

// But volatile doesn't guarantee atomicity
volatile uint32_t counter;  // Not thread-safe for RMW

// Use atomic operations or disable interrupts
void incrementCounter() {
    __disable_irq();
    counter++;
    __enable_irq();
}

// Or use C11 atomics (if available)
#include <stdatomic.h>
atomic_uint counter = 0;
atomic_fetch_add(&counter, 1);  // Thread-safe
```

### Critical Sections

```cpp
// RAII-based critical section
class CriticalSection {
    uint32_t primask_;
public:
    CriticalSection() {
        primask_ = __get_PRIMASK();
        __disable_irq();
    }
    ~CriticalSection() {
        __set_PRIMASK(primask_);
    }
    
    // Non-copyable
    CriticalSection(const CriticalSection&) = delete;
    CriticalSection& operator=(const CriticalSection&) = delete;
};

// Usage
void sharedResourceAccess() {
    CriticalSection cs;
    // Access shared resource safely
    sharedData++;
}  // Interrupts restored automatically
```

## Hardware Abstraction Layer (HAL)

### Register Access Patterns

```cpp
// Type-safe register definition
struct GPIO_TypeDef {
    volatile uint32_t MODER;
    volatile uint32_t OTYPER;
    volatile uint32_t OSPEEDR;
    volatile uint32_t PUPDR;
    volatile uint32_t IDR;
    volatile uint32_t ODR;
    volatile uint32_t BSRR;
    volatile uint32_t LCKR;
    volatile uint32_t AFR[2];
};

#define GPIOA_BASE 0x40020000UL
#define GPIOA ((GPIO_TypeDef*)GPIOA_BASE)

// Bit manipulation helpers
constexpr uint32_t BIT(int n) { return 1U << n; }

template<uint32_t MASK, uint32_t POS>
constexpr uint32_t FIELD(uint32_t value) {
    return (value << POS) & MASK;
}

// Safe bit operations
inline void setBit(volatile uint32_t& reg, uint32_t bit) {
    reg |= bit;
}

inline void clearBit(volatile uint32_t& reg, uint32_t bit) {
    reg &= ~bit;
}

inline bool readBit(volatile uint32_t& reg, uint32_t bit) {
    return (reg & bit) != 0;
}
```

### GPIO Abstraction

```cpp
// Abstract GPIO pin
class Pin {
public:
    enum class Mode { Input, Output, Alternate, Analog };
    enum class Pull { None, Up, Down };
    
    Pin(GPIO_TypeDef* port, uint8_t pin) 
        : port_(port), pin_(pin), mask_(1U << pin) {}
    
    void setMode(Mode mode) {
        uint32_t moder = port_->MODER;
        moder &= ~(0x3U << (pin_ * 2));
        moder |= (static_cast<uint32_t>(mode) << (pin_ * 2));
        port_->MODER = moder;
    }
    
    void setPull(Pull pull) {
        uint32_t pupdr = port_->PUPDR;
        pupdr &= ~(0x3U << (pin_ * 2));
        pupdr |= (static_cast<uint32_t>(pull) << (pin_ * 2));
        port_->PUPDR = pupdr;
    }
    
    void set() { port_->BSRR = mask_; }
    void clear() { port_->BSRR = mask_ << 16; }
    void toggle() { port_->ODR ^= mask_; }
    bool read() const { return (port_->IDR & mask_) != 0; }
    
private:
    GPIO_TypeDef* port_;
    uint8_t pin_;
    uint32_t mask_;
};

// Usage
Pin led(GPIOA, 5);
led.setMode(Pin::Mode::Output);
led.set();
```

### Peripheral Interface Pattern

```cpp
// Abstract interface for drivers
class IUART {
public:
    virtual ~IUART() = default;
    virtual bool init(uint32_t baudRate) = 0;
    virtual bool send(const uint8_t* data, size_t len) = 0;
    virtual size_t receive(uint8_t* data, size_t maxLen) = 0;
    virtual bool available() const = 0;
};

// Concrete implementation
class STM32_UART : public IUART {
    USART_TypeDef* usart_;
    // ... ring buffers, state, etc.
    
public:
    explicit STM32_UART(USART_TypeDef* usart) : usart_(usart) {}
    
    bool init(uint32_t baudRate) override {
        // Configure USART peripheral
        usart_->BRR = SystemCoreClock / baudRate;
        usart_->CR1 = USART_CR1_UE | USART_CR1_TE | USART_CR1_RE;
        return true;
    }
    
    // ... other implementations
};
```

## Real-Time Patterns

### State Machine

```cpp
// Hierarchical state machine
class DeviceStateMachine {
public:
    enum class State { Idle, Initializing, Running, Error, Sleep };
    enum class Event { Start, Timeout, DataReceived, Error, Sleep, Wake };
    
private:
    State currentState_ = State::Idle;
    uint32_t stateEntryTime_ = 0;
    
public:
    void processEvent(Event event) {
        State nextState = currentState_;
        
        switch (currentState_) {
            case State::Idle:
                if (event == Event::Start) {
                    nextState = State::Initializing;
                } else if (event == Event::Sleep) {
                    nextState = State::Sleep;
                }
                break;
                
            case State::Initializing:
                if (event == Event::Timeout) {
                    if (initComplete()) {
                        nextState = State::Running;
                    } else {
                        nextState = State::Error;
                    }
                }
                break;
                
            case State::Running:
                if (event == Event::Error) {
                    nextState = State::Error;
                } else if (event == Event::DataReceived) {
                    handleData();
                }
                break;
                
            case State::Error:
                if (event == Event::Start) {
                    nextState = State::Initializing;
                }
                break;
                
            case State::Sleep:
                if (event == Event::Wake) {
                    nextState = State::Idle;
                }
                break;
        }
        
        if (nextState != currentState_) {
            exitState(currentState_);
            currentState_ = nextState;
            enterState(nextState);
            stateEntryTime_ = getTick();
        }
    }
    
private:
    void enterState(State state);
    void exitState(State state);
    bool initComplete();
    void handleData();
};
```

### Timeout Handling

```cpp
// Non-blocking timeout
class Timeout {
    uint32_t startTick_;
    uint32_t duration_;
    
public:
    Timeout(uint32_t durationMs) 
        : startTick_(HAL_GetTick()), duration_(durationMs) {}
    
    bool expired() const {
        return (HAL_GetTick() - startTick_) >= duration_;
    }
    
    void reset() {
        startTick_ = HAL_GetTick();
    }
    
    uint32_t elapsed() const {
        return HAL_GetTick() - startTick_;
    }
    
    uint32_t remaining() const {
        uint32_t e = elapsed();
        return (e >= duration_) ? 0 : (duration_ - e);
    }
};

// Usage
bool waitForResponse(uint32_t timeoutMs) {
    Timeout timeout(timeoutMs);
    while (!timeout.expired()) {
        if (responseReceived()) {
            return true;
        }
    }
    return false;  // Timeout
}
```

### Cooperative Scheduler

```cpp
// Simple round-robin scheduler
using TaskFunc = void(*)();

struct Task {
    TaskFunc func;
    uint32_t period;
    uint32_t lastRun;
    bool enabled;
};

class Scheduler {
    static constexpr size_t MAX_TASKS = 16;
    Task tasks_[MAX_TASKS];
    size_t taskCount_ = 0;
    
public:
    bool addTask(TaskFunc func, uint32_t periodMs) {
        if (taskCount_ >= MAX_TASKS) return false;
        tasks_[taskCount_++] = {func, periodMs, 0, true};
        return true;
    }
    
    void run() {
        while (true) {
            uint32_t now = HAL_GetTick();
            for (size_t i = 0; i < taskCount_; ++i) {
                Task& task = tasks_[i];
                if (task.enabled && (now - task.lastRun) >= task.period) {
                    task.func();
                    task.lastRun = now;
                }
            }
            // Optional: Enter low-power mode between tasks
            __WFI();
        }
    }
};

// Usage
void ledTask() { /* toggle LED */ }
void sensorTask() { /* read sensors */ }
void commTask() { /* handle communication */ }

Scheduler scheduler;
scheduler.addTask(ledTask, 500);     // Every 500ms
scheduler.addTask(sensorTask, 100);  // Every 100ms
scheduler.addTask(commTask, 10);     // Every 10ms
scheduler.run();
```

## Power Management

### Low-Power Modes

```cpp
class PowerManager {
public:
    enum class Mode { Run, Sleep, Stop, Standby };
    
    static void enterSleep() {
        // Configure wake sources
        __WFI();  // Wait For Interrupt
    }
    
    static void enterStop() {
        // Disable unused peripherals
        disableUnusedClocks();
        
        // Configure wake sources
        configureWakeupPin();
        
        // Enter STOP mode
        HAL_PWR_EnterSTOPMode(PWR_LOWPOWERREGULATOR_ON, 
                              PWR_STOPENTRY_WFI);
        
        // Reconfigure clocks after wake
        SystemClock_Config();
    }
    
    static void enterStandby() {
        // Save state to backup registers if needed
        RTC->BKP0R = state;
        
        // Enter standby (RAM lost)
        HAL_PWR_EnterSTANDBYMode();
        // Never returns - system resets on wake
    }
    
private:
    static void disableUnusedClocks() {
        // Disable peripheral clocks not needed during sleep
        __HAL_RCC_GPIOB_CLK_DISABLE();
        __HAL_RCC_GPIOC_CLK_DISABLE();
        // ... etc
    }
    
    static void configureWakeupPin() {
        HAL_PWR_EnableWakeUpPin(PWR_WAKEUP_PIN1);
    }
};
```

## Communication Protocols

### Ring Buffer

```cpp
template<typename T, size_t N>
class RingBuffer {
    T buffer_[N];
    volatile size_t head_ = 0;
    volatile size_t tail_ = 0;
    
public:
    bool push(T value) {
        size_t nextHead = (head_ + 1) % N;
        if (nextHead == tail_) return false;  // Full
        buffer_[head_] = value;
        head_ = nextHead;
        return true;
    }
    
    bool pop(T& value) {
        if (tail_ == head_) return false;  // Empty
        value = buffer_[tail_];
        tail_ = (tail_ + 1) % N;
        return true;
    }
    
    bool isEmpty() const { return head_ == tail_; }
    
    size_t size() const {
        if (head_ >= tail_) return head_ - tail_;
        return N - tail_ + head_;
    }
    
    size_t available() const { return N - 1 - size(); }
};

// ISR-safe usage
RingBuffer<uint8_t, 256> rxBuffer;

extern "C" void USART1_IRQHandler(void) {
    if (USART1->SR & USART_SR_RXNE) {
        rxBuffer.push(USART1->DR);
    }
}
```

### Protocol Parser

```cpp
// Simple packet protocol: [START][LEN][DATA...][CRC]
class PacketParser {
    enum class State { WaitStart, WaitLen, ReceiveData, WaitCrc };
    
    State state_ = State::WaitStart;
    uint8_t buffer_[64];
    size_t index_ = 0;
    uint8_t expectedLen_ = 0;
    
    static constexpr uint8_t START_BYTE = 0xAA;
    
public:
    bool processByte(uint8_t byte) {
        switch (state_) {
            case State::WaitStart:
                if (byte == START_BYTE) {
                    state_ = State::WaitLen;
                }
                break;
                
            case State::WaitLen:
                expectedLen_ = byte;
                if (expectedLen_ > sizeof(buffer_)) {
                    state_ = State::WaitStart;  // Invalid length
                } else {
                    index_ = 0;
                    state_ = State::ReceiveData;
                }
                break;
                
            case State::ReceiveData:
                buffer_[index_++] = byte;
                if (index_ >= expectedLen_) {
                    state_ = State::WaitCrc;
                }
                break;
                
            case State::WaitCrc:
                state_ = State::WaitStart;
                if (calculateCrc(buffer_, expectedLen_) == byte) {
                    return true;  // Packet complete and valid
                }
                break;
        }
        return false;
    }
    
    const uint8_t* getData() const { return buffer_; }
    size_t getLength() const { return expectedLen_; }
    
private:
    static uint8_t calculateCrc(const uint8_t* data, size_t len);
};
```

## Defensive Programming

### Watchdog

```cpp
class WatchdogManager {
public:
    static void init(uint32_t timeoutMs) {
        // Configure independent watchdog
        IWDG->KR = 0x5555;  // Enable register access
        IWDG->PR = 4;       // Prescaler /64
        IWDG->RLR = (timeoutMs * 32) / 1000;  // Reload value
        IWDG->KR = 0xAAAA;  // Reload
        IWDG->KR = 0xCCCC;  // Start watchdog
    }
    
    static void kick() {
        IWDG->KR = 0xAAAA;  // Reload counter
    }
};

// In main loop
void mainLoop() {
    while (true) {
        processTask1();
        processTask2();
        processTask3();
        WatchdogManager::kick();  // Must be called regularly
    }
}
```

### Assertions and Error Handling

```cpp
// Custom assert for embedded
#define ASSERT(expr) do { \
    if (!(expr)) { \
        assertFailed(__FILE__, __LINE__, #expr); \
    } \
} while(0)

void assertFailed(const char* file, int line, const char* expr) {
    __disable_irq();
    // Log error
    // Save to persistent memory for post-mortem
    // Enter safe state
    while (true) {
        // Blink error LED
    }
}

// Fault handlers
extern "C" void HardFault_Handler(void) {
    __disable_irq();
    // Capture fault information
    volatile uint32_t cfsr = SCB->CFSR;
    volatile uint32_t hfsr = SCB->HFSR;
    // Log and reset or halt
    NVIC_SystemReset();
}
```

## Best Practices Summary

| Practice | Why |
|----------|-----|
| Avoid dynamic allocation | Prevents fragmentation, deterministic |
| Use volatile for hardware/shared | Prevents compiler optimization issues |
| Keep ISRs short | Reduces interrupt latency |
| Use watchdog | Recovers from hangs |
| Static allocation | Predictable memory usage |
| Disable C++ exceptions/RTTI | Reduces code size |
| Check all return values | Catches errors early |

### Compiler Flags for Embedded

```makefile
# Size optimization
CFLAGS += -Os

# No exceptions/RTTI (C++)
CXXFLAGS += -fno-exceptions -fno-rtti

# Stack usage analysis
CFLAGS += -fstack-usage

# Warnings
CFLAGS += -Wall -Wextra -Wpedantic -Werror

# Dead code elimination
CFLAGS += -ffunction-sections -fdata-sections
LDFLAGS += -Wl,--gc-sections

# Link-time optimization
CFLAGS += -flto
LDFLAGS += -flto
```

**Remember**: In embedded systems, predictability and reliability are more important than flexibility. Every byte and cycle counts.
