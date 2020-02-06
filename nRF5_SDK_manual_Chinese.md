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
用法：用来启用中断。这个函数相当于`NVIC_EnableIRQ`。它的取值一般使用常量`GPIOTE_IRQn`（GPIOTE）等。


# 变量类型
## nrf_gpio_pin_pull_t
头文件位置：`modules\nrfx\hal\nrf_gpio.h`  
用法：用来指定GPIO管脚的上下拉类型，可以取`NRF_GPIO_PIN_NOPULL`（无上下拉），`NRF_GPIO_PIN_PULLDOWN`（下拉），`NRF_GPIO_PIN_PULLUP`（上拉）三个值。


# 常量
## NRF_GPIOTE
用法：这个常量定义了GPIOTE相关寄存器的基地址（nRF52840芯片中这个基地址为`0x40006000`）。在实际应用中，一般使用`NRF_GPIOTE->CONFIG[0]`这样的方法，在基地址的基础上加上`CONFIG[0]`偏移量，计算出每个具体寄存器的地址。  
## GPIOTE_CONFIG_POLARITY_LoToHi
用法：这个常量定义了GPIOTE中断的触发条件，低电平变化到高电平的上升缘触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_LoToHi << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_POLARITY_HiToLo
用法：这个常量定义了GPIOTE中断的触发条件，高电平变化到低电平的下降缘触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_HiToLo << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。
## GPIOTE_CONFIG_POLARITY_Toggle
用法：这个常量定义了GPIOTE中断的触发条件，只要出现电平变化都会触发。实际应用中，一般使用`GPIOTE_CONFIG_POLARITY_Toggle << GPIOTE_CONFIG_POLARITY_Pos`将这个值偏移到对应的位，再赋值给`CONFIG[0-7]`寄存器。