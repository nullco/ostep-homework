```bash
python process-run.py -h
Usage: process-run.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
  -P PROGRAM, --program=PROGRAM
                        more specific controls over programs
  -l PROCESS_LIST, --processlist=PROCESS_LIST
                        a comma-separated list of processes to run, in the
                        form X1:Y1,X2:Y2,... where X is the number of
                        instructions that process should run, and Y the
                        chances (from 0 to 100) that an instruction will use
                        the CPU or issue an IO (i.e., if Y is 100, a process
                        will ONLY use the CPU and issue no I/Os; if Y is 0, a
                        process will only issue I/Os)
  -L IO_LENGTH, --iolength=IO_LENGTH
                        how long an IO takes
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        when to switch between processes: SWITCH_ON_IO,
                        SWITCH_ON_END
  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        type of behavior when IO ends: IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
  -c                    compute answers for me
  -p, --printstats      print statistics at end; only useful with -c flag
                        (otherwise stats are not printed)
```

# Chapter - The Abstraction: The Process

## Homework

1. Run process-run.py with the following flags: -l 5:100,5:100.
What should the CPU utilization be (e.g., the percent of time the
CPU is in use?) Why do you know this? Use the -c and -p flags to
see if you were right.

    **Answer**: With those flags we are running 2 processes
    with 5 instructions each and fully CPU bound. That means 100%
    of the time the CPU is being used.

    ```bash
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

2. Now run with these flags: ./process-run.py -l 4:100,1:0.
These flags specify one process with 4 instructions (all to use the
CPU), and one that simply issues an I/O and waits for it to be done.
How long does it take to complete both processes? Use -c and -p
to find out if you were right.

    **Answer**: It takes 11 cycles in total. 4 to run the first process
    and the other 7 for the one doing IO. IO takes 5 cycles by default
    with IO_LENGTH (see help for -L option).

    ```bash
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

3. Switch the order of the processes: -l 1:0,4:100. What happens
now? Does switching the order matter? Why? (As always, use -c
and -p to see if you were right)

    **Answer**: Yes, it should matter. Since the first process run is IO bound,
    it will enter into RUN:io, and then the CPU will be free to execute the other
    process concurrently while IO is done

    ```bash
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

4. We’ll now explore some of the other flags. One important flag is
-S, which determines how the system reacts when a process is-
sues an I/O. With the flag set to SWITCH ON END, the system
will NOT switch to another process while one is doing I/O, in-
stead waiting until the process is completely finished. What hap-
pens when you run the following two processes (-l 1:0,4:100
-c -S SWITCH_ON_END), one doing I/O and the other doing CPU
work?

    **Answer**: With the -S SWITCH_ON_END flag, the CPU won't be able
    to run the CPU bound process while the IO bound one is blocked.
    So we will have again 11 cycles to run both processes. Just like
    running the CPU bound first, and then the IO bound one.

    ```bash
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
     10           DONE       RUN:cpu             1
     11           DONE       RUN:cpu             1

    Stats: Total Time 11
    Stats: CPU Busy 6 (54.55%)
    Stats: IO Busy  5 (45.45%)
    ```

5. Now, run the same processes, but with the switching behavior set
to switch to another process whenever one is WAITING for I/O (-l
1:0,4:100 -c -S SWITCH_ON_IO). What happens now? Use -c
and -p to confirm that you are right.

    **Answer**: With the -S SWITCH_ON_IO flag, the CPU will be able to switch
    to another process when IO comes. So 7 cycles in total are needed

    ```bash
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

6. One other important behavior is what to do when an I/O com-
pletes. With -I IO RUN LATER, when an I/O completes, the pro-
cess that issued it is not necessarily run right away; rather, what-
ever was running at the time keeps running. What happens when
you run this combination of processes? (./process-run.py -l
3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I
IO_RUN_LATER) Are system resources being effectively utilized?

    **Answer**: IO bound process will start running, then after it finishes
    all other CPU bound processes will be run. Only after all are run, then
    IO bound one is run again. Resources are not effectively utilized because
    there's gonna be point in time when CPU is doing nothing while IO happens.

    ```bash
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

7. Now run the same processes, but with -I IO RUN IMMEDIATE set,
which immediately runs the process that issued the I/O. How does
this behavior differ? Why might running a process that just com-
pleted an I/O again be a good idea?

    **Answer**: This will allow system resources to be used more wisely
    because the CPU will be able to run other stuff while IO happens,

    ```bash
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
    Stats: IO Busy  15 (71.43%)
    ```
