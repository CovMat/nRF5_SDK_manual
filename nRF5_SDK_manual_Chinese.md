# 需要手动添加的头文件位置（include PATH）
下面的头文件位置以nRF52840芯片，S140协议栈，SEGGER Embedded Studio开发环境为例。不同的芯片与协议栈要进行对应的修改。  
首先在SEGGER的project->option->Common->Code->Build->Project Macros中，添加宏定义，设置SDK文件夹的位置：  
```
SDK_PATH=C:\Users\chenxh\Downloads\nRF5_SDK_16.0.0_98a08e2
```
然后在project->option->Common->Code->Preprocessor->User Include Directories中，添加如下头文件include位置：
```
$(SDK_PATH)\modules\nrfx\hal
$(SDK_PATH)\modules\nrfx
$(SDK_PATH)\modules\nrfx\drivers\include
$(SDK_PATH)\modules\nrfx\drivers\src\prs
$(SDK_PATH)\integration\nrfx
$(SDK_PATH)\components\libraries\util
$(SDK_PATH)\components\libraries\log
$(SDK_PATH)\components\libraries\log\src
$(SDK_PATH)\components\libraries\strerror
$(SDK_PATH)\components\libraries\experimental_section_vars
$(SDK_PATH)\components\libraries\uart
$(SDK_PATH)\components\libraries\fifo
$(SDK_PATH)\components\softdevice\s140\headers
$(SDK_PATH)\integration\nrfx\legacy
```
在代码中，按需添加以下头文件：  
```
#include <stdint.h>
#include <stdbool.h>
#include <nrf_gpio.h> // GPIO操作
#include <nrfx_gpiote.h> // GPIOTE组件库
#include <nrfx_timer.h>　// 计时器组件库
#include <sdk_errors.h> // debug查错相关
#include <app_error.h> // debug查错相关
#include <app_uart.h>　// 串口UART相关
```

# 需要手动添加的库文件
可以按需添加以下库文件：
```
modules\nrfx\drivers\src\nrfx_gpiote.c (GPIOTE组件库)
modules\nrfx\drivers\src\nrfx_timer.c (计时器组件库)
modules\nrfx\drivers\src\prs\nrfx_prs.c (外设资源共享库，UART，UARTE必需)
modules\nrfx\drivers\src\nrfx_uart.c (UART串口库)
modules\nrfx\drivers\src\nrfx_uarte.c (UARTE串口库)
components\libraries\uart\app_uart_fifo.c (UART应用FIFO驱动库)
components\libraries\uart\retarget.c (串口重定义文件)
components\libraries\util\app_util_platform.c (UART需要)
integration\nrfx\legacy\nrf_drv_uart.c (老版本UART窗口驱动库)
components\libraries\fifo\app_fifo.c (应用FIFO的驱动库)
components\libraries\util\app_error.c (debug查错组件库)
components\libraries\util\app_error_weak.c (debug查错组件库)
components\libraries\util\app_error_handler_gcc.c (debug查错组件库，如果是Keil和IAR则要使用另外的c文件)
```

# SDK使用中的一些坑
## 新版芯片外设驱动nrfx
SDK14及以前版本SDK使用nrf_drv老版本外设驱动（又称legacy），SDK15及其以后版本使用nrfx新版本外设驱动。根据官方文档的描述，如果想使用新版nrfx驱动，必须将`sdk_config.h`文件中与`nrf_drv_*`相关的设置全部删除。注意把这些设置改为0是没用的，必须删除才有效果。  
官方文档对这一注意事项的描述：https://infocenter.nordicsemi.com/index.jsp -> Software Development Kit -> nRF5 SDK v16.0.0 -> Hardware Drivers -> Migration guide for nrfx drivers -> Migrate nrf_drv to nrfx drivers


