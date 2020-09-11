# CS 4375 - Operating Systems

## Homework 3

* OSTEP Chapter 27 questions 1, 2, 3, 4, 5, 6, 7

### Author: Matthew S Montoya

## Chapter 27

In this section, we’ll write some simple multi-threaded programs and use a specific tool, called helgrind, to find problems in these programs.</br>
Read the README in the homework download for details on how to build the programs and run helgrind.

**1. First build main-race.c. Examine the code so you can see the (hopefully obvious) data race in the code. Now run helgrind (by typing valgrind --tool=helgrind main-race) to see how it reports the race. Does it point to the right lines of code? What other information does it give to you?**

The output from the code notes 2 errors from 2 contexts, and points to lines 8 & 15 of the code, where possible data races may exist. These two lines are where the errors exist because _balance_ was zero when it should have been greater than zero. Furthermore, it tells us there are 2 threads that were created: _main_ and _worker_.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-race
==66904== Helgrind, a thread error detector
==66904== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==66904== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==66904== Command: ./main-race
==66904==
==66904== ---Thread-Announcement------------------------------------------
==66904==
==66904== Thread #1 is the program's root thread
==66904==
==66904== ---Thread-Announcement------------------------------------------
==66904==
==66904== Thread #2 was created
==66904==    at 0x518287E: clone (clone.S:71)
==66904==    by 0x4E49EC4: create_thread (createthread.c:100)
==66904==    by 0x4E49EC4: pthread_create@@GLIBC_2.2.5 (pthread_create.c:797)
==66904==    by 0x4C36E33: pthread_create_WRK (hg_intercepts.c:427)
==66904==    by 0x4C37F25: pthread_create@* (hg_intercepts.c:460)
==66904==    by 0x108C54: Pthread_create (mythreads.h:51)
==66904==    by 0x108D26: main (main-race.c:14)
==66904==
==66904== ----------------------------------------------------------------
==66904==
==66904== Possible data race during read of size 4 at 0x30A014 by thread #1
==66904== Locks held: none
==66904==    at 0x108D27: main (main-race.c:15)
==66904==
==66904== This conflicts with a previous write of size 4 by thread #2
==66904== Locks held: none
==66904==    at 0x108CDF: worker (main-race.c:8)
==66904==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==66904==    by 0x4E496DA: start_thread (pthread_create.c:463)
==66904==    by 0x518288E: clone (clone.S:95)
==66904==  Address 0x30a014 is 0 bytes inside data symbol "balance"
==66904==
==66904== ----------------------------------------------------------------
==66904==
==66904== Possible data race during write of size 4 at 0x30A014 by thread #1
==66904== Locks held: none
==66904==    at 0x108D30: main (main-race.c:15)
==66904==
==66904== This conflicts with a previous write of size 4 by thread #2
==66904== Locks held: none
==66904==    at 0x108CDF: worker (main-race.c:8)
==66904==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==66904==    by 0x4E496DA: start_thread (pthread_create.c:463)
==66904==    by 0x518288E: clone (clone.S:95)
==66904==  Address 0x30a014 is 0 bytes inside data symbol "balance"
==66904==
==66904==
==66904== Use --history-level=approx or =none to gain increased speed, at
==66904== the cost of reduced accuracy of conflicting-access information
==66904== For lists of detected and suppressed errors, rerun with: -s
==66904== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

**2. What happens when you remove one of the offending lines of code? Now add a lock around one of the updates to the shared variable, and then around both. What does helgrind report in each of these cases?**

When we remove one of the offending lines of code, it removes the two errors that were displayed in the previous output. We see this listed below.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-race
==67936== Helgrind, a thread error detector
==67936== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==67936== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==67936== Command: ./main-race
==67936==
==67936==
==67936== Use --history-level=approx or =none to gain increased speed, at
==67936== the cost of reduced accuracy of conflicting-access information
==67936== For lists of detected and suppressed errors, rerun with: -s
==67936== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

