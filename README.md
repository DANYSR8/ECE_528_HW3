# ECE_528_HW3

**1. (10 pts) The MSP432 uses the eUSCI_A modules for UART communication. Describe the function of the following registers:**

- **UCAxBRW (Baud Rate Control Word)**

This register holds the measurements for baud rate auto detection. This holds the values that in short tell us what baud (data rate) the communication is using. It's the clock prescaler setting of the baud-rate generator.

- **UCAxMCTLW (Modulation Control Word)**

This register holds the modulation pattern for BITCLK16 and tells whether oversampling is enabled or not.

- **UCAxCTLW0 (Control Word 0)**

This register holds information regarding the structure of the data/words being communicated: whether or not parity bits are used, if the data is MSB or LSB first, the length of the data, number of stop bits, UART mode, synchronization mode, what the clock source is, and when an interrupt should be sent based on if there is an error in the data or a break in the data, as well as if the next word is an address or a break, along with if the eUSCI should be held in a reset state.

- **UCAxRXBUF (Receive Buffer)**

    - This register contains the last received character from the receive shift register.

- **UCAxTXBUF (Transmit Buffer)**

    - This register holds the data waiting to be moved into the transmit shift register and transmitted on UCAxTXD.

---
**2. (5 pts) What is the purpose of the UCAxSTATW status register in the UART module? Which bits indicate the following?**

This status register's main purpose is to give flags and/or provide a status that notifies if there are any errors or missing data from the message, as well as tell us if the transmit or receive operation is active.

- **Whether the UART module is currently transmitting**

    -   Bit 0 displays whether or not eUSCI_A is busy transmitting or receiving.

- **Whether a framing, parity, or overrun error has occurred**

    - Bits 6, 4, and 5 respectively tell us if any of these errors have occurred.

- **Whether the RX buffer contains unread data**

    - There is no explict bit in this register that contains this infomation

---
**3. (5 pts) Assume that the eUSCI_A3 module is using SMCLK as its clock source, which has a frequency of 12 MHz. Calculate the value that is needed to be written to the UCAxBRW register to configure the eUSCI_A3 module for 115200 baud.**

Need to write a value of $N = 104$ into UCAxBRW to get a 115200 baud 

$Baud Rate = \frac{1}{BitDuration}$

$PreScalar(N) = \frac{ClockFrequency}{BaudRate} \rightarrow \frac{12MHz}{115200} = 104.16 $





**4. (a) (15 pts) Write a function named UART2_Init that configures the eUSCI_A2 module for the following settings:**
- **Data Length: 8 bits**
    -  Clear Bit 12 in UCAxCTLW0 
- **Parity: None**
    - Clear Bit 15 in UCAxCTLW0
- **Stop Bits: 1**
    - Clear Bit 11 in UCAxCTLW0
- **Bit Order Most Significant Bit (MSB) First**
    - Set Bit 13 in UCAxCTLW0
- **Clock Source: SMCLK (12 MHz)**
    - Set Bit 7-6 as 11 in UCAxCTLW0
- **Pins: P3.2 (UCA2TXD) and P3.3 (UCA2RXD)**
    - Set Pins to the following using Table 6-64 of the MSP432 Datasheet (Bitmask = 0x00001100b / 0x0C)
    - DIR = X 
    - SEL1 = 0
    - SEL0 = 1 

```c
void UART2_Init(void){

    P3->SEL0 |= 0X0C; 
    P3->SEL1 &= ~0x0C; 

    EUSCI_A2->CTLW0 |= 0x0001; // Setting bit 0, this is good practice given by lecture slides to allow congifuration Register  

    //-------------------------------------------------------------
    // Setting Given 
    EUSCI_A2->CTLW0 &= ~0x1000 ; //Clearing bit 12  
    EUSCI_A2->CTLW0 &= ~0x8000 ; //Clearing bit 15 
    EUSCI_A2->CTLW0 &= ~0x0800 ; //Clearing bit 11  
    EUSCI_A2->CTLW0 |= 0x2000 ; //Seting bit 13 
    EUSCI_A2->CTLW0 |= 0x00C0 ; //Seting bit 6 and 7 

    // Additonal setup 
    EUSCI_A2->CTLW0 &= ~0x0600 ; //Selecting UART Mode ,Clearing bits 10,9
    EUSCI_A2->CTLW0 &= ~0x0100 ; //Setting to Unsyc Mode , Clearing bit 8

    //----------------------------------------------------------------

    EUSCI_A2->CTLW0 &= ~0x0001; // Clearing bit 0, to resume operation of the register

}
```