# 函数
## nrf_gpio_cfg_output
函数原型：`void nrf_gpio_cfg_output(uint32_t pin_number)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：将`pin_number`对应的GPIO管脚配置为输出管脚，标准驱动能力，关闭感应。
## nrf_gpio_cfg_input
函数原型：`void nrf_gpio_cfg_input(uint32_t pin_number, nrf_gpio_pin_pull_t pull_config)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：将`pin_number`对应的GPIO管脚配置为输入管脚，设置为`pull_config`指定的指定的上下拉类型，标准驱动能力，关闭感应。  
## nrf_gpio_pin_set
函数原型：`void nrf_gpio_pin_set(uint32_t pin_number)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：当`pin_number`是GPIO输出管脚时，将其设置为高电平1。
## nrf_gpio_pin_clear
函数原型：`void nrf_gpio_pin_clear(uint32_t pin_number)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：当`pin_number`是GPIO输出管脚时，将其设置为低电平0。
## nrf_gpio_pin_toggle
函数原型：`void nrf_gpio_pin_toggle(uint32_t pin_number)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：当`pin_number`是GPIO输出管脚时，将其电平反转：低变高，高变低，0变1，1变0  
## nrf_gpio_pin_write
函数原型：`void nrf_gpio_pin_write(uint32_t pin_number, uint32_t value)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：当`pin_number`是GPIO输出管脚时，将其设置为高电平`value=1`或低电平`value=0`。
## nrf_gpio_pin_read
函数原型：`uint32_t nrf_gpio_pin_read(uint32_t pin_number)`  
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：当`pin_number`是GPIO输入管脚时，读取电平。返回值1为高电平，0为低电平。
## NVIC_EnableIRQ
用法：用来启用中断。它的参数一般使用常量`GPIOTE_IRQn`（GPIOTE）等，例如`NVIC_EnableIRQ(GPIOTE_IRQn);`
## nrfx_gpiote_init
函数原型：`nrfx_err_t nrfx_gpiote_init(void)`  
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，需要在最开始使用这个函数进行初始化。它会将中断类型设置为PORT中断，优先度设置为低。
```
ret_code_t err_code;
err_code = nrfx_gpiote_init();
```
## nrfx_gpiote_in_init
函数原型：
```
nrfx_err_t nrfx_gpiote_in_init(nrfx_gpiote_pin_t               pin,
                               nrfx_gpiote_in_config_t const * p_config,
                               nrfx_gpiote_evt_handler_t       evt_handler);
```
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数初始化某个具体的管脚为GPIOTE触发管脚。例如
```
err_code = nrfx_gpiote_in_init(11, &in_config, in_pin_handler);
```
表示初始化11号管脚为GPIOTE触发管脚，管脚相关设置为`in_config`。而`in_pin_handler`是自定义的函数，当中断触发时，执行这个函数的代码
```
void in_pin_handler(nrfx_gpiote_pin_t pin, nrf_gpiote_polarity_t action) // 中断触发时传入的参数
{

}
```
## nrfx_gpiote_out_init
函数原型：
```
nrfx_err_t nrfx_gpiote_out_init(nrfx_gpiote_pin_t                pin,
                                nrfx_gpiote_out_config_t const * p_config)
```
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数初始化某个具体的管脚为GPIOTE任务管脚。例如
```
nrfx_gpiote_out_init(15, &config);
```
表示初始化15号管脚为GPIOTE任务管脚，管脚相关设置为`config`。
## nrfx_gpiote_in_event_enable
函数原型：`void nrfx_gpiote_in_event_enable(nrfx_gpiote_pin_t pin, bool int_enable)`  
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数启用某个具体的管脚为GPIO触发管脚。例如`nrfx_gpiote_in_event_enable(11,true)`，表示启用第11号管脚。第二个参数必须是`true`。
## nrfx_gpiote_out_task_enable
函数原型：`void nrfx_gpiote_out_task_enable(nrfx_gpiote_pin_t pin)`  
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数启用某个具体的管脚为GPIO任务管脚。
## nrfx_gpiote_out_task_trigger
函数原型：`void nrfx_gpiote_out_task_trigger(nrfx_gpiote_pin_t pin)`  
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数手动触发某个任务管脚。
## nrfx_timer_init
函数原型：
```
nrfx_err_t nrfx_timer_init(nrfx_timer_t const * const  p_instance,
                           nrfx_timer_config_t const * p_config,
                           nrfx_timer_event_handler_t  timer_event_handler);
```
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_timer.c`  
用法：使用定时器组件库的时候，用这个函数来初始化某个定时器。`timer_event_handler`是自定义的函数，当定时结束的时候，触发执行这个函数的代码。
```
void timer_event_handler(nrf_timer_event_t event_type, void* p_context){

}
```
## nrfx_timer_ms_to_ticks
函数原型：
```
uint32_t nrfx_timer_ms_to_ticks(nrfx_timer_t const * const p_instance,
                                uint32_t                   timer_ms)
```
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
用法：用来计算在某个计时器配置下，某段时间相当于晶振振荡多少次。`p_instance`是指向某个计时器地址的指针，`timer_ms`则是时间长度，单位毫秒。函数返回晶振的振荡次数。
## nrfx_timer_extended_compare
函数原型：
```
void nrfx_timer_extended_compare(nrfx_timer_t const * const p_instance,
                                 nrf_timer_cc_channel_t     cc_channel,
                                 uint32_t                   cc_value,
                                 nrf_timer_short_mask_t     timer_short_mask,
                                 bool                       enable_int);
