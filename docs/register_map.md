# ARM CPU Registers

| Register | Purpose |
|-----------|------------------|
| R0|  General scratch register, memory addresses, random-number return value, debounce counter, row-offset calculation|
| R1| Temporary values, random seed calculations, button input, score value|
| R2| LCD commands and addresses, GPIO configuration values, collision calculations, score hundreds digit|
| R3| LCD character data, obstacle countdown values, loop counters, score tens digit|
| R4| Top obstacle X position|
| R5| Temporary random values and LCD custom-character loop counter|
| R6| Previous obstacle position and outer delay-loop counter|
| R7| Previous player row and inner delay-loop counter|
| R8| Top obstacle animation frame; reused for score hundreds character at game over|
| R9| Bottom obstacle X position; reused for score tens character at game over|
| R10| Previous button state; reused for score ones character at game over|
| R11| Bottom obstacle animation frame|
| R12| Current player row: 0 for top and 1 for bottom|
| R13/SP| Stack pointer, used implicitly by PUSH and POP|
| R14/LR| Link register containing subroutine return addresses|
| R15/PC| Program counter, modified by branches and returns|
| APSR| Condition flags used by CMP, SUBS, ADDS, and conditional branches|

# STM32 Peripheral Registers

| Register | Address | Usage |
|-----------|------------------| ---------|
| RCC_AHB1ENR| 0X40023830 |Enables the GPIOA and GPIOC peripheral clocks |
| GPIO_MODER| 0X40020000 |Configures PA5, PA6, and PA7 as LCD control outputs; PA0 remains an input |
| GPIOA_IDR| 0X40020010 | Reads the active-low pushbutton on PA0|
| GPIOA_ODR| 0X40020014 |Writes LCD RS, RW, and Enable signals on PA5–PA7 |
| GPIOC_MODER| 0X40020800|Configures PC0–PC7 as outputs |
| GPIOC_ODR| 0X40020814| Writes the LCD’s 8-bit data and command values|

# LCD Internal Registers

| LCD Register | RS | Purpose |
|--------------|----|---------|
|Instruction Register| 0| Receives commands and memory addresses|
|Data Register| 1| Receives charcters and custom-character data|


Important LCD addresses:
| Value| Purpose|
|------|--------|
|0x80|Top-row DDRAM base address|
|0xC0|Bottom-row DDRAM base address|
|0x40|Custom character 0 CGRAM address|
|0x48|Custom character 1 CGRAM address|
|0x50|Custom character 2 CGRAM address|
|0x58|Player character CGRAM address|
|0x38|8-bit, two-line LCD mode|
|0x0C|Display on, cursor off|
|0x01|Clear display|
|0x06|Increment cursor without shifting|