When we addd a lock around one of the updates to the shared variable, helgrind reports that there is still a possible data race, similar to what we saw in the previous questions. However, this time, helgrind acknowledges that there is a lock around the update we implemented the lock on.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-race
==68862== Helgrind, a thread error detector
==68862== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==68862== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==68862== Command: ./main-race
==68862==
==68862== ---Thread-Announcement------------------------------------------
==68862==
==68862== Thread #1 is the program's root thread
==68862==
==68862== ---Thread-Announcement------------------------------------------
==68862==
==68862== Thread #2 was created
==68862==    at 0x518287E: clone (clone.S:71)
==68862==    by 0x4E49EC4: create_thread (createthread.c:100)
==68862==    by 0x4E49EC4: pthread_create@@GLIBC_2.2.5 (pthread_create.c:797)
==68862==    by 0x4C36E33: pthread_create_WRK (hg_intercepts.c:427)
==68862==    by 0x4C37F25: pthread_create@* (hg_intercepts.c:460)
==68862==    by 0x108C54: Pthread_create (mythreads.h:51)
==68862==    by 0x108D26: main (main-race.c:18)
==68862==
==68862== ----------------------------------------------------------------
==68862==
==68862==  Lock at 0x30A040 was first observed
==68862==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==68862==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==68862==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==68862==    by 0x108D32: main (main-race.c:20)
==68862==  Address 0x30a040 is 0 bytes inside data symbol "lock"
==68862==
==68862== Possible data race during read of size 4 at 0x30A024 by thread #1
==68862== Locks held: 1, at address 0x30A040
==68862==    at 0x108D33: main (main-race.c:21)
==68862==
==68862== This conflicts with a previous write of size 4 by thread #2
==68862== Locks held: none
==68862==    at 0x108CDF: worker (main-race.c:11)
==68862==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==68862==    by 0x4E496DA: start_thread (pthread_create.c:463)
==68862==    by 0x518288E: clone (clone.S:95)
==68862==  Address 0x30a024 is 0 bytes inside data symbol "balance"
==68862==
==68862== ----------------------------------------------------------------
==68862==
==68862==  Lock at 0x30A040 was first observed
==68862==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==68862==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==68862==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==68862==    by 0x108D32: main (main-race.c:20)
==68862==  Address 0x30a040 is 0 bytes inside data symbol "lock"
==68862==
==68862== Possible data race during write of size 4 at 0x30A024 by thread #1
==68862== Locks held: 1, at address 0x30A040
==68862==    at 0x108D3C: main (main-race.c:21)
==68862==
==68862== This conflicts with a previous write of size 4 by thread #2
==68862== Locks held: none
==68862==    at 0x108CDF: worker (main-race.c:11)
==68862==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==68862==    by 0x4E496DA: start_thread (pthread_create.c:463)
==68862==    by 0x518288E: clone (clone.S:95)
==68862==  Address 0x30a024 is 0 bytes inside data symbol "balance"
==68862==
==68862==
==68862== Use --history-level=approx or =none to gain increased speed, at
==68862== the cost of reduced accuracy of conflicting-access information
==68862== For lists of detected and suppressed errors, rerun with: -s
==68862== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

When we addd a lock around **both** of the updates to the shared variable, helgrind reports that there are no errors. It also shows 7 suppressions. We see this listed below.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-race
==68763== Helgrind, a thread error detector
==68763== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==68763== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==68763== Command: ./main-race
==68763==
==68763==
==68763== Use --history-level=approx or =none to gain increased speed, at
==68763== the cost of reduced accuracy of conflicting-access information
==68763== For lists of detected and suppressed errors, rerun with: -s
==68763== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 7 from 7)
```

**3. Now let’s look at main-deadlock.c. Examine the code. This code has a problem known as deadlock (which we discuss in much more depth in a forthcoming chapter). Can you see what problem it might have?**

The problem ```main-deadlock.c``` has is that there could be a deadlock if one of the threads (_p1_ or _p2_) attempt to access the locks simultaneously. If this occurs, both threads can potantially hang while waiting for locks.

**4. Now run helgrind on this code. What does helgrind report?**

Running helgrind on ```main-deadlock.c``` reports an incorrect lock order on Thread #3, and exits with 1 error. After running this multiple times, the same output was observed. The output is shown below.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-deadlock
==69871== Helgrind, a thread error detector
==69871== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==69871== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==69871== Command: ./main-deadlock
==69871==
==69871== ---Thread-Announcement------------------------------------------
==69871==
==69871== Thread #3 was created
==69871==    at 0x518287E: clone (clone.S:71)
==69871==    by 0x4E49EC4: create_thread (createthread.c:100)
==69871==    by 0x4E49EC4: pthread_create@@GLIBC_2.2.5 (pthread_create.c:797)
==69871==    by 0x4C36E33: pthread_create_WRK (hg_intercepts.c:427)
==69871==    by 0x4C37F25: pthread_create@* (hg_intercepts.c:460)
==69871==    by 0x108C54: Pthread_create (mythreads.h:51)
==69871==    by 0x108D89: main (main-deadlock.c:24)
==69871==
==69871== ----------------------------------------------------------------
==69871==
==69871== Thread #3: lock order "0x30A040 before 0x30A080" violated
==69871==
==69871== Observed (incorrect) order is: acquisition of lock at 0x30A080
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108D06: worker (main-deadlock.c:13)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==
==69871==  followed by a later acquisition of lock at 0x30A040
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108D12: worker (main-deadlock.c:14)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==
==69871== Required order was established by acquisition of lock at 0x30A040
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108CEC: worker (main-deadlock.c:10)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==
==69871==  followed by a later acquisition of lock at 0x30A080
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108CF8: worker (main-deadlock.c:11)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==
==69871==  Lock at 0x30A040 was first observed
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108CEC: worker (main-deadlock.c:10)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==  Address 0x30a040 is 0 bytes inside data symbol "m1"
==69871==
==69871==  Lock at 0x30A080 was first observed
==69871==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69871==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69871==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69871==    by 0x108CF8: worker (main-deadlock.c:11)
==69871==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69871==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69871==    by 0x518288E: clone (clone.S:95)
==69871==  Address 0x30a080 is 0 bytes inside data symbol "m2"
==69871==
==69871==
==69871==
==69871== Use --history-level=approx or =none to gain increased speed, at
==69871== the cost of reduced accuracy of conflicting-access information
==69871== For lists of detected and suppressed errors, rerun with: -s
==69871== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 7 from 7)
```