```
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_timer.c`  
用法：用来设置计时器的比较通道。`p_instance`是指向某个计时器地址的指针。`cc_channel`设置为某个比较通道，例如`NRF_TIMER_CC_CHANNEL0`。`cc_value`为比较值，当计时器的振荡次数等于该值的时候，触发。`timer_short_mask`一般设置为`NRF_TIMER_SHORT_COMPARE0_CLEAR_MASK`或`NRF_TIMER_SHORT_COMPARE0_STOP_MASK`。前者表示触发的时候，计时器清零，从头开始计时。后者表示触发的时候，停止计时。最后一项一般设置为`true`,表示用中断的方式触发。
## nrfx_timer_enable
函数原型：
```
void nrfx_timer_enable(nrfx_timer_t const * const p_instance)
```
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_timer.c`  
用法：用来启动计时器。`p_instance`是指向某个计时器地址的指针。


# 变量类型
## nrf_gpio_pin_pull_t
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：用来指定GPIO管脚的上下拉类型，可以取`NRF_GPIO_PIN_NOPULL`（无上下拉），`NRF_GPIO_PIN_PULLDOWN`（下拉），`NRF_GPIO_PIN_PULLUP`（上拉）三个值。
## nrfx_gpiote_in_config_t
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
用法：用来定义GPIOTE输入参数变量，实际应用中一般和`NRFX_GPIOTE_CONFIG_IN_SENSE_LOTOHI`，`NRFX_GPIOTE_CONFIG_IN_SENSE_HITOLO`，`NRFX_GPIOTE_CONFIG_IN_SENSE_TOGGLE`这三个宏配合使用。
## nrfx_gpiote_out_config_t
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
用法：用来定义GPIOTE输出参数变量，实际应用中一般和`NRFX_GPIOTE_CONFIG_OUT_TASK_LOW`，`NRFX_GPIOTE_CONFIG_OUT_TASK_HIGH`,`NRFX_GPIOTE_CONFIG_OUT_TASK_TOGGLE`这三个宏配合使用。
## nrfx_timer_t
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
用法：在使用计时器组件时，需要定义一个这个类型的计时器变量。实际中一般和`NRFX_TIMER_INSTANCE`这个宏配合使用。
## nrfx_timer_config_t
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
用法：用来定义计时器组件的配置变量，实际应用中一般和`NRFX_TIMER_DEFAULT_CONFIG`这个宏配合使用。
## app_uart_comm_params_t
头文件位置：`components\libraries\uart\app_uart.h`  
用法：这是一个结构体变量，用来定义UART使用的配置变量。


# 宏与常量
## NRF_GPIOTE
用法：这个常量定义了GPIOTE相关寄存器的基地址（nRF52840芯片中这个基地址为`0x40006000`）。在实际应用中，一般使用`NRF_GPIOTE->CONFIG[0]`这样的方法，在基地址的基础上加上`CONFIG[0]`偏移量，计算出每个具体寄存器的地址。  
## NRF_CLOCK
用法：这个常量定义了时钟晶振相关寄存器的基地址（nRF52840芯片中这个基地址为`0x40000000`）。在实际应用中，一般使用`NRF_CLOCK->EVENTS_HFCLKSTARTED`这样的方法，在基地址的基础上加上`EVENTS_HFCLKSTARTED`偏移量，计算出每个具体寄存器的地址。  
## NRF_TIMER0 NRF_TIMER1 NRF_TIMER2 NRF_TIMER3 NRF_TIMER4
用法：这几个常量定义了计时器TIMER相关的寄存器的基地址。在nRF52840手册的`6.30.5 Registers`小节中能查阅到各个寄存器的详情。
## GPIOTE_CONFIG_POLARITY_LoToHi
用法：这个常量定义了GPIOTE中断的触发条件，低电平变化到高电平的上升缘触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_LoToHi << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_POLARITY_HiToLo
用法：这个常量定义了GPIOTE中断的触发条件，高电平变化到低电平的下降缘触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_HiToLo << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_POLARITY_Toggle
用法：这个常量定义了GPIOTE中断的触发条件，只要出现电平变化都会触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_Toggle << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_POLARITY_Pos
用法：这个常量定义了GPIOTE触发条件设置对应的位。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_LoToHi << GPIOTE_CONFIG_POLARITY_Pos`将设置值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_PSEL_Pos
用法：这个常量定义了设置GPIOTE管脚的位。例如，`11 << GPIOTE_CONFIG_PSEL_Pos`，再赋值给`CONFIG[0-7]`寄存器。这么做的效果是将第11号管脚设置为GPIOTE。
## GPIOTE_CONFIG_MODE_Event
用法：这个常量表示将GPIOTE设置为事件模式，即对应的管脚是一个触发管脚。实际应用中，一般使用`GPIOTE_CONFIG_MODE_Event << GPIOTE_CONFIG_MODE_Pos`将设置值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_MODE_Pos
用法：这个常量定义了GPIOTE模式设置的位。例如，使用`GPIOTE_CONFIG_MODE_Event << GPIOTE_CONFIG_MODE_Pos`将事件设置值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_INTENSET_IN0_Set GPIOTE_INTENSET_IN0_Pos
用法：用来启用第0号GPIOTE通道，前者是设定值，后者是对应的位。实际应用中，一般使用`NRF_GPIOTE->INTENSET = GPIOTE_INTENSET_IN0_Set << GPIOTE_INTENSET_IN0_Pos`，修改`INTENSET`寄存器达到启用第0号GPIOTE通道的效果。同理用类似的方法可以启动1-7号通道。
## GPIOTE_INTENSET_IN0_Msk
用法：这个常量定义了`INTENSET`寄存器中，第0号通道所在位的掩码。实际应用中，一般使用`NRF_GPIOTE->INTENSET & GPIOTE_INTENSET_IN0_Msk`来取出第0号通道的设置值。同理类似的方法可以取出1-7号通道的值。
## NRFX_GPIOTE_CONFIG_IN_SENSE_LOTOHI
## NRFX_GPIOTE_CONFIG_IN_SENSE_HITOLO
## NRFX_GPIOTE_CONFIG_IN_SENSE_TOGGLE
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
用法：一般用来初始化GPIOTE输入参数变量，例如
```
nrfx_gpiote_in_config_t in_config = NRFX_GPIOTE_CONFIG_IN_SENSE_LOTOHI(true);
nrfx_gpiote_in_config_t in_config = NRFX_GPIOTE_CONFIG_IN_SENSE_TOGGLE(false);
```
这三个宏分别表示将GPIOTE输入的触发方式设置为：低电平->高电平，高电平->低电平，高低之间变化。  
后面的`true`表示使用高精度的EVENTS_IN中断（功耗高），`false`表示使用低精度的EVENTS_PORT中断（功耗低）。
## NRFX_GPIOTE_CONFIG_OUT_TASK_LOW
## NRFX_GPIOTE_CONFIG_OUT_TASK_HIGH
## NRFX_GPIOTE_CONFIG_OUT_TASK_TOGGLE(init_high)
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
用法：一般用来初始化GPIOTE输出参数变量，例如
```
nrfx_gpiote_out_config_t config = NRFX_GPIOTE_CONFIG_OUT_TASK_TOGGLE(true);
nrfx_gpiote_out_config_t config = NRFX_GPIOTE_CONFIG_OUT_TASK_LOW;
```
这三个宏分别表示将GPIOTE的输出任务模式设置为：低电平输出，高电平输出，电平翻转输出。  
`init_high=true`表示初始化为高电平，`init_high=false`表示初始化为低电平。
## NRFX_TIMER_INSTANCE(id)
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
用法：用来初始化计时器变量。`id`表示使用哪个计时器。
## NRFX_TIMER_DEFAULT_CONFIG
头文件位置：`modules\nrfx\drivers\include\nrfx_timer.h`  
用法：用来初始化计时器组件的配置变量。初始值由`sdk_config.h`中的设置决定。注意需要仔细估计振荡次数的计数值，选择合适的位宽和频率。如果位宽和频率选择不正确，则无法正确计数。
## APP_UART_FIFO_INIT
头文件位置：`components\libraries\uart\app_uart.h`  
用法：这个宏用来初始化UART功能，它需要带6个参数：  
1. `app_uart_comm_params_t`所定义的变量的地址  
2. RX缓冲区的大小，参考值`256`  
3. TX缓冲区的大小，参考值`256`  
4. 出错时跳转的函数，填写一个自定义的函数名。出错时会跳转到这个函数执行代码。如果不想使用可以填`NULL`。  
5. UART中断的优先级，填写枚举变量`app_irq_priority_t`所规定的几个值之一。参考值`APP_IRQ_PRIORITY_LOWEST`。
6. `uint32_t err_code`变量`err_code`，用于出错时返回错误码。


# 中断向量表定义的中断程序
## GPIOTE_IRQHandler
用法：在代码中自己编写`GPIOTE_IRQHandler`的代码。当GPIOTE中断发生的时候，会进入这个函数执行代码。
```
void GPIOTE_IRQHandler(void){

}
```
