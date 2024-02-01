## Images --> for review of concept and terms 
1. Push pull drive
2. Noise Margin
3. Open Drain logic
4. Schmitt Trigger Voltage Characteristics
5. Opamp

# I2C Protocol

## Basic Understanding

1. Bus interface connection protocol.
2. SDA (serial data line --> data line (half duplex) ) and SCL (serial clock line --> control signal for I2C)
3. 9-bit protocol. 8 for address and 1 reserved for ACK/NACK bit i.e Acknowledgment and Not Acknowledge.
4. SDA and SCL bus lines are Open Drain; connected with pullup resistors as devices are active low.

[Open-Drain](https://www.analog.com/en/design-center/glossary/open-drain-collector.html) mode is for FET wherein the drain connected to the output is pulled to High in OFF state (pull-up resistor) and reflect the required output in ON state

### Signal format

Write Operation is shown here:

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/083aa7cf-e3d7-4fe9-92fa-7c9b51276fad)

Read Operation is shown here:

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/ab2e3667-2c05-4723-8161-218308b0b846)

### Working

1. Master checks if the SDA and SCL lines are **IDLE** (floating high).
2. Master asserts control over the bus by generating the **START** signal (SDA --> H-L when SCL --> H; after which SCL --> H-L).
3. Master sends an address (7 to 10 bits --> includes the **R/W bit**; 1 --> M reads from S; 0 --> M writes to S) to all the connected slaves (max approx 1023) via the **SDA line serially**. If the address matches, the respective slave will send an **ACK** bit by pulling SDA low for 1 bit.
4. Based of the R/W bit, either Master sends data or receives data in the form of **8 bit data frames** followed by an **ACK** bit.
5. Once the operation is complete, the Master can either release the bus by generating the **STOP** signal (SDA --> L-H when SCL --> H;SCL --> L-H before this) or **START** signal if it wants to hold onto the bus still and to perform another transfer (this is called **REPEAT** signal).

### Features 

1. Low speed communication
2. Half duplex
3. Synchronous (negedge of SCL) 
4. Two wire interface
5. Clock stretching: holds SCL line at zero until slave can accept/transfer data. Prevents Master from raising clock line due to AND wires. ( look into this in design )
6. Arbitration: I2C protocol supports multi-master bus system but more than one bus can not be used simultaneously. The SDA and SCL are monitored by the masters. If the SDA is found high when it was supposed to be low it will be inferred that another master is active and hence it stops the transfer of data.
7. Serial communication

### Advantages

1. less complexity due to only two bi-directional bus lines. 
2. multi master mode can be configured.
3. Cost efficient.
4. ACK/NACK can aid with improved error handling capability.

### Limitations

1. Slower Speed.
2. Half duplex.
<br>
<br>
#### Example of I2C applications
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/f38748d4-545c-4052-9d4a-68092181e0e5)

<br>
<br>

## In-Depth Understanding

Checking spec sheet by NXP  [I2C.pdf](https://github.com/Visruat/Comm_Protocol/files/13886952/I2C.pdf)

### Important Points

1. Open drain of outputs for devices is necessary for I2C protocol as multiple controllers trying to access the bus should not corrupt the data. Arbitration and Clock Synchronisation (both need AND-wired config) is configured to prevent corruption of data.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/7bf3ea92-7790-4c62-a9c2-d800220befda)

2. SDA and SCL can have multiple devices connected to them with different VDDs (device dependent eg 12V).
3. SDA and SCL logic levels are set for associated VDD where VIL = 0.3VDD and VIH = 0.7VDD.
4. Data is valid if SDA is stable when SCL is HIGH. SDA can change when SCL is LOW.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/a62506a3-cf56-4ce3-ad9d-ebaa702dfe05)

5. START, STOP and Repeated START generated when SCL is HIGH (START --> SDA = 1 to 0; STOP --> SDA = 0 to 1). The Byte Format is 8 bits followed by 1 bit acknowledgement

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/9b3e200c-95ef-459d-9ef3-8eff06cdd406)

