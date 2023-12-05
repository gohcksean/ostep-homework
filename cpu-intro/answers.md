## Questions

1.  Run process-run.py with the following flags: -l 5:100,5:100.
    What should the CPU utilization be (e.g., the percent of time the
    CPU is in use?) Why do you know this? Use the -c and -p flags to
    see if you were right.
    `The CPU utilization should be 100% of the time as there is no blocking IO.`

    ```
    Time        PID: 0        PID: 1           CPU           IOs
    1        RUN:cpu         READY             1
    2        RUN:cpu         READY             1
    3        RUN:cpu         READY             1
    4        RUN:cpu         READY             1
    5        RUN:cpu         READY             1
    6           DONE       RUN:cpu             1
    7           DONE       RUN:cpu             1
    8           DONE       RUN:cpu             1
    9           DONE       RUN:cpu             1
    10           DONE       RUN:cpu             1

    Stats: Total Time 10
    Stats: CPU Busy 10 (100.00%)
    Stats: IO Busy  0 (0.00%)
    ```

2.  Now run with these flags: ./process-run.py -l 4:100,1:0.
    These flags specify one process with 4 instructions (all to use the
    CPU), and one that simply issues an I/O and waits for it to be done.
    How long does it take to complete both processes? Use -c and -p
    to find out if you were right.
    `The CPU should run for 4 time units for the first process, 1 time for io run, 5 time units being blocked, 1 time for io end. Total of 11 time units`

    ```
     Time        PID: 0        PID: 1           CPU           IOs
     1        RUN:cpu         READY             1
     2        RUN:cpu         READY             1
     3        RUN:cpu         READY             1
     4        RUN:cpu         READY             1
     5           DONE        RUN:io             1
     6           DONE       BLOCKED                           1
     7           DONE       BLOCKED                           1
     8           DONE       BLOCKED                           1
     9           DONE       BLOCKED                           1
     10           DONE       BLOCKED                           1
     11*          DONE   RUN:io_done             1

     Stats: Total Time 11
     Stats: CPU Busy 6 (54.55%)
     Stats: IO Busy  5 (45.45%)
    ```

3.  Switch the order of the processes: -l 1:0,4:100. What happens
    now? Does switching the order matter? Why? (As always, use -c
    and -p to see if you were right)
    `The order matters as process PID:1 can run when the process PID:0 is blocked due to IO`

    ```
     Time        PID: 0        PID: 1           CPU           IOs
     1         RUN:io         READY             1
     2        BLOCKED       RUN:cpu             1             1
     3        BLOCKED       RUN:cpu             1             1
     4        BLOCKED       RUN:cpu             1             1
     5        BLOCKED       RUN:cpu             1             1
     6        BLOCKED          DONE                           1
     7*   RUN:io_done          DONE             1

     Stats: Total Time 7
     Stats: CPU Busy 6 (85.71%)
     Stats: IO Busy  5 (71.43%)
    ```

4.  We’ll now explore some of the other flags. One important flag is -S, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH ON END, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (-l 1:0,4:100 -c -S SWITCH ON END), one doing I/O and the other doing CPU work?
    `The proces PID:0 will take 1 unit to run io, 5 units in blocked, during which PID:1 cannot start running, PID:0 will take 1 unit to run end io, after which point pid can start running for 4. It will take 11 units of time to run, of which, CPU is only busy for 6 units of time.`

    ```
    Time        PID: 0        PID: 1           CPU           IOs
    1         RUN:io         READY             1
    2        BLOCKED         READY                           1
    3        BLOCKED         READY                           1
    4        BLOCKED         READY                           1
    5        BLOCKED         READY                           1
    6        BLOCKED         READY                           1
    7*   RUN:io_done         READY             1
    8           DONE       RUN:cpu             1
    9           DONE       RUN:cpu             1
    10          DONE       RUN:cpu             1
    11           DONE       RUN:cpu             1

    Stats: Total Time 11
    Stats: CPU Busy 6 (54.55%)
    Stats: IO Busy  5 (45.45%)
    ```

5.  Now, run the same processes, but with the switching behavior set
    to switch to another process whenever one is WAITING for I/O (-l
    1:0,4:100 -c -S SWITCH ON IO). What happens now? Use -c
    and -p to confirm that you are right.
    `The pid:1 can run when pid:0 is blocked, this will reduce total time taken to 7, and busy cpu time is still 6, io busy time will be 2`

    ```
    Time        PID: 0        PID: 1           CPU           IOs
    1         RUN:io         READY             1
    2        BLOCKED       RUN:cpu             1             1
    3        BLOCKED       RUN:cpu             1             1
    4        BLOCKED       RUN:cpu             1             1
    5        BLOCKED       RUN:cpu             1             1
    6        BLOCKED          DONE                           1
    7*   RUN:io_done          DONE             1

    Stats: Total Time 7
    Stats: CPU Busy 6 (85.71%)
    Stats: IO Busy  5 (71.43%)
    ```

