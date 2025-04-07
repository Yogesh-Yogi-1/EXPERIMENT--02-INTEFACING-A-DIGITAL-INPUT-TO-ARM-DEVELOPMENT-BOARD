# EXPERIMENT--02-INTEFACING-A-DIGITAL-INPUT-TO-ARM-DEVELOPMENT-BOARD
## Aim: To Interface a Digital Input  (userpush button  ) to ARM   development board and write a  program to obtain  the data and flash the led  
## Components required: STM32 CUBE IDE, ARM IOT development board,  STM programmer tool.
## Theory 
The full form of an ARM is an advanced reduced instruction set computer (RISC) machine, and it is a 32-bit processor architecture expanded by ARM holdings. The applications of an ARM processor include several microcontrollers as well as processors. The architecture of an ARM processor was licensed by many corporations for designing ARM processor-based SoC products and CPUs. This allows the corporations to manufacture their products using ARM architecture. Likewise, all main semiconductor companies will make ARM-based SOCs such as Samsung, Atmel, TI etc.

 
  
## Procedure:
 1. click on STM 32 CUBE IDE, the following screen will appear 
 ![image](https://user-images.githubusercontent.com/36288975/226189166-ac10578c-c059-40e7-8b80-9f84f64bf088.png)

 2. click on FILE, click on new stm 32 project 
 ![image](https://user-images.githubusercontent.com/36288975/226189215-2d13ebfb-507f-44fc-b772-02232e97c0e3.png)
![image](https://user-images.githubusercontent.com/36288975/226189230-bf2d90dd-9695-4aaf-b2a6-6d66454e81fc.png)
3. select the target to be programmed  as shown below and click on next 

![image](https://user-images.githubusercontent.com/36288975/226189280-ed5dcf1d-dd8d-43ae-815d-491085f4863b.png)

4.select the program name 
![image](https://user-images.githubusercontent.com/36288975/226189316-09832a30-4d1a-4d4f-b8ad-2dc28f137711.png)


5. corresponding ioc file will be generated automatically 
![image](https://user-images.githubusercontent.com/36288975/226189378-3abbdee2-0df6-470f-a3cd-79c74e3d3ad8.png)

6.select the appropriate pins as gipo, in or out, USART or required options and configure 
![image](https://user-images.githubusercontent.com/36288975/226189403-f7179f1a-3eae-4637-826b-ab4ec35ba1e1.png)
![image](https://user-images.githubusercontent.com/36288975/226189425-2b2414ce-49b3-4b61-a260-c658cb2e4152.png)


7.click on cntrl+S , automaticall C program will be generated 
![image](https://user-images.githubusercontent.com/36288975/226189443-8b43451d-0b14-47e4-a20b-cc09c6ad8458.png)
![image](https://user-images.githubusercontent.com/36288975/226189450-85ffa969-2ffb-4788-81e5-72d60fdda0f1.png)
8. edit the program and as per required 
![image](https://user-images.githubusercontent.com/36288975/226189461-a573e62f-a109-4631-a250-a20925758fe0.png)

9. use project and build all 
![image](https://user-images.githubusercontent.com/36288975/226189554-3f7101ac-3f41-48fc-abc7-480bd6218dec.png)
10. once the project is bulild 
![image](https://user-images.githubusercontent.com/36288975/226189577-c61cc1eb-3990-4968-8aa6-aefffc766b70.png)

11. click on debug option 
![image](https://user-images.githubusercontent.com/36288975/226189625-37daa9a3-62e9-42b5-a5ce-2ac63345905b.png)

12. connect the  ARM board to power supply and usb 


13. check for execution of the output using run option 



## STM 32 CUBE PROGRAM :
```

// *  Created on: 10/04/2023
 //

#include "lcd.h"
const uint8_t ROW_16[] = {0x00, 0x40, 0x10, 0x50};
const uint8_t ROW_20[] = {0x00, 0x40, 0x14, 0x54};
/************************************** Static declarations **************************************/

static void lcd_write_data(Lcd_HandleTypeDef * lcd, uint8_t data);
static void lcd_write_command(Lcd_HandleTypeDef * lcd, uint8_t command);
static void lcd_write(Lcd_HandleTypeDef * lcd, uint8_t data, uint8_t len);


/************************************** Function definitions **************************************/

/**
 * Create new Lcd_HandleTypeDef and initialize the Lcd
 */
Lcd_HandleTypeDef Lcd_create(
		Lcd_PortType port[], Lcd_PinType pin[],
		Lcd_PortType rs_port, Lcd_PinType rs_pin,
		Lcd_PortType en_port, Lcd_PinType en_pin, Lcd_ModeTypeDef mode)
{
	Lcd_HandleTypeDef lcd;

	lcd.mode = mode;

	lcd.en_pin = en_pin;
	lcd.en_port = en_port;

	lcd.rs_pin = rs_pin;
	lcd.rs_port = rs_port;

	lcd.data_pin = pin;
	lcd.data_port = port;

	Lcd_init(&lcd);

	return lcd;
}

/**
 * Initialize 16x2-lcd without cursor
 */
void Lcd_init(Lcd_HandleTypeDef * lcd)
{
	if(lcd->mode == LCD_4_BIT_MODE)
	{
			lcd_write_command(lcd, 0x33);
			lcd_write_command(lcd, 0x32);
			lcd_write_command(lcd, FUNCTION_SET | OPT_N);				// 4-bit mode
	}
	else
		lcd_write_command(lcd, FUNCTION_SET | OPT_DL | OPT_N);


	lcd_write_command(lcd, CLEAR_DISPLAY);						// Clear screen
	lcd_write_command(lcd, DISPLAY_ON_OFF_CONTROL | OPT_D);		// Lcd-on, cursor-off, no-blink
	lcd_write_command(lcd, ENTRY_MODE_SET | OPT_INC);			// Increment cursor
}

/**
 * Write a number on the current position
 */
void Lcd_int(Lcd_HandleTypeDef * lcd, int number)
{
	char buffer[11];
	sprintf(buffer, "%d", number);

	Lcd_string(lcd, buffer);
}

/**
 * Write a string on the current position
 */
void Lcd_string(Lcd_HandleTypeDef * lcd, char * string)
{
	for(uint8_t i = 0; i < strlen(string); i++)
	{
		lcd_write_data(lcd, string[i]);
	}
}

/**
 * Set the cursor position
 */
void Lcd_cursor(Lcd_HandleTypeDef * lcd, uint8_t row, uint8_t col)
{
	#ifdef LCD20xN
	lcd_write_command(lcd, SET_DDRAM_ADDR + ROW_20[row] + col);
	#endif

	#ifdef LCD16xN
	lcd_write_command(lcd, SET_DDRAM_ADDR + ROW_16[row] + col);
	#endif
}

/**
 * Clear the screen
 */
void Lcd_clear(Lcd_HandleTypeDef * lcd) {
	lcd_write_command(lcd, CLEAR_DISPLAY);
}

void Lcd_define_char(Lcd_HandleTypeDef * lcd, uint8_t code, uint8_t bitmap[]){
	lcd_write_command(lcd, SETCGRAM_ADDR + (code << 3));
	for(uint8_t i=0;i<8;++i){
		lcd_write_data(lcd, bitmap[i]);
	}

}


/************************************** Static function definition **************************************/

/**
 * Write a byte to the command register
 */
void lcd_write_command(Lcd_HandleTypeDef * lcd, uint8_t command)
{
	HAL_GPIO_WritePin(lcd->rs_port, lcd->rs_pin, LCD_COMMAND_REG);		// Write to command register

	if(lcd->mode == LCD_4_BIT_MODE)
	{
		lcd_write(lcd, (command >> 4), LCD_NIB);
		lcd_write(lcd, command & 0x0F, LCD_NIB);
	}
	else
	{
		lcd_write(lcd, command, LCD_BYTE);
	}

}

/**
 * Write a byte to the data register
 */
void lcd_write_data(Lcd_HandleTypeDef * lcd, uint8_t data)
{
	HAL_GPIO_WritePin(lcd->rs_port, lcd->rs_pin, LCD_DATA_REG);			// Write to data register

	if(lcd->mode == LCD_4_BIT_MODE)
	{
		lcd_write(lcd, data >> 4, LCD_NIB);
		lcd_write(lcd, data & 0x0F, LCD_NIB);
	}
	else
	{
		lcd_write(lcd, data, LCD_BYTE);
	}

}

/**
 * Set len bits on the bus and toggle the enable line
 */
void lcd_write(Lcd_HandleTypeDef * lcd, uint8_t data, uint8_t len)
{
	for(uint8_t i = 0; i < len; i++)
	{
		HAL_GPIO_WritePin(lcd->data_port[i], lcd->data_pin[i], (data >> i) & 0x01);
	}

	HAL_GPIO_WritePin(lcd->en_port, lcd->en_pin, 1);
	DELAY(1);
	HAL_GPIO_WritePin(lcd->en_port, lcd->en_pin, 0);
}

```
## Output  :
 ![Screenshot 2025-04-07 093635](https://github.com/user-attachments/assets/19e8ee53-c543-452d-ad8d-d0c3622c4959)

## layout of the circuit 
 ![Screenshot 2025-04-07 094005](https://github.com/user-attachments/assets/599aa927-35ca-416d-ae89-689961245ef5)

## Result :
Interfacing a digital Input (Pushbutton ) with ARM microcontroller based IOT development is executed and the results are verified.