Controller/target - transmitter releases control over SDA at 9th bit so that the target/controller - receiver can pull down SDA for ACK/NACK.  
6. **Clock Synchronisation**: Clock synchronization is performed using the wired-AND connection of I2C interfaces to the SCL line. This means that a HIGH to LOW transition on the SCL line causes the controllers concerned to start **counting off their LOW period** and, once a **controller** **clock** has **gone LOW**, it **holds the SCL line** in that **state** until the **clock HIGH** state is reached. When all controllers concerned have counted off their LOW period, the clock line is **released** and goes **HIGH**. There is then **no difference** between the **controller clocks** and the state of the **SCL line**, and all the controllers start **counting their HIGH periods**. The **first controller** to complete its **HIGH period** pulls the **SCL line LOW** again.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/2a950e27-fdea-4bc4-b46e-b22aaee2e553)

**A synchronized SCL clock is generated with its LOW period determined by the controller with the longest clock LOW period, and its HIGH period determined by the one with the shortest clock HIGH period.** <br>
7. **Arbitration**: This process in I2C is more like a **race** between the **transmitters** which initiated the **START signals** within that **THOLD(min)**. The SDA line is **monitored** by the **controller-transmitters** and its respective **data line** is **compared** whenever **SCL** is **HIGH**. If the data line **matches** the SDA line all **good**; If the data line **!= SDA line** it has lost the **arbitration**. No data is lost as the controller can continue to generate clk pulses for that BYTE and must restart the **transaction** when bus is free.
If a controller also incorporates a target function and it loses arbitration during the addressing stage, it is possible that the winning controller is trying to address it. The **losing controller** must therefore switch over immediately to its **target mode**.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/d1435d19-9e66-440f-b23f-f9eb8c50c025)


8. Clock Stretching occurs at two levels: Byte level: similar to Clock synchronisation--> clk1 faster than clk2 but SCL line held low so clk1 was thrown into a wait state until clk2 turned HIGH.
9. Target address and R/W bit: self explanatory refer diagram

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/4f5153c8-98c1-4a29-a9b1-7f3c65a38edc)

10. Some cases when it comes to target addressing and R/W bit and a combined format.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/f479ba4c-982a-4d97-8740-683796a8ee20)


11. 10-bit Address: Not widely used for wont be goin through is as such. Basic idea of it is that it works with S, Target address 1111 0XX, R/W,A1, Target address XXXX XXXX, A2, DATA , A .... ,A/Abar,P. To change direction R/W is altered by just sending --> 1111 0XX, R/W then data is sent.  

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/5283e5a2-1779-4f2e-8067-48b5cecdfb4a)

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/424edb8d-a9fb-4119-a563-b56275cea4bf)

12. General Call: It is used to address all the devices connected to the I2C bus. Devices capable of the handling the data can send an acknowledgement and change into a target receiver. The controller transmitter will not know how many devices have acknowledged the data. Those that cannot handle the data can leave the SDA line High. It again doesn't know if the how many NACK it received from the devices  

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/cf888de8-1db2-444b-8aae-2c4e9cd32e78)

based of LSB B in the second byte, there are 2 cases to consider: B --> 0 and B --> 1

0000 0110 --> software reset and write programmable part of target address by hardware.
0000 0100 --> write programmable part of target address by hardware.
0000 0000 --> not taken as 2nd byte.

XXXX XXX1 --> hardware general call; hardware controller device like keyboard scanner, sends desired target address. Intelligent device microcontroller picks up the info and starts communicating with the hardware controller device. <br>

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/e3b654eb-2b68-438e-b9f5-9714300cb5a1)

On software reset, the hardware controller transmitter is set to target receiver mode. A system configuring controller dumps the target address to the target receiver. After the programming procedure, it changes into a controller transmitter and begins to communicate with that device. 

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/2d8818a2-f5bd-45b7-92c7-b28cd80643a7)

13. Software reset: general call --> S,0000 0000,A,0000 0110,A ...
14. Start BYTE: **Unsure of its usage.** (check with shashank or sameer)
eg. After the START condition S has been transmitted by a controller which requires bus access, the START byte (0000 0001) is transmitted. Another microcontroller can therefore sample the SDA line at a low sampling rate until one of the seven zeros in the START byte is detected. After detection of this LOW level on the SDA line, the microcontroller can switch to a higher sampling rate to find the repeated START condition Sr which is then used for synchronization
15. Bus Clear: HW reset to clear the bus through HW reset input on device;  cycle power to the devices to activate the mandatory internal Power-On Reset (POR) circuit.
If the data line (SDA) is stuck LOW, the controller should send nine clock pulses. The device that held the bus LOW should release it sometime within those nine clocks. If not, then use the HW reset or cycle power to clear the bus.
16. Device ID: refer the PDF for details 

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/a689ea95-c5a6-48ed-bb08-adff04f1fbd2)

