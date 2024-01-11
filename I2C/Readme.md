## Images --> for review of concept and terms 
1. Push pull drive
2. Noise Margin

# I2C Protocol

## Basic Understanding

1. Bus interface connection protocol.
2. SDA (serial data line --> data line (half duplex) ) and SCL (serial clock line --> control signal for I2C)
3. 9-bit protocol. 8 for address and 1 reserved for ACK/NACK bit i.e Acknowledgment and Not Acknowledge.
4. SDA and SCL bus lines are Open Drain; connected with pullup resistors as devices are active low.

[Open-Drain](https://www.analog.com/en/design-center/glossary/open-drain-collector.html) mode is for FET wherein the drain connected to the output is pulled to High in OFF state and reflect the required output in ON state

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

Controller - transmitter releases control over SDA at 9th bit so that the target/controller - receiver can pull down SDA for ACK/NACK.  
6. **Clock Synchronisation**: Clock synchronization is performed using the wired-AND connection of I2C interfaces to the SCL line. This means that a HIGH to LOW transition on the SCL line causes the controllers concerned to start **counting off their LOW period** and, once a **controller** **clock** has **gone LOW**, it **holds the SCL line** in that **state** until the **clock HIGH** state is reached. When all controllers concerned have counted off their LOW period, the clock line is **released** and goes **HIGH**. There is then **no difference** between the **controller clocks** and the state of the **SCL line**, and all the controllers start **counting their HIGH periods**. The **first controller** to complete its **HIGH period** pulls the **SCL line LOW** again.

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/2a950e27-fdea-4bc4-b46e-b22aaee2e553)

**A synchronized SCL clock is generated with its LOW period determined by the controller with the longest clock LOW period, and its HIGH period determined by the one with the shortest clock HIGH period.** <br>
7. **Arbitration**: This process in I2C is more like a **race** between the **transmitters** which initiated the **START signals** during that **THOLD(min)**. The SDA line is **monitored** by the **controller-transmitters** and its respective **data line** is **compared** whenever **SCL** is **HIGH**. If the data line **matches** the SDA line all **good**; If the data line **!= SDA line** it has lost the **arbitration**. No data is lost as the controller can continue to generate clk pulses for that BYTE and must restart the **transaction** when bus is free.
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
14. Start BYTE: explore later
15. Bus Clear: HW reset to clear the bus through HW reset input on device;  cycle power to the devices to activate the mandatory internal Power-On Reset (POR) circuit.
If the data line (SDA) is stuck LOW, the controller should send nine clock pulses. The device that held the bus LOW should release it sometime within those nine clocks. If not, then use the HW reset or cycle power to clear the bus.
16. Device ID: refer the PDF for details 

![image](https://github.com/Visruat/Comm_Protocol/assets/125136551/a689ea95-c5a6-48ed-bb08-adff04f1fbd2)

S, 1111 1000, A, target address, X, Sr, 1111 1001, DEVICE ID READ --> 8+4, 4+5, 3, NACK,P

if ACK instead of NACK the DEVICE ID READ repeats.  
17. f
18. f
19. f
20. 
21. 


### References
1. [GFG](https://www.geeksforgeeks.org/i2c-communication-protocol/)
2. [Basics of I2C communication](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)\
3. [I2C.pdf](https://github.com/Visruat/Comm_Protocol/files/13887643/I2C.pdf)
