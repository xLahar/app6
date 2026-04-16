-----------------------------------------------------
# Velocity Kingdom – Thunder Vortex Ride Control System
**Author:** Logan Sullivan  
**Course:** EEE 4775 – Real-Time Systems  
**Semester:** Spring 2026  
-----------------------------------------------------

# AI Use:
Asked it to help me come up with names for the park and ride. Also asked for help with themed serial output.
https://chatgpt.com/share/69e127a7-d04c-83ea-88ba-e657c1546ced


# Synopsis:
Velocity Kingdom is a fictional Central Florida theme park technology company focused on safe and immersive ride experiences.
This project simulates the real-time control system for its flagship roller coaster, **Thunder Vortex**. Real-time behavior is critical
because ride control tasks must respond within strict timing deadlines to ensure rider safety. Missing a deadline in brake activation or
emergency stop handling could result in unsafe ride conditions, making several tasks in this system hard real-time.


# Task Table:

|       Task / ISR        |      Period         | Hard / Soft |          Consequence if Missed           |
|-------------------------|---------------------|-------------|------------------------------------------|
| Sensor Task             | 500 ms              | Hard        | Unsafe delay in track position detection |
| Brake Control Task      | 500 ms              | Hard        | Late braking may cause ride hazard       |
| E-STOP ISR              | Event-driven        | Hard        | Delayed emergency response               |
| E-STOP Task             | Event-driven        | Hard        | Ride may not halt safely                 |
| Heartbeat Task          | 1000 ms             | Hard        | Loss of timing proof / system status     |
| Station Operations Task | 200–500 ms variable | Soft        | Minor delay in station processes         |


# Engineering Analysis:

1. Scheduler Fit
The task priorities were assigned so all hard real-time safety tasks always run before non-critical tasks. The E-stop task has the
highest priority (5), followed by the brake task (4) and sensor task (3), which guarantees that safety-critical functions preempt the
soft real-time station task. For example, the timestamps [7504 ms] emergency stop activated and [7832 ms] brakes engaged show the system
responded well within the 500 ms hard task deadline.

Here is those timestamps:
[7504 ms] CONTROL ALERT: Thunder Vortex emergency stop activated
[7742 ms] Thunder Vortex station operations task running...
[7832 ms] Thunder Vortex magnetic brakes ENGAGED

2. Race-Proofing
A race condition could occur when multiple tasks try to control the brake and run LEDs at the same time, especially between the brake task
and E-stop task. This was protected using a mutex (brakeMutex) around the critical section:

xSemaphoreTake(brakeMutex, portMAX_DELAY);
...
xSemaphoreGive(brakeMutex);

This ensures only one task updates the ride control outputs at a time.

3. Worst-Case Spike
The heaviest load was triggering the emergency stop while the sensor and station task were actively running. Even during this spike,
the E-stop was logged at 7504 ms and the brakes engaged at 7832 ms, leaving about 328 ms of margin before the 500 ms hard deadline would
be missed.

4. Design Trade-Off
One feature I intentionally simplified was wireless communication / web monitoring. While it would have made the prototype more realistic,
adding Wi-Fi tasks could introduce unpredictable latency and scheduling overhead. For a ride safety system, keeping timing predictable was
the better choice because rider safety depends on deterministic brake and emergency response timing.
