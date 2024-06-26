# Exponential Backoff with Collision Avoidance

This module defines a Python iterator implementing exponential backoff
with collision avoidance. See
https://en.wikipedia.org/wiki/Exponential_backoff#Collision_avoidance for
a complete explanation of the algorithm.

A fixed time slot size is defined and then in each iteration a random
number of these times slots is chosen. The wait time is the total amount
of time represented by these time slots. In each iteration the number of
available time slots increases.

For example, define the time slot size to be two seconds. In the first
iteration the number of available slots is `2**1-1=1`. The number N of
slots is chosen randomly in the integer interval [0, 1], that is, either
zero or one. The wait time returned is two seconds multiplied by N.

In the second iteration the number of available slots is `2**2-1=3`. The
number N of slots is chosen randomly in the integer interval [0, 3], that
is, either zero, one, two, or three. The wait time returned is two seconds
multiplied by N.

And so on.

Because the number of slots is randomly chosen in [0, N], the wait time
returned in each iteration can sometimes go _down_.

Here is a simple example:
```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0 # The number of seconds in each time slot.
num_iterations = 10  # The number of iterations.

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations)

for interval in exp_boff:
    print(f"wait seconds is {interval}")
```
Will output (something like)
```
wait seconds is 2.0
wait seconds is 6.0
wait seconds is 4.0
wait seconds is 24.0
wait seconds is 0.0
wait seconds is 114.0
wait seconds is 238.0
wait seconds is 184.0
wait seconds is 926.0
wait seconds is 1438.0
```

You can also access where you are in the iteration using the `counter` property.
```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0
num_iterations = 5

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations)

for interval in exp_boff:
    print(f"number of times going through loop: {exp_boff.counter}")
```
Will output
```
number of times going through loop: 1
number of times going through loop: 2
number of times going through loop: 3
number of times going through loop: 4
number of times going through loop: 5
```

To reuse the iterator you need to reset the counter back to zero. You do this 
via the `reset()` method:

```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0
num_iterations = 5

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations)

for interval in exp_boff:
    print(f"number of times going through loop: {exp_boff.counter}")

exp.boff.reset()
for interval in exp_boff:
    print(f"number of times going through loop (take two): {exp_boff.counter}")
```
Will output
```
number of times going through loop: 1
number of times going through loop: 2
number of times going through loop: 3
number of times going through loop: 4
number of times going through loop: 5
number of times going through loop (take two): 1
number of times going through loop (take two): 2
number of times going through loop (take two): 3
number of times going through loop (take two): 4
number of times going through loop (take two): 5
```

To limit the maxiumum number of available slots use the optional `max_slots` parameter:

```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0 # The number of seconds in each time slot.
num_iterations = 10  # The number of iterations.
max_slots      = 4   # No more than 4 slots available (so wait time <= 4 * 2.0 seconds)

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations, max_slots=max_slots)

for interval in exp_boff:
    print(f"wait seconds is {interval}")
```
Will output (something like)
```
wait seconds is 2.0
wait seconds is 2.0
wait seconds is 8.0
wait seconds is 0.0
wait seconds is 8.0
wait seconds is 8.0
wait seconds is 4.0
wait seconds is 4.0
wait seconds is 0.0
wait seconds is 4.0
```

You can also limit the wait time returned by passing in the `limit_value` parameter.
```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0  # The number of seconds in each time slot.
num_iterations = 10   # The number of iterations.
limit_value    = 13.0 # Never return more than 13.0 seconds.

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations, limit_value=limit_value)

for interval in exp_boff:
    print(f"wait seconds is {interval}")
```
will output (something like)
```
wait seconds is 0.0
wait seconds is 4.0
wait seconds is 4.0
wait seconds is 13.0
wait seconds is 0.0
wait seconds is 13.0
wait seconds is 13.0
wait seconds is 6.0
wait seconds is 13.0
wait seconds is 13.0
```

The exponential backoff object doubles the number of slots each time.
However, there may be circumstances where that doubling is too much (or
too little). You can adjust how fast the number of available slots
increases using the `multiplier` parameter. Passing in a
`multiplier` value less than 1.0 will raise an exception.
```
from exponential_backoff_ca import ExponentialBackoff

time_slot_secs = 2.0 # The number of seconds in each time slot.
num_iterations = 10  # The number of iterations.
multiplier     = 1.5 # Increase the number of available slots by 1.5 (takes integer part)

exp_boff = ExponentialBackoff(time_slot_secs, num_iterations, multiplier=multiplier)

for interval in exp_boff:
    print(f"wait seconds is {interval}")
```
Will output something like
```
wait seconds is 0.0
wait seconds is 2.0
wait seconds is 0.0
wait seconds is 4.0
wait seconds is 8.0
wait seconds is 6.0
wait seconds is 14.0
wait seconds is 38.0
wait seconds is 30.0
wait seconds is 22.0
```

To see what the iterator is doing, pass in the `debug` parameter set to
`True`.
