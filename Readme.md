# Zone Job-Scheduler & Deadlock-Safety Engine

Run `python scheduler.py`, `python synchronization.py`, `python banker.py`, and `python memory_translation.py`. All scheduler implementations import the fixed workload from `jobs.py`.

## Measured scheduling results

| Algorithm | Avg. waiting | Avg. turnaround |
|---|---:|---:|
| FCFS | 17.125 | 22.625 |
| Non-preemptive SJF | 13.000 | 18.500 |
| SRTF | 11.500 | 17.000 |
| RR q=3 | 22.625 | 28.125 |
| RR q=6 | 20.375 | 25.875 |
| Priority, no aging | 14.125 | 19.625 |
| Priority, aging | 17.125 | 22.625 |

RR q=3 has 17 dispatch slices and 16 context switches; q=6 has 11 slices and 10 switches. In a real OS q=3 would incur more switching overhead because it causes 16 switches versus 10 for q=6.

## Production choice

I would deploy the **SJF/SRTF family**, specifically SRTF, for this fixed sensor-processing workload: it has the lowest measured average waiting time (11.500) and turnaround time (17.000).

- FCFS is less suitable because its average waiting time is 17.125, versus SRTF's 11.500.
- Round Robin is less suitable because even q=6 has 20.375 average waiting time and 10 context switches; q=3 rises to 22.625 and 16 switches.
- Priority scheduling is less suitable because aging increases average waiting time to 17.125, while no-aging leaves Z3-J02 as the longest waiter, demonstrating fairness risk.