S, 1111 1000, A, target address, X, Sr, 1111 1001, DEVICE ID READ --> 8+4, 4+5, 3, NACK,P

if ACK instead of NACK the DEVICE ID READ repeats.  

### UFm for I2C
1. data rates can reach upto 5mb/s.
2. uni-directional flow of data (controller-transmitter to target receiver).
3. used for led controllers, lcd drivers which require > 1 mb/s data rate.
4. push-pull drivers instead of AND wired open drain lines.
5. Signal format of UFm I2C:

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/b2f5484e-b531-4116-91c1-702ec296cf30)

6. USDA and USCL; at 9th clock pulse (USCL), drives USDA to HIGH (always NACK). Wbar is called as command bit because its always "0" (uni-directional).
7. Target is not allowed to hold USCL LOW even if its servicing internal interrupts or unable to accept data at the moment. (no clock-stretching)
8. No device ID
9. other properties are similar to I2C Standard mode.  ( Refer PDF for images )

### Other uses of I2C bus communication protocols
1. CBUS compatability - 0000 001X CBUS addressing; Dlen replaces ACK/NACK for CBUS DATA; Common STOP Signal for all devices.    
2. System Management Bus (SMBus) - Dynamic unique address allocation (hot plug for all I2C devices connected to the bus)
3. I2C/SMBus Compliancy - Very similar working; 10kHz to 100kHz --> SMBus ; 0 - 100kHz, 100 - 400 kHz , 1MHz , 3.4 MHz based of the mode of I2C; Time out feature of SMBus will not for I2C devices between 0 to 10 kHz.
4. Power Management Bus System (PMBus): Power converter and System Host intelligent control.
5. Intelligent Platform Management Interface (IPMI) -  IPMI increases reliability of systems by monitoring parameters such as temperatures, voltages, fans and chassis intrusion.
6. Advanced Telecom Computing Architecture (ATCA)
7. Display Data Channel (DDC) - Display to inform its identity and capabilities.

## BUS Speeds
**Bidirectional bus** <br>
– Standard-mode (Sm), with a bit rate up to 100 kbit/s <br>
– Fast-mode (Fm), with a bit rate up to 400 kbit/s <br>
– Fast-mode Plus (Fm+), with a bit rate up to 1 Mbit/s <br>
– High-speed mode (Hs-mode), with a bit rate up to 3.4 Mbit/s <br>
**Unidirectional bus** <br>
– Ultra Fast-mode (UFm), with a bit rate up to 5 Mbit/s <br>

Note: All faster bidirectional I2C modes are downward compatible and hence can be used in mixed signal bus systems. This achieved by holding the SCL line low to stretch the clock period, bridges (to make hs and fm operate together) and pull up and pull down n/w ( current source / resistor )

### Fm: 
- SDA and SCL have spike suppression and Schmitt Trigger to prevent noise interference 
- Output buffers have slope control of the falling edges of SDA and SCL <!-- but why do we need slope control? -->
- Higher bus loads (>200 pF & <400 pF) needs pull up current source (3 mA max) else resistor is sufficient.

### Fm+:
- Stronger drive current is used; Hence can drive higher capacitance buses without using bus buffers.
- bus speed can be traded with load capacitance; max factor of 10.
- the high drive strength and tolerance for slow rise and fall times allow the use of larger bus capacitance as long as set-up, minimum LOW time and minimum HIGH time for Fast-mode Plus are all satisfied.
- The fall time and rise time do not exceed the 300 ns tf and 1 μs tr specifications of Standard-mode.
- SDA and SCL have spike suppression and Schmitt Trigger to prevent noise interference 
- Output buffers have slope control of the falling edges of SDA and SCL
  
Note: Sm,Fm,Fm+ use the same protocol, format and logic levels. Fm+ has different max capacitive loads.

