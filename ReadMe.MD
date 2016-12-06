# typeperf
Writes performance counter data to the command window, or to a supported log file format. To stop Typeperf, press CTRL+C.

Great code is all alike; every unhappy code block is unhappy in its own way.

Also, the less code you write the less bugs there are to fix.
And stored procedures take more code to perform the same set of actions.


See counters.txt and counters_qx.txt for available counters or:

```
typeperf -q

typeperf -qx

```

If you run any of these examples below make sure calculator is running or just change the "calculator" process name to your process which
you want to monitor.

# Server Level Monitoring
High CPU Utilization indicates application is performing tasks inefficiently or running computationally intensive tasks.  An application
which maintains CPU Utilization above 90% is considered high.  

High memory utilization indicates that an application is not using efficiently, or is perhaps not releasing resources appropriately.  There are
obviously instances  where using a lot of memory is required, but not releasing that memory as soon as possible is a serious issue, and this
is how memory leaks can manifest (An application memory keeps growing over time and never releases).



Monitor with
* % Processor Time  
* % Committed Bytes In Use  
* Available MBytes  

```txt

typeperf "\Processor(_total)\% Processor Time" "\Memory\Committed Bytes" "\Memory\Available Bytes"

```

```txt

typeperf "\Processor(_total)\% Processor Time" "\Memory\Committed Bytes" "\Memory\Available Bytes" -o systemPerf.csv

```
writes output to a file...

# Task Manager Examples

## % CPU
Total CPU of machine.  

CPU utilization for a database server should be 20-30%  Anything over the database is doing too much work.

CPU utilization for an application server around 20-30% is excellent, 50-70% is well utilized.  
Anything over 75% indicates the system is reaching computational capacity.

```txt

typeperf "Processor(_Total)\% Processor Time"

```
% Processor Time shows the total percentage of processor utilization across all processes.

## Memory

The total available memory should remain above 25% to allow the system to have headroom to operate efficiently.

At best when memory peaks, paging will occur, where memory will be read and written from disk, whch means s-l-o-w.

In .Net memory should follow the saw tooth pattern.

```txt

typeperf "\Memory\Available MBytes" "\Memory\Available KBytes" "\Memory\Available Bytes"

```

# Resource Manager - Overview
-- PID , Description, Status, Threads, CPU, Average CPU  
TODO

# Resource Manager - Disk
TODO

# Resource Manager - Memory
* PID = "\Process(calculator)\ID Process"                  	REVIEWED  
* Hard Faults / Sec = ? "\Memory\Pages/sec" ?			          Unknown(needs a filter for the process)
* Commit =  "\Process(calculator)\Private Bytes"  		      REVIEWED (but Maybe)  
* Shareable????                                             Unknown...
* Working Set = "\Process(Calculator)\Working Set"         	REVIEWED  
* Private =  "\Process(calculator)\Working Set - Private"  	REVIEWED  

Example to get memory of the calculator

```txt

typeperf "\Process(calculator)\ID Process" "\Memory\Pages/sec" "\Process(calculator)\Private Bytes" "\Process(Calculator)\Working Set" "\Process(calculator)\Working Set - Private"   

```

## Additional Use Cases
Check your stats every 5 seconds:
```txt

typeperf "\Process(calculator)\ID Process" "\Process(Calculator)\Working Set" "\Process(calculator)\Working Set - Private" -si 5

```

Write stats to a file
```txt

typeperf "\Process(calculator)\ID Process" "\Process(Calculator)\Working Set" "\Process(calculator)\Working Set - Private" -o calculator.csv

```

# Disk Speed for Writing Files

# Web Application Metrics

# Database / SQL Server Counters

"The most efficient query is the one which never has to run"

CPU should be as low as possible, and not exceed 50-60%
Memory should be as low as possible, ideally averaging below 50%.

* Deadlocks
* transactions
* Disk I/O

## Category: Physical Disk
### PhysicalDisk(*)\Avg. Disk Queue Length**

