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
$(SDK_PATH)\modules\nrfx\templates\nRF52840
$(SDK_PATH)\modules\nrfx\templates
$(SDK_PATH)\modules\nrfx\drivers\include
$(SDK_PATH)\components\libraries\util
$(SDK_PATH)\components\libraries\log
$(SDK_PATH)\components\libraries\log\src
$(SDK_PATH)\components\libraries\strerror
$(SDK_PATH)\components\libraries\experimental_section_vars
$(SDK_PATH)\components\softdevice\s140\headers
```
在代码中，按需添加以下头文件：  
```
#include <nrf_gpio.h> // GPIO操作
#include <nrfx_gpiote.h> // GPIOTE组件库
#include <sdk_errors.h> // debug查错相关
#include <app_error.h> // debug查错相关
```

# 需要手动添加的库文件
可以按需添加以下库文件：
```
modules\nrfx\drivers\src\nrfx_gpiote.c (GPIOTE组件库)
components\libraries\util\app_error.c (debug查错组件库)
components\libraries\util\app_error_weak.c (debug查错组件库)
components\libraries\util\app_error_handler_gcc.c (debug查错组件库，如果是Keil和IAR则要使用另外的c文件)
```


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
## sd_nvic_EnableIRQ
函数原型：`uint32_t sd_nvic_EnableIRQ(IRQn_Type IRQn);`  
头文件位置：`components\softdevice\s140\headers\nrf_nvic.h`  
用法：用来启用中断。这个函数相当于`NVIC_EnableIRQ`。它的参数一般使用常量`GPIOTE_IRQn`（GPIOTE）等。
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
## nrfx_gpiote_in_event_enable
函数原型：`void nrfx_gpiote_in_event_enable(nrfx_gpiote_pin_t pin, bool int_enable)`  
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
库文件位置：`modules\nrfx\drivers\src\nrfx_gpiote.c`  
用法：使用GPIOTE驱动组件库时，用这个函数启用某个具体的管脚为GPIO触发管脚。例如`nrfx_gpiote_in_event_enable(11,true)`，表示启用第11号管脚。第二个参数必须是`true`。


# 变量类型
## nrf_gpio_pin_pull_t
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：用来指定GPIO管脚的上下拉类型，可以取`NRF_GPIO_PIN_NOPULL`（无上下拉），`NRF_GPIO_PIN_PULLDOWN`（下拉），`NRF_GPIO_PIN_PULLUP`（上拉）三个值。
## nrfx_gpiote_in_config_t
头文件位置：`modules\nrfx\drivers\include\nrfx_gpiote.h`  
用法：用来定义GPIOTE输入参数变量，实际应用中一般和`NRFX_GPIOTE_CONFIG_IN_SENSE_LOTOHI`，`NRFX_GPIOTE_CONFIG_IN_SENSE_HITOLO`，`NRFX_GPIOTE_CONFIG_IN_SENSE_TOGGLE`这三个宏配合使用。


# 宏与常量
## NRF_GPIOTE
用法：这个常量定义了GPIOTE相关寄存器的基地址（nRF52840芯片中这个基地址为`0x40006000`）。在实际应用中，一般使用`NRF_GPIOTE->CONFIG[0]`这样的方法，在基地址的基础上加上`CONFIG[0]`偏移量，计算出每个具体寄存器的地址。  
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


# 中断向量表定义的中断程序
## GPIOTE_IRQHandler
用法：在代码中自己编写`GPIOTE_IRQHandler`的代码。当GPIOTE中断发生的时候，会进入这个函数执行代码。
```
void GPIOTE_IRQHandler(void){

}
```
