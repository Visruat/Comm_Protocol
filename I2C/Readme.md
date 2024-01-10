# I2C Protocol

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

### Modes ( come back to it later )

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

### References
1. [GFG](https://www.geeksforgeeks.org/i2c-communication-protocol/)
2. [Basics of I2C communication](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)\
3. 