Very important for database an applications. Counter is the average number of read and write requests that have
been queued and are yet to be fulfilled.  As the number of simultaneous requests for data from the disk increases
the disk subsystem can become overloaded and unable to fulfill the requests as fast as required.  If the queue
continues to grow the database is having performance problems.

Fewer requests, more efficient requests, and faster disks help improve this problem.

```txt

typeperf "\PhysicalDisk(*)\Avg. Disk Queue Length"

```

### Drive(s) experiencing I/O stall –> Avg. Disk sec/Transfer
Use Case from Cleveland
SQL Server has encountered 1 occurrence(s) of I/O requests taking longer than 15 seconds to complete on file [E:\MSSQL10_50.CLV\Log\ElsterStaging.ldf] in database [ElsterStaging] (10). The OS file handle is 0x00000000000009B0. The offset of the latest long I/O is: 0x000000012caa00

E drive has 460 GB free space out of 526 GB

Need to run perf.counter
Logical Disk –> Drive(s) experiencing I/O stall –> Avg. Disk sec/Transfer

Average Disk Sec/Transfer on a SQL Server machine is typically less than 15ms. If this value climbs, it may indicate that the I/O subsystem is not keeping up with I/O demand. (Source: 833 Error Details). And if you want to gauge I/O perfectly, here are the values for reference:

Any number Less than 10 ms – very good
Between 10 – 20 ms – okay
Between 20 – 50 ms – slow, needs attention
Greater than 50 ms – Serious I/O bottleneck (Source: MSDN)

```txt

typeperf "\PhysicalDisk(*)\Avg. Disk Queue Length"

```

## SQL Server: SQL Statistics
\SQLServer:SQL Statistics\Batch Requests/sec

1000+ requests per second can indicate potential issues and that the server may soon begin
to experience stress.

```txt

typeperf "\SQLServer:SQL Statistics\Batch Requests/sec"

```

## SQLServer:Databases(*)\Transactions/sec

Number of transactions the service is processing per second, and can be
thought of as a sub measurement of the previously discussed Batch Requests/sec.

Transactions are expensive, and a high relative value may indicate a need to evaluate
the isolation level and transaction policy of the application.


```txt

typeperf "\SQLServer:Databases(*)\Transactions/sec"

```

### Additional ones
\Process(sqlservr)\% Privileged Time
\Process(sqlservr)\% Processor Time
\Process(sqlservr)\% User Time
\Process(sqlservr)\IO Data Operations/sec
\Process(sqlservr)\Page Faults/sec
\Process(sqlservr)\Private Bytes
\Process(sqlservr)\Thread Count
\Process(sqlservr)\Virtual Bytes
\Process(sqlservr)\Working Set
\SQLServer:Buffer Manager\*
\SQLServer:Cache Manager(*)\*
\SQLServer:Databases(*)\Data File(s) Size (KB)
\SQLServer:Databases(*)\Log File(s) Size (KB)
\SQLServer:Databases(*)\Transactions/sec
\SQLServer:General Statistics\*
\SQLServer:Locks(*)\*
\SQLServer:Memory Manager\*
\SQLServer:SQL Statistics\Batch Requests/sec
\SQLServer:SQL Statistics\SQL Compilations/sec
\SQLServer:SQL Statistics\SQL Re-Compilations/sec

## BrentOzar The Best SQL Server Performance Monitor Counters to Analyze

These are listed OBJECT first, then COUNTER
Memory – Available MBytes
Physical Disk – Avg. Disk sec/Read
Physical Disk – Avg. Disk sec/Write
Physical Disk – Disk Reads/sec
Physical Disk – Disk Writes/sec
Processor – % Processor Time
SQLServer: General Statistics – User Connections
SQLServer: Memory Manager – Memory Grants Pending
SQLServer: SQL Statistics – Batch Requests/sec
SQLServer: SQL Statistics – Compilations/sec
SQLServer: SQL Statistics – Recompilations/sec
System – Processor Queue Length
After adding those, click the Close button, and we’re back to the counter log setup window. Under “Sample data every”, the default should be 15 seconds, and that’s fine. In the “Run As” entry, type in your domain username in the form domainname\username, and click the Set Password button to save your password. This lets the Perfmon service gather statistics using your domain permissions – otherwise, it tries to use its own credentials, and they probably won’t work on the remote server.

