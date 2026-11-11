# Experiment 3: Synchronization Mechanisms

A C programming experiment to understand how threads safely access
shared data in parallel programs.

## Programs

  -----------------------------------------------------------------------
  File                                What it demonstrates
  ----------------------------------- -----------------------------------
  `race.c`                            A race condition caused by
                                      unsynchronized updates

  `critical.c`                        Protecting a shared section with
                                      OpenMP

  `atomic.c`                          Performing a shared update
                                      atomically

  `barrier.c`                         Making threads wait at a common
                                      point

  `semaphore.c`                       Controlling access with a POSIX
                                      semaphore

  `compare_sync.c`                    Comparing synchronization methods
                                      across thread counts
  -----------------------------------------------------------------------

## What I Learned

-   **Race condition:** `counter++` involves reading, changing, and
    writing a value. Threads can overlap these steps, causing updates to
    be lost.
-   **Critical section:** Allows only one thread at a time into a
    protected block. It is safe, but waiting can reduce performance.
-   **Atomic operation:** Protects a simple update as one indivisible
    operation. It is useful for shared counters and often has less
    overhead than a critical section.
-   **Barrier:** Makes threads wait until every thread reaches the same
    point before continuing. It coordinates work phases rather than
    protecting data.
-   **Semaphore:** Uses `sem_wait()` and `sem_post()` to control access
    to a shared resource.
-   **Comparison:** Shows the trade-off between correctness and
    execution time as the number of threads increases.

## Requirements

-   Ubuntu / Linux
-   GCC
-   OpenMP
-   POSIX Threads

Install compiler tools on Ubuntu:

``` bash
sudo apt update
sudo apt install build-essential -y
```

## Compile

``` bash
gcc race.c -fopenmp -o race
gcc critical.c -fopenmp -o critical
gcc atomic.c -fopenmp -o atomic
gcc barrier.c -fopenmp -o barrier
gcc semaphore.c -pthread -o semaphore
gcc compare_sync.c -fopenmp -o compare_sync
```

## Run

``` bash
./race
./critical
./atomic
./barrier
./semaphore
./compare_sync
```

`compare_sync` runs a 10-million-increment test with 1, 2, 4, and 8
threads. It displays the final counter, whether the result is correct,
and the execution time. Timings and race-condition results may vary by
machine and run.

## Screenshots

### Individual Program Execution

<img width="640" height="400" alt="Programs" src="https://github.com/user-attachments/assets/4746af56-3eba-4896-afde-111c6d8004bf" />


### Comparison Output

<img width="640" height="400" alt="compare-sync" src="https://github.com/user-attachments/assets/56eb99c3-9625-4ff3-b431-ca329bab4448" />


## Conclusion

Synchronization is essential when multiple threads update shared data.
Atomic operations are a useful choice for simple updates, while critical
sections, locks, and semaphores protect larger or controlled sections.
Barriers coordinate when threads move between phases.

------------------------------------------------------------------------

**Topics:** C · Multithreading · OpenMP · POSIX Threads ·
Synchronization
