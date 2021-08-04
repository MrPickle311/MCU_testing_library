# MCU Testing Library

This library is used to carry out unit tests on microcontrollers.
Results are printed by special codes , so it is necessary to use 
an externall application to visualize them. 
Such as this [application ](https://github.com/MrPickle311/unit_test_visualizer) 
is a good to be used . 

# How to

1. Connect MCU to PC
2. Upload testing program to MCU
3. Run and configure this [application ](https://github.com/MrPickle311/unit_test_visualizer)

# General Information

This library comes with a macro set. Library requires add 
include a usart library and define start code (a number from 0 to 255). 
It also requires define the following usart functions: 
- send byte 
- send text 
- get byte

It should work with any MCU family.

# Setup

1. Copy content to your project folder
2. Open unit_test.h and you will see this:
```
// add include to your usart library
#include 

// in these definitions place your usart functions
#define TEST_UART_SEND_TXT(txt_ptr)	
#define TEST_UART_SEND_BYTE(byte)	
#define TEST_UART_GET_BYTE()	

#define START_CODE
```

You must place here your usart library include and also
define three functions. For example

```
// add include to your usart library
#include "usart.h"

// in these definitions place your usart functions
#define TEST_UART_SEND_TXT(txt_ptr)	USART0_sendText(txt_ptr)
#define TEST_UART_SEND_BYTE(byte)	USART0_sendByte(byte)
#define TEST_UART_GET_BYTE()		USART0_getByte()

#define START_CODE 0x33
```

# Usage

There is example of usage:

```
#include <avr/io.h>
#include <avr/pgmspace.h>
#include <util/delay.h>
#include "unit_tests.h"

//just retun true
bool rtrn_true_function()
{
	return true;
}

TEST_CASE(test1)
{
	EXPECT_TRUE(rtrn_true_function());

	uint32_t xc = 433;
	EXPECT_EQUAL_uint32_t(xc,433);//passed

	int64_t openmb = -343434;
	EXPECT_EQUAL_int64_t(openmb,433);//failure

	uint16_t reg = 0b10000000;
	EXPECT_BIT_SET(reg,7);//exptect bit set at 7'th place

	uint16_t* ptr = NULL;
	EXPECT_NULL(ptr);//passed

	TEST_CASE_END();
}

TEST_CASE(test2)
{
	//there is range test
	int16_t name1 = 67;

	EXPTECT_IN_RANGE_int16_t(name1,-7,5090);//passed

	TEST_CASE_END();
}


int main(void)
{
	// setting up USART
	USART_Setup setup = USART_DefaultSettings;
	setup.startup_transmit_enable_ = true;
	setup.startup_receive_enable_ = true;
	USART0_init(setup);
	
	//tests loop
	while(true)
	{
		TEST_INIT();//wait for START_CODE
		
		EXAMINE_TEST_CASE(test1);
		EXAMINE_TEST_CASE(test2);
		
		TEST_PROTOCOL_END();
	}	
}
```

# Docs

TEST_INIT() - this macro initalizes testing procedure and waits for signal
from PC. This signal is a byte which value must be equal to START_CODE.

EXAMINE_TEST_CASE(case) - runs test case

TEST_PROTOCOL_END() - sends stop signal