# Additional Examples

```txt

typeperf  "\Process(calculator)\% processor time" "\Process(calculator)\Private Bytes" "Process(Calculator)\Working Set"

typeperf  "\PhysicalDisk(Calculator)\Disk Read Bytes/sec" "\PhysicalDisk(Calculator)\Disk Write Bytes/sec"

```

# Make Sense of Memory Management and Key Memory Measurements

Memory management is confusing, perhaps more so than any other aspect of PC performance. If you scour the Web
for information on this topic, you will surely run into misguided advice and technical errors. Knowing the meaning
of the following specialized memory-measurement terms helps you make sense of it all:

Physical memory refers to actual RAM chips or modules, typically installed on a computer�s motherboard.
The amount of physical RAM available to Windows might be less than the total physical amount if another
system component is using that memory for its own purposes, as is the case with �shared memory� video
subsystems on portable computers. Physical memory measurements (total and in use) are reported on the
Performance tab of Windows Task Manager and on the Memory tab of Resource Monitor.

Virtual memory consists of physical memory plus the amount of space in the page file, which is stored on the hard disk.

Kernel memory is owned by Windows and is used to provide system services to applications.

Paged memory can be backed up to the page file and replaced by application memory if necessary. Nonpaged memory must remain in physical RAM at all times.

Cached memory holds data or program code that has been fetched into memory during the current session but is no longer in use now.
If necessary, the Windows memory manager will flush the contents of cached memory to make room for newly summoned data.

Free memory represents RAM that does not contain any data or program code and is free for use immediately.

Working Set is the term that defines the amount of memory currently in use for a process. Private Working Set is the amount
of memory that is dedicated to that process and will not be given up for other programs to use; Shareable Working Set can
be surrendered if physical RAM begins to run scarce. Peak Working Set is the highest value recorded for the current instance of this process.

Commit Charge (also called commit size) is the total amount of virtual memory that a program has touched (committed) in the current session,
including memory that has been paged out of physical memory to the disk-backed page file.
The Memory and Physical Memory counters on **Task Manager�s Performance** tab represent the sum of this value for all processes and the kernel.
The Commit Charge Limit is the total amount of physical RAM and page file available�in other words, the maximum virtual memory.

Memory\Available Bytes reports available bytes; its value tends to fall during a memory leak.
Memory\Committed Bytes reports the private bytes committed to processes; its value tends to rise during a memory leak.
Process( process_name )\Private Bytes reports bytes allocated exclusively for a specific process; its value tends to rise for a leaking process.
Process( process_name )\Working Set reports the shared and private bytes allocated to a process; its value tends to rise for a leaking process.
Process( process_name )\Page Faults/sec reports the total number of faults (hard and soft faults) caused by a process; its value tends to rise for a leaking process.
Process( process_name )\Page File Bytes reports the size of the paging file; its value tends to rise during a memory leak.
Process( process_name )\Handle Count reports the number of handles that an application opened for objects it creates. Handles are used by programs to identify resources that they must access. The value of this counter tends to rise during a memory leak; however, you cannot rule out a leak simply because this counters value is stable

"\Process(calculator)\% Privileged Time"
"\Process(calculator)\% Processor Time"
"\Process(calculator)\% User Time"
"\Process(calculator)\IO Data Operations/sec"
"\Process(calculator)\Page Faults/sec"


# References:
https://technet.microsoft.com/en-us/library/bb490960.aspx
http://ss64.com/nt/syntax-performance-counters.html
https://technet.microsoft.com/en-us/library/cc958287.aspx
https://www.brentozar.com/archive/2006/12/dba-101-using-perfmon-for-sql-performance-tuning/