**5. Now run helgrind on main-deadlock-global.c. Examine the code; does it have the same problem that main-deadlock.c has? Should helgrind be reporting the same error? What does this tell you about tools like helgrind?**

When comparing helgrind on ```main-deadlock.c``` to helgrind on ```main-deadlock-global.c```, we notice that the output notes an incorrect lock order on Thread #3, and exits with 1 error. This is very similar output to what we had in question 4. However, after examining the code, it appears that ```main-deadlock-global.c``` should run without the issue presented in the third question because there is a g (global) lock,that should prevent deadlock. Yet, after running this multiple times, the same output was observed.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-deadlock-global
==69970== Helgrind, a thread error detector
==69970== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==69970== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==69970== Command: ./main-deadlock-global
==69970==
==69970== ---Thread-Announcement------------------------------------------
==69970==
==69970== Thread #3 was created
==69970==    at 0x518287E: clone (clone.S:71)
==69970==    by 0x4E49EC4: create_thread (createthread.c:100)
==69970==    by 0x4E49EC4: pthread_create@@GLIBC_2.2.5 (pthread_create.c:797)
==69970==    by 0x4C36E33: pthread_create_WRK (hg_intercepts.c:427)
==69970==    by 0x4C37F25: pthread_create@* (hg_intercepts.c:460)
==69970==    by 0x108C54: Pthread_create (mythreads.h:51)
==69970==    by 0x108DA1: main (main-deadlock-global.c:27)
==69970==
==69970== ----------------------------------------------------------------
==69970==
==69970== Thread #3: lock order "0x30A080 before 0x30A0C0" violated
==69970==
==69970== Observed (incorrect) order is: acquisition of lock at 0x30A0C0
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108D12: worker (main-deadlock-global.c:15)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==
==69970==  followed by a later acquisition of lock at 0x30A080
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108D1E: worker (main-deadlock-global.c:16)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==
==69970== Required order was established by acquisition of lock at 0x30A080
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108CF8: worker (main-deadlock-global.c:12)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==
==69970==  followed by a later acquisition of lock at 0x30A0C0
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108D04: worker (main-deadlock-global.c:13)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==
==69970==  Lock at 0x30A080 was first observed
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108CF8: worker (main-deadlock-global.c:12)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==  Address 0x30a080 is 0 bytes inside data symbol "m1"
==69970==
==69970==  Lock at 0x30A0C0 was first observed
==69970==    at 0x4C34480: mutex_lock_WRK (hg_intercepts.c:909)
==69970==    by 0x4C3830C: pthread_mutex_lock (hg_intercepts.c:925)
==69970==    by 0x108AD7: Pthread_mutex_lock (mythreads.h:23)
==69970==    by 0x108D04: worker (main-deadlock-global.c:13)
==69970==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==69970==    by 0x4E496DA: start_thread (pthread_create.c:463)
==69970==    by 0x518288E: clone (clone.S:95)
==69970==  Address 0x30a0c0 is 0 bytes inside data symbol "m2"
==69970==
==69970==
==69970==
==69970== Use --history-level=approx or =none to gain increased speed, at
==69970== the cost of reduced accuracy of conflicting-access information
==69970== For lists of detected and suppressed errors, rerun with: -s
==69970== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 7 from 7)
```

**6. Let’s next look at main-signal.c. This code uses a variable (done) to signal that the child is done and that the parent can now continue. Why is this code inefficient? What does the parent end up spending its time doing, particularly if the child thread takes a long time to complete?**

This code is inefficient because if the child thread is taking a long time to complete, the parent performs a spin-wait, which uses and wastes CPU cycles. This wastes cycles because it will continuously check the value during this time, which is time and cycles that another process can utilize the CPU for.

**7. Now run helgrind on this program. What does it report? Is the code correct?**

Running helgrind on ```main-signal.c``` reports that there are data races on lines 9, 16, and 18. After looking at the code, these three data races are correct because it performs either a done check or prints out put to the terminal. The code is correct and the program runs as intended. However, one thing I noticed is that helgrind did not catch the printf() statement on line 8, which should've identified another data race. It goes on to note 24 errors. We see this in the output listed below.

```text
(base) montoms1@ubuntu:~/Downloads/HW-THREADS$ valgrind --tool=helgrind ./main-signal
==70014== Helgrind, a thread error detector
==70014== Copyright (C) 2007-2017, and GNU GPL'd, by OpenWorks LLP et al.
==70014== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==70014== Command: ./main-signal
==70014==
this should print first
==70014== ---Thread-Announcement------------------------------------------
==70014==
==70014== Thread #2 was created
==70014==    at 0x518287E: clone (clone.S:71)
==70014==    by 0x4E49EC4: create_thread (createthread.c:100)
==70014==    by 0x4E49EC4: pthread_create@@GLIBC_2.2.5 (pthread_create.c:797)
==70014==    by 0x4C36E33: pthread_create_WRK (hg_intercepts.c:427)
==70014==    by 0x4C37F25: pthread_create@* (hg_intercepts.c:460)
==70014==    by 0x108CA4: Pthread_create (mythreads.h:51)
==70014==    by 0x108D81: main (main-signal.c:15)
==70014==
==70014== ---Thread-Announcement------------------------------------------
==70014==
==70014== Thread #1 is the program's root thread
==70014==
==70014== ----------------------------------------------------------------
==70014==
==70014== Possible data race during write of size 4 at 0x30A014 by thread #2
==70014== Locks held: none
==70014==    at 0x108D36: worker (main-signal.c:9)
==70014==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==70014==    by 0x4E496DA: start_thread (pthread_create.c:463)
==70014==    by 0x518288E: clone (clone.S:95)
==70014==
==70014== This conflicts with a previous read of size 4 by thread #1
==70014== Locks held: none
==70014==    at 0x108D83: main (main-signal.c:16)
==70014==  Address 0x30a014 is 0 bytes inside data symbol "done"
==70014==
==70014== ----------------------------------------------------------------
==70014==
==70014== Possible data race during read of size 4 at 0x30A014 by thread #1
==70014== Locks held: none
==70014==    at 0x108D83: main (main-signal.c:16)
==70014==
==70014== This conflicts with a previous write of size 4 by thread #2
==70014== Locks held: none
==70014==    at 0x108D36: worker (main-signal.c:9)
==70014==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==70014==    by 0x4E496DA: start_thread (pthread_create.c:463)
==70014==    by 0x518288E: clone (clone.S:95)
==70014==  Address 0x30a014 is 0 bytes inside data symbol "done"
==70014==
==70014== ----------------------------------------------------------------
==70014==
==70014== Possible data race during write of size 1 at 0x5C531A5 by thread #1
==70014== Locks held: none
==70014==    at 0x4C3C496: mempcpy (vg_replace_strmem.c:1537)
==70014==    by 0x50EC993: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1258)
==70014==    by 0x50E1A8E: puts (ioputs.c:40)
==70014==    by 0x108D98: main (main-signal.c:18)
==70014==  Address 0x5c531a5 is 21 bytes inside a block of size 1,024 alloc'd
==70014==    at 0x4C3121B: malloc (vg_replace_malloc.c:309)
==70014==    by 0x50DF18B: _IO_file_doallocate (filedoalloc.c:101)
==70014==    by 0x50EF378: _IO_doallocbuf (genops.c:365)
==70014==    by 0x50EE497: _IO_file_overflow@@GLIBC_2.2.5 (fileops.c:759)
==70014==    by 0x50EC9EC: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1266)
==70014==    by 0x50E1A8E: puts (ioputs.c:40)
==70014==    by 0x108D35: worker (main-signal.c:8)
==70014==    by 0x4C37027: mythread_wrapper (hg_intercepts.c:389)
==70014==    by 0x4E496DA: start_thread (pthread_create.c:463)
==70014==    by 0x518288E: clone (clone.S:95)
==70014==  Block was alloc'd by thread #2
==70014==
this should print last
==70014==
==70014== Use --history-level=approx or =none to gain increased speed, at
==70014== the cost of reduced accuracy of conflicting-access information
==70014== For lists of detected and suppressed errors, rerun with: -s
==70014== ERROR SUMMARY: 24 errors from 3 contexts (suppressed: 40 from 40)
```