### Hs:
- separate bus lines; SDAH and SCLH
- no arbitration and clock synchronization.
- Built in bridge to use in mixed bus signal systems.
- SDAH and SCLH have spike suppression and Schmitt Trigger to prevent noise interference 
- Output buffers have slope control of the falling edges of SDAH and SCLH.

#### Serial Data format for Hs mode: 
Initially operates in Fm. Until the following sequence is encountered.

- S
- 8 bit controller code ( 0000 1XXX )
- not ack

Enters Hs after ~A.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/9ca51d7d-0f52-479d-abb0-a6b1623179d4)

Once ~A is received T1. At Th the SCLH line is pulled HIGH to enable switches and enables for Hs mode. Current source pull up (MCS) circuit  is used for shorter rise time. SCLH is pulled HIGH only after all devices releases the SCLH line to prevent slowing down the transition. Sr bit is sent and regular I2C comms is initiated now.

<br>
After every acknowledgement bit, MCS circuit is disengaged and re-enagaged only after the SCLH line is released byu all devices. It operates in Hs mode until P is encountered after which it switches back to Fm mode. 

<br>
<br>
Note: To reduce overhead  of the controller code, it is possible that a controller links a number of Hs-mode transfers, separated by repeated START conditions (Sr). 

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/df8cf6ef-abc5-4699-b001-9864f8b7664c)

#### Switching from Fm to Hs

The active (winning) controller:

1. Adapts its SDAH and SCLH input filters according to the spike suppression requirement in Hs-mode.
2. Adapts the set-up and hold times according to the Hs-mode requirements.
3. Adapts the slope control of its SDAH and SCLH output stages according to the Hsmode requirement.
4. Switches to the Hs-mode bit-rate, which is required after time tH.
5. Enables the current source pull-up circuit of its SCLH output stage at time tH.

The non-active, or losing controllers:

1. Adapt their SDAH and SCLH input filters according to the spike suppression requirement in Hs-mode.
2. Wait for a STOP condition to detect when the bus is free again.

All targets:

1. Adapt their SDAH and SCLH input filters according to the spike suppression requirement in Hs-mode.
2. Adapt the set-up and hold times according to the Hs-mode requirements. This requirement may already be fulfilled by the adaptation of the input filters.
3. Adapt the slope control of their SDAH output stages, if necessary. For target devices, slope control is applicable for the SDAH output stage only and, depending on circuit tolerances, both the Fast-mode and Hs-mode requirements may be fulfilled without switching its internal circuit.

Note: At lower speeds, MCS is disabled and SDAH and SCLH lines are compatible with SDA and SCL.

### Mixed Bus Signal Systems
A bridge is present to link the SCL, SDA (VDD2 = 5V; should be 5V tolerant) and SCLH, SDAH ( VDD1 = 3V ). Bridge --> TR1, TR2 (transfer gate function)  AND TR3 (open drain pull down MOSFET) are N-channel MOSFETS.   
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/7fc025da-1c4f-4097-807d-3c355956517c)

Bridge functioning after controller code is read:
The controller code is recognized by the bridge in the active or non-active controller. 
The bridge performs the following actions:
1. Between t1 and tH, transistor TR1 opens to separate the SDAH and SDA lines, after which transistor TR3 closes to pull-down the SDA line to VSS.
2. When both SCLH and SCL become HIGH, transistor TR2 opens to separate the SCLH and SCL lines. TR2 must be opened before SCLH goes LOW after Sr.

Note: In irregular situations, F/S-mode devices can close the bridge (TR1 and TR2 closed, TR3 open) at any time by pulling down the SCL line for at least 1 μs, for example, to recover from a bus hang-up.

Bridge Functioning after STOP is read:
1. Transistor TR2 closes after tFS to connect SCLH with SCL; both of which are HIGH at this time. Transistor TR3 opens after tFS, which releases the SDA line and allows it to be pulled HIGH by the pull-up resistor Rp. This is the STOP condition for the F/S mode devices. TR3 must open fast enough to ensure the bus free time between the STOP condition and the earliest next START condition is according to the Fast-mode specification.
2. When SDA reaches a HIGH, transistor TR1 closes to connect SDAH with SDA. (Note: interconnections are made when all lines are HIGH, thus preventing spikes on the bus lines.) TR1 and TR2 must be closed within the minimum bus free time according to the Fast-mode specification.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/d6434d4b-1e9f-49a3-8a19-b9a1718cc45c)

