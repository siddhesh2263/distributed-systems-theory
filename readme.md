# Revisiting distributed systems concepts

<br/>

## Part 1 - Introduction, time, and clocks
<em>1/18/2025</em>

There are various definitions for distributed systems. A distributed system runs on several `nodes`, connected by a network. They are characterized by `partial failures`.

Partial failures occur when some parts of a system fail while others continue to work. These situations are particularly challenging because software often struggles to identify which parts are broken. Ideally, distributed systems should distribute the workload fairly or operate as though they are a single cohesive system, maintaining transparency. However, these goals are often overly optimistic.

A common example of partial failure is when a network of machines experiences a failure due to a network issue. Failures can also occur because of software misbehavior. There are additional perspectives on partial failure as well. For instance, on a single machine, if a part of the operating system fails, it can lead to a kernel panic, requiring the machine to be restarted to recover. In a system with multiple machines, if one machine fails, it becomes critical to ensure the rest of the system continues to function. Restarting all the other machines just because one has failed would not be practical. This philosophy of maintaining system functionality despite partial failures is at the heart of cloud computing.

Cloud computing is specifically designed to handle partial failures by working around them. On the other hand, high-performance computing (HPC) takes a different approach, treating partial failures as complete failures. In HPC, if something goes wrong, the computation is restarted entirely. To minimize the impact, checkpointing is used so the process can resume from the point of failure. Despite these differences in approach, HPC is still a type of distributed system.

In addition to machines themselves failing, the connection between them can fail as well. Let's say there are 2 machines. Machine M1 sends a request to machine M2 requesting a value for X. What are some of the things that could go wrong?
```
1. Request sent from M1 gets lost.
2. Request from M1 is very slow. M2 can't wait forever.
3. M2 has crashed.
4. M1 sends a message, but M2 is taking a lot of time to compute.
5. M2 sends a response, but it is very slow.
6. M2 sends a response, but the message gets lost.
7. M2 could be a bad actor - it could lie or refuse to answer.
8. M2's response gets corrupted.
```

Does M1 have any way of knowing what happened in the above situations? No. From M1's perspective, all these situations are indistinguishable. If a machine sends a request to another node, and it doesn't receive a response, it is impossible to know why (without the global knowledge of the system, which we don't know).
The last 2 causes are known as Byzantine faults.
There can be another case where the value sent by M2 is stale by the time it reaches M1. This is a very important topic of discussion.

How do real systems deal with such cases? Machines need to have some kind of timeout. After a certain amount of time, if we don't get a response, we assume that some failure has taken place. But is it correct to assume failure? What if M1 caused a side effect on M2? M1 asked to increment a value to M2, which M2 does, and sends an OK response. If M1 gets the response, it can assume that the value was incremented. However, if M1 doesn't receive the OK response, it would be wrong to assume that the increment didn't occur. This sort of uncertainty is a fundamental aspect of distributed systems.

So do timeouts really work?

One of the biggest causes of uncertainty in distributed systems is network latency. If we know the maximum time it takes for a message to travel from one machine (M1) to another (M2), represented as d, and we also know the maximum time M2 might take to process a request, represented as r, we can calculate a timeout using the formula `(2d + r)`. This calculation helps eliminate some of the uncertainty caused by delays or slowness in the system.

However, even with this formula, many uncertainties remain. There is no guarantee that the maximum delay value we use is accurate, as network conditions can change unpredictably. This means that, in addition to managing partial failures, distributed systems must also handle the challenges of unbounded latency. In essence, a distributed system can be described as a combination of partial failures and unbounded latency, both of which must be addressed for the system to function effectively.

Why would we want a distributed system, given it has so many issues? Data can be too big to fit on a single machine. There can also be a need to make tasks faster, which can be done with more computers.

49