The code between the dashed lines can be condense to one line `EUSCI_A2->CTLW0 = 0x20C1;`





**(b) (10 pts) Suppose that you have a main program that is transmitting data using the eUSCI_A2 module given the configuration in part (a). Draw the waveform for the UART TX data line if the main program is transmitting to bytes: 0xF1 followed by 0x55. Clearly label the start bits, data bits, and stop bits. Assume that there is a 50 ms delay between the to bytes.**

Part (a) mentions that we have a stop bit with of 1 bit so the we can infer that a bit duration is 50ms as well. We can also see that hte Baud is calulate using the equation below. 

$Baud Rate = \frac{1}{BitDuration} \rightarrow \frac{1}{50ms} = 20 $

Used DRAWIO for generating image: 

![UART_2Byte_MSG](./Images/Q4_PartB-HW3%20UART%20Submisison_Transparent.png)



**5. (5 pts) What is the function of the eUSCI_B module? How does SPI communication differ from UART communication in terms of:**

The eUSCI_B module functions as an additional serial communication module, but this one only offers $I^2C$ or SPI and does not support UART.

- **Synchronization**

    - SPI synchronization is controlled by the "Master," since it generates the clock signal that is to be used by the slaves (Synchronous), while UART uses a pre-determined/pre-set baud rate that both devices agree on, and the receiver resynchronizes on each start bit (Asynchronous).

- **Clocking**

    - SPI uses the Master's clock signal, which is shared among everyone, while in UART each device makes its own clock signal using a local source.

- **Signaling**

    - SPI uses 4 wires (SCLK, MOSI, MISO, CS), while UART uses only 2 (TX and RX), allowing SPI to communicate with multiple slaves from one master, while UART is point-to-point.

- **Data Framing**

    - SPI doesn't use any framing like UART, since it has the "CS" line marking when the transmission starts and ends. So SPI is just raw data, while UART has the addition of start/stop bits and an optional parity bit.


**6. (a) (5 pts) Explain the role of the following SPI signals:**
- **SCLK**

    - This signal is the clock signal, generated by the master, being used for communications.

- **CS**

    - This is the "Chip Select" line that allows the master to select what chip/slave to send data to.

- **MISO**

    - This is the "Master Input/Slave Output" signal that contains the data being sent back to the master by the slave.

- **MOSI**

    - This is the "Master Output/Slave Input" signal that contains the data being sent to the slave from the master.

**(b) (5 pts) What happens if multiple SPI peripheral devices have their CS line asserted simultaneously on the MSP432?**

If multiple peripherals have their CS line asserted, the communication among the devices fails, since all are trying to drive the shared MISO line back to the one master at the same time. This leads to corrupted data and unknown/undesirable states, and the selected slaves may also act on commands meant for another device.


**7. (a) (5 pts) Briefly describe the function of the UCBxCTLW0 (Control Word 0) register from the eUSCI_B module.**

The UCBxCTLW0 register controls what clock is to be used, its phase and polarity, what synchronization mode, the direction of the data (MSB or LSB first), its length, whether the peripheral is in master or slave mode, and overall enabling of the peripheral (software reset).


**(b) (5 pts) Which bits need to be cleared or set in the UCBxCTLW0 register to configure the SPI mode used by the eUSCI_B module? Provide a description of each bit.**

Bits 10, 9, and 8 determine what SPI mode is to be used: bits 10 and 9 select what specific mode the SPI will be following, and bit 8 determines what synchronization mode is to be used (must be 1 for SPI). Below is a summary of the bits.
```
    Bit 8 (UCSYNC)
    0b = Asynchronous mode
    1b = Synchronous mode (required for SPI)

    Bits 10 and 9 (UCMODEx)
    00b = 3-pin SPI
    01b = 4-pin SPI with UCxSTE active high: Slave enabled when UCxSTE = 1
    10b = 4-pin SPI with UCxSTE active low: Slave enabled when UCxSTE = 0
    11b = I2C mode (not SPI)

    Bit 15 (UCCKPH) - Clock phase select
    Bit 14 (UCCKPL) - Clock polarity select (0b = inactive low, 1b = inactive high)
    Bit 13 (UCMSB)  - 0b = LSB first, 1b = MSB first
    Bit 12 (UC7BIT) - 0b = 8-bit data, 1b = 7-bit data
    Bit 11 (UCMST)  - 0b = Slave mode, 1b = Master mode
    Bit 1  (UCSTEM) - STE pin function in 4-pin master mode
    Bit 0  (UCSWRST) - 1b = hold in reset while configuring, 0b = release for operation
```


