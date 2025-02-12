# Tests for UniversalDebug module

## Should this be object oriented?

No. There's no good reason for that here.
Why would we ever support multiple debuggers?

## Public interfaces

1. Init
2. Transmit (for usage by pdu router?)
3. Receive (for usage by pdu router)

4. Commands APIs should exist for public interface from other modules.
    1. Diagnostics and SelfTest
        1. Status() - Reports current status of the debug module
        2. SelfTest()
        3. ErrorHistory()
    2. Configuration()
        1. Config_LogLevel()
        2. Config_BenchmarkOutputFormat() - Plain Text, JSON, Binary
        3. Config_Timestamps()
        4. Config_Buffer(size, window)
        5. 
    3. Logging to NVM
        1. Log() - Store data locally for retreival later.
        2. Erase() - Clears the debugging log stored in NVM
    4. Printing to comm output
        1. Print()
        2. SetOutput()
        3. Dump()
        4. StreamLogs(Interval, format) - stream active log information also to active output
            1. Format? Could be too much overhead. Could target efficient transmit instead (i.e. binary)
        5. ExportLogs(format) from Logs in NVM out to active output
        6. Benchmark()

            ```text
            [Benchmark Report]
            -------------------------------
            Timestamp: 2025-02-12 14:35:20
            Active Output: UART @ 115200 baud
            Fallback Order: UART → CAN → Ethernet
            -------------------------------
            Execution Time:
            - Log() Avg: 150 µs
            - Print() Avg: 200 µs
            - Read() Avg: 300 µs
            - Dump() Avg: 1.5 ms
            -------------------------------
            Memory Usage:
            - NVM Log Space: 65% used (130 KB / 200 KB)
            - RAM Buffer: 40% used (8 KB / 20 KB)
            -------------------------------
            Throughput:
            - Max Print Rate: 50 messages/sec
            - Avg Log Storage Rate: 30 KB/sec
            -------------------------------
            System Load:
            - CPU Usage: 12% (Debug Module)
            - Peak Latency: 2.1 ms (Print to UART)
            -------------------------------
            Status: OK (No errors detected)
            ```


## Private Interfaces

1. ProcessCmd() - branch to specific function handling based on RX'd command
2. updateRuntime(key, newVal) - updates funcs average, min, and max runtime given new data.
3. retrive_cpu_load( who knows how to do this. )
4. 

