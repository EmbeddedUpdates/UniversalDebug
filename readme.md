# Universal Debugging Module

## Core Features
1. Formatted Printing (printf-like API)
    1. Support Strings, Integers, Hex, floats
    2. Example: UniversalDebug_Print("Speed: %d km/h\n", speed);
2. Multiple output transports
    1. Transnmit debugging information over various, configurable outputs.
    2. E.g. UART, CAN, RTT, Syslog (long term goal)
    3. Telnet, MQTT for wireless debugging
3. Log levels and Filtering
    1. Supports varying debug levels, which can be enabled or disabled at runtime
    2. INFO, WARNING, ERROR, DEBUG
    3. Exampl: UniversalDebug_Log(DEBUG_WARN, "Low battery warning: %d%%", battery_level);
4. Timestamps
    1. Adds timestamps (ms/us resolution) for event tracking
    2. Could interface with RTOS or direct to hardware timer
5. Non-blocking for transmit
    1. Use RingBuffer to allow for non-blocking transmit of new requests.
6. Memory Dump
7. Peripheral Dump
8. Debug Commands
    1. Change Debug level
    2. Get System stats/usage metrics

## Example API design:
```
// Initialize UniversalDebug
UniversalDebug_Init(DEBUG_UART, DEBUG_LEVEL_INFO);

// Print formatted message
UniversalDebug_Print("System Booted! Version: %s", FIRMWARE_VERSION);

// Log with levels
UniversalDebug_Log(DEBUG_ERROR, "Sensor Failure: Code %d", error_code);

// Change log transport
UniversalDebug_SetOutput(DEBUG_RTT);

// Memory dump
UniversalDebug_DumpMemory(buffer, sizeof(buffer));
```
So, at a simple point, we also have to receive and process commands.
We also want to have a transmit function, esp since Print/Log/will all transmit similarly, although possible to different paths.
Should logs go to a differnet space? FLashMem? Print vs Log?

So, we can start with management of a few local variables in UniDebug_asd(), UniversalDebug_asd(), UD_asd()
UniversalDebug_Init().



For things like RingBuffer, we have two challenges. RX interrupt to this task should
only be triggered when we have processed the data up to app level. But Tx data should be triggered when we want to transmit on this level.
It would be advantagous if we can not copy the data all over the place.
I.e. From UniDebug to CanTp to CanIf to CAN. we should not prepare four messages.
So it would be a good idea if we can retreive a slot of memory that data would be located in the ringbuffer.
For that purpose - it would be good to know how big the total data could be:
i.e. TOTAL packet at app level + overhead from TP + overhead from IF + overhead of physical layer and so on.
That should not be handled in the app layer, but handled by the PDU router (whatever that is called).
Then the router can query each of the stacks/modules for sizes of data, for either path we have a couple of things to think about. 
Tx - out of the module, we must provide buffer pointer to somewhere inside the mailbox. (_________ABCDSEFGHIJKLMNOPQRSTUVWXYZ_______)
I.e. the highest layer gets a subset of the sub-data array.
Counterpoint - what about segmenting data and the overhead that must go between the segments? I.e. CANTP? If the original app data was
ABCDEF....YZ, then we have to segment that A, B, C, D, E, and the overhead should be inserted in this array, that would not work.
So the RB that pdur has should do this parsing on its own? That is a lot of calls to PDUR. Lots of stack usage. Lots of jmps

So what if each module has its own buffer? RAM usage /^ but simpler design - better start.
Consider a sharedComBuffer sometime in the future...