##  Electrical specifications and timing for I/O stages and bus lines

#### Characteristics of SDA and SCL I/0 stage 
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/3ac5df2d-6a2f-4f34-9f68-2390f7ec4a26)
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/691fd650-0e93-4158-b958-b7b0b2021ff6)

Key Points: <br>
- Vhys = Vut - Vlt
- low level output voltage has two cases 
  - Vol1 --> 3mA sink current; Vdd > 2V
  - Vol2 --> 2mA sink current; Vdd <= 2V (fm,fm+)
- low level output current has two cases
  - Vol = 0.4 (0.2Vdd; Vdd = 1.8V)
  - Vol = 0.6 (0.2Vdd; Vdd = 3.3V)
- tSP filter any noise that occurs within 50ns
- High level input voltage
  - Vih = Vdd(max) + 0.5V; <5.5V
  - Vih = 5.5V
- fall time (tf) = 300ns ; output fall time ( Vih(min) - Vil(max) ) = 250ns; series protection resistor can be used provided tof > tf <br>
<br>

#### Characteristics of SDA and SCL bus lines for Sm,Fm,FM+
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/32d86ad4-0dc2-4daf-a250-c4ad1b2560c5)
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/61614d0d-4592-4786-ba26-b86a73dd0f17)

**Key Points:** <br>
- tHIGH : tLOW = 2:1 (fm,fm+)
- SCL must be low (<0.3Vdd) before SDA begins to change (0.3Vdd < SDA <0.7Vdd)
- if tLOW of clock is stretched by a device, tVD;DAT(max) >= tSU;DAT(min) (data should be valid by the setup time when the device releases the clock)
- bus capacitance value may vary depending on the actual operating voltage and frequency of application.
<br>

**Waveform desciption:** <br>
- thold for start --> time between SDA H-L (30) and SCL H-L (70)
- tsetup for start --> time between SCL L-H (70) and SDA H-L (70)
- tsetup for stop --> time between SCL L-H (70) and SDA L-H (30)
- tbuf --> Bus free time between stop and start
- thold for data --> time between SCL H-L (30) and SDA (start: 70 , 30)
- tsetup for data --> time between SDA (end: 30 , 70) and  SCL L-H (30)

<br>
Note: To avoid confusion, according to definition Tsetup --> signal remains stable before control signal edge (control edge = SCL L-H). Thold --> signal remains stable after control signal edge (control edge = SCL H-L). This is because SDA is not allowed to change when SCL is HIGH (level triggered type of control signal)
<br>
<br>

- tvalid for data --> time between SCL H-L (30) till SDA (worse: 30 , 70)
- tvalid for acknowledgement --> time between SCL H-L (30) till SDA (worse: 30 , 70) (9th clock cycle).
- tsp --> supress noise which occur under 50ns time frame.

#### Characteristics of SCLH, SDAH, SDA and SCL I/0 stage for Hs mode
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/24e4453b-b003-476b-b724-bf7e32658310)
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/e2bf5e9d-8197-4df3-898d-0333f6eb0f37)

#### Characteristics of SDA and SCL bus lines for Hs mode
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/8673740d-3e85-4236-9483-6f0ee00c1484)
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/26fb9db8-dfc8-4676-9fd9-c23c4d3ed088)
<br>
<br>

**Waveform desciption:** <br>
most are similar to previous case.
trCL --> rise time for SCLH 
trCL1 --> rise time SCLH at first cycle (Rp pull up)
tfCL --> fall time for SCLH
trDA --> rise time for SDAH
tfDA --> fall time for SDAH
<br>

#### Characteristics of the USDA and USCL I/O stages
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/ddf8a8be-e0f3-4761-845d-c4e5de193739)

#### UFm I2C-bus frequency and timing specifications
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/24c05760-4851-41f9-a0c1-155b74322db4)
![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/6df10f0e-6a1c-4d07-8e50-17c72543ffbb)

### References
1. [GFG](https://www.geeksforgeeks.org/i2c-communication-protocol/)
2. [Basics of I2C communication](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)\
3. [I2C.pdf](https://github.com/Visruat/Comm_Protocol/files/13887643/I2C.pdf)

