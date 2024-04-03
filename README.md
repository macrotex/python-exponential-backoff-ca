# Hello


Each iteration returns the total time (in seconds) to delay. It
does this by increasing the maximum number of slots each iteration and
then choosing a random number of slots between 0 and the maximum
number of slots.

(See https://en.wikipedia.org/wiki/Exponential_backoff#Collision_avoidance)

Example:
```
import exponentional_backoff_ca

time_slot_secs = 2.0 # The number of seconds in each time slot.
num_iterations = 10  # The number of iterations.

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations)

for interval in exp_boff:
    print(f"number of seconds in this slot is {interval}")
```