6.  One other important behavior is what to do when an I/O completes. With -I IO RUN LATER, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when
    you run this combination of processes? (./process-run.py -l
    3:0,5:100,5:100,5:100 -S SWITCH ON IO -c -p -I
    IO RUN LATER) Are system resources being effectively utilized?
    `The pid:0 with 3 IO can run in between the other processes which do not have IO calls, the other processes can then run while pid:0 is blocked by IO. But this is not happening with the IO RUN LATER option specified, reducing the ideal cpu utilization.`

    ```
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
    1         RUN:io         READY         READY         READY             1
    2        BLOCKED       RUN:cpu         READY         READY             1             1
    3        BLOCKED       RUN:cpu         READY         READY             1             1
    4        BLOCKED       RUN:cpu         READY         READY             1             1
    5        BLOCKED       RUN:cpu         READY         READY             1             1
    6        BLOCKED       RUN:cpu         READY         READY             1             1
    7*         READY          DONE       RUN:cpu         READY             1
    8          READY          DONE       RUN:cpu         READY             1
    9          READY          DONE       RUN:cpu         READY             1
    10          READY          DONE       RUN:cpu         READY             1
    11          READY          DONE       RUN:cpu         READY             1
    12          READY          DONE          DONE       RUN:cpu             1
    13          READY          DONE          DONE       RUN:cpu             1
    14          READY          DONE          DONE       RUN:cpu             1
    15          READY          DONE          DONE       RUN:cpu             1
    16          READY          DONE          DONE       RUN:cpu             1
    17    RUN:io_done          DONE          DONE          DONE             1
    18         RUN:io          DONE          DONE          DONE             1
    19        BLOCKED          DONE          DONE          DONE                           1
    20        BLOCKED          DONE          DONE          DONE                           1
    21        BLOCKED          DONE          DONE          DONE                           1
    22        BLOCKED          DONE          DONE          DONE                           1
    23        BLOCKED          DONE          DONE          DONE                           1
    24*   RUN:io_done          DONE          DONE          DONE             1
    25         RUN:io          DONE          DONE          DONE             1
    26        BLOCKED          DONE          DONE          DONE                           1
    27        BLOCKED          DONE          DONE          DONE                           1
    28        BLOCKED          DONE          DONE          DONE                           1
    29        BLOCKED          DONE          DONE          DONE                           1
    30        BLOCKED          DONE          DONE          DONE                           1
    31*   RUN:io_done          DONE          DONE          DONE             1

    Stats: Total Time 31
    Stats: CPU Busy 21 (67.74%)
    Stats: IO Busy  15 (48.39%)
    ```

7.  Now run the same processes, but with -I IO RUN IMMEDIATE set,
    which immediately runs the process that issued the I/O. How does
    this behavior differ? Why might running a process that just completed an I/O again be a good idea?
    `CPU utilization increased to 100% as the other processes can run when pid:0 is blocked by IO. It might be a good idea to run a process that just completed I/O as there may we might expect such processes to have more I/O calls, we can then issue the I/O calls first and run the other processes when the current process gets blocked by I/O!`

    ```
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
    1         RUN:io         READY         READY         READY             1
    2        BLOCKED       RUN:cpu         READY         READY             1             1
    3        BLOCKED       RUN:cpu         READY         READY             1             1
    4        BLOCKED       RUN:cpu         READY         READY             1             1
    5        BLOCKED       RUN:cpu         READY         READY             1             1
    6        BLOCKED       RUN:cpu         READY         READY             1             1
    7*   RUN:io_done          DONE         READY         READY             1
    8         RUN:io          DONE         READY         READY             1
    9        BLOCKED          DONE       RUN:cpu         READY             1             1
    10        BLOCKED          DONE       RUN:cpu         READY             1             1
    11        BLOCKED          DONE       RUN:cpu         READY             1             1
    12        BLOCKED          DONE       RUN:cpu         READY             1             1
    13        BLOCKED          DONE       RUN:cpu         READY             1             1
    14*   RUN:io_done          DONE          DONE         READY             1
    15         RUN:io          DONE          DONE         READY             1
    16        BLOCKED          DONE          DONE       RUN:cpu             1             1
    17        BLOCKED          DONE          DONE       RUN:cpu             1             1
    18        BLOCKED          DONE          DONE       RUN:cpu             1             1
    19        BLOCKED          DONE          DONE       RUN:cpu             1             1
    20        BLOCKED          DONE          DONE       RUN:cpu             1             1
    21*   RUN:io_done          DONE          DONE          DONE             1

    Stats: Total Time 21
    Stats: CPU Busy 21 (100.00%)
    Stats: IO Busy 15 (71.43%)
    ```

8.  Now run with some randomly generated processes using flags -s
    1 -l 3:50,3:50 or -s 2 -l 3:50,3:50 or -s 3 -l 3:50,
    3:50. See if you can predict how the trace will turn out. What happens when you use the flag -I IO RUN IMMEDIATE versus that
    flag -I IO RUN LATER? What happens when you use the flag -S
    SWITCH ON IO versus -S SWITCH ON END?
    `General observation is that switching on IO affects the CPU utilization much more than changing the IO RUN flag, by logical deduction, this makes sense as we cannot really predict how many IOs the process has, it would be more efficient to switch upon IO most of the time unless the OS has precomputed information of which process has larger proportion of IO calls, in which case, it can intelligently form a schedule for the processes!`