**(c) (5 pts) Which field in the UCBxCTLW0 register selects the clock source used by the eUSCI_B module? Provide the values used and the corresponding clock sources.**

The UCSSELx field, which is bits 7 and 6. Below are the clock sources to be selected.

```
00b = Reserved
01b = ACLK
10b = SMCLK
11b = SMCLK
```

**8. (a) (15 pts) Write a function named EUSCI_B0_SPI_Init that configures the eUSCI_B0 module for SPI communication with the following settings:**

- **SPI Mode: 0**
    
    - Clear Bit 14 in CTLW0 
    - Set Bit 15 in CTLW0  

- **Data Length: 8 bits**

    - Clear Bit 12 in CTLW0

- **Bit Order: Most Significant Bit (MSB) First**

    - Set Bit 13 in CTLW0

- **Clock Source: SMCLK (12 MHz)**

    - Set bit 7 and 6 in CTLW0

- **SPI Clock Frequency: 1 MHz**

    - $ f_{Bitclock} = \frac{f_{BRCLK}}{UCBRx} \rightarrow \frac{12MHz}{UCBRx} = 1MHz $ 
    - Input value 12 into UCBRx ( 0x000C)

- **SPI format: 3-wire SPI**

    - Clear Bit 10 and 9 in CTLW0

- **Addtional Setups**
    
    - Clear Bit 1 in CTLW0 ( UCSTEM feild )
    - Set Bit 11 in CTLW0  ( UCMST feild )

- **SPI Pins:**
    - **P1.5: SCLK (UCB0CLK)**
    - **P1.6: MOSI (UCB0SIMO)**
    - **P1.7: MIS-(UCB0SOMI)**
    - **P9.0: Active Low Chip Select (GPIO)**


```c
void EUSCI_B0_SPI_Init(void){

    // Setting up Pin 1 primary function
    P1->SEL0 |= 0XE0; 
    P1->SEL1 &= ~0xE0; 

    // Setting up Pin 9 as GPIO Active Low 
    P9->SEL0 &= ~0X01; 
    P9->SEL1 &= ~0x01;
    P9->OUT |= 0X01; 
    P9->DIR |= 0X01;
    

    EUSCI_B0->CTLW0 |= 0x0001; // Setting bit 0, this is good practice given by lecture slides to allow congifuration Register  

    //-------------------------------------------------------------

    //Given Settings 

    EUSCI_B0->CTLW0 |=  0x8000;   // Set bit 15 (UCCKPH = 1)
    EUSCI_B0->CTLW0 &= ~0x4000;   // Clear bit 14 (UCCKPL = 0)

    EUSCI_B0->CTLW0 &= ~0x1000; // Clearing Bit 12
    EUSCI_B0->CTLW0 |= 0x2000; // Setting Bit 13

    EUSCI_B0->CTLW0 |= 0x00C0; // Setting Bit 6 and 7
    EUSCI_B0->CTLW0 &= ~0x0600; // Clear Bit 9 and 10

    EUSCI_B0->BRW = 0x000C; // Inputing prescale value of 12 

    // Additonal Setup 

    EUSCI_B0->CTLW0 &= ~0x0002; // Clearing Bit 1 
    EUSCI_B0->CTLW0 |=  0x0100; // // Set bit 8 (UCSYNC)
    EUSCI_B0->CTLW0 |= 0x0800; // Setting Bit 11


    //----------------------------------------------------------------

    EUSCI_B0->CTLW0 &= ~0x0001; // Clearing bit 0, to resume operation of the register

}
```


**(b) (10 pts) If a main program transmits one byte with a value of 0x57, draw the waveforms for:**

- **Chip Select (Active Low)**
- **SCLK**
- **MOSI**

Used DRAWIO for generating image: 

![SPI_1Byte_MSG](./Images/Q8_PartB-%20HW3%20SPI%20Submisison.png)