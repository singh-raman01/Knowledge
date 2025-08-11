### Chapter 22: Concurrency in Go
*What is Concurrency*?

We will define some terms first, then go through the definition of concurrency:
*Partial Order*: Some things are not ordered, but some are ordered.

![[Screenshot 2025-07-27 at 11.53.55 am.png | 400]]
- 1 happens before parts of 2 & 3
- 2 and 3 are complete before 4
- parts of 2 & 3 are ordered among themselves
- No relationship between 2's and 3's

*Non deterministic*: different behaviours on different runs, even with same input, but from outside the output looks the same (just different tracks of execution)

*Execute independently*:
Sub routines are subordinate, while co-routines are co-equal
![[Screenshot 2025-07-27 at 11.57.36 am.png  | 400]]

Thus, we have:
**Concurrency**: Parts of Program may execute independently, in some non-deterministic (partial) order 
- (this is an aspect of how you put your program together)
- (can have speed ups when we have async processes, like network connections)
- (Concurrency is about dealing with things happening out of order)
**Parallelism**: Parts of program execute independently at the same time 
- (demands multiple cores of a processor)
- (demands concurrency, and this happens at runtime)
- (Parallelism is about things actually happening at the same time)

Idea:
- You can have concurrency with a single-core processor (interrupt handling in a single core processor)
- Parallelism can happen only in a multi-core processor
- Concurrency doesn't make the program faster, parallelism does
- A single program won't have parallelism without concurrency
- We need concurrency to allow parts of program to execute independently

*Race conditions*: (is a bug)
System behaviour depends on the (non-deterministic) sequence or timing of parts of the program executing independently, where some possible behaviours (orders of execution) produce invalid results

Solutions to solve race conditions:
- Do not share things
- Make the shared things read only
- Allow only one writer to the shared things
- Make the Read-Modify-Write operations atomic (In this case we are adding more (sequential) order to our operations)


### Chapter 23: CSP(Communicating Sequential Processes) , Go Routines, Channels

*Channel*:
A channel is a one-way communication pipe
- Things go in one end, come out the other
- In the same order that they come in
- Until the channel is closed
- Multiple readers and writers can share it safely (in go)

*Sequential Processes*:
Looking at a single independent part of a program
It appears to be sequential
Ex. Reading and Writing from file Sockets
```go
for {
	read()
	process()
	write()
}
```
![[Screenshot 2025-07-27 at 12.35.05 pm.png | 400]]
Channels allow these multiple sequential processes to be concurrent as a group.
- Each part is independent
- All they share are the channels between them
- The parts can run in parallel as the hardware allows

*Though CSP, which provides a model for thinking about it takes it less hard*
GO let's you write asynchronous code in synchronous style

*Go Routines* (is a co-routine) 
this is not a thread, can have 1000's routines, and go handles the threading by itself
Put `go` in front of a function call to start a Go routine
How does a Go routine stop?
- You have a well defined loop terminating condition
- You signal completion through a channel or context
- You let it run until the program stops
But we need to make sure it doesn't get blocked by mistake

*Channels* (is a data type)
- method of synchronisation as well as communication
- Also a vehicle for transferring ownership of data, so that only one go routine at a time is writing the data (to avoid race conditions)


```go

We have the ability to restrict our channels to write only or read only

Ex. func(ch chan<- result) // this func can write to the channel but cannot read data from it
```

If there is nothing to read in the channel, we will just wait
- You can only close a channel once
- If we have multiple routines, we don't know which routine will close the channel
- In most of the cases, we know that we started an 'n' number of routines, so we need to read 'n' number of times
- We cannot write to a channel unless somebody is ready to read from it? (for unbuffered channels)
	- - A goroutine **trying to send a value to the channel** (`ch <- value`) will **block (pause execution)** until **another goroutine is receiving** from that channel (`<-ch`)
	- If **no one is reading**, the **sending goroutine will hang**
	- Similarly, a goroutine **trying to receive from a channel** will block **until someone sends a value.**


*Example to understand Channels: **Prime Sieve***:
![[Screenshot 2025-07-27 at 1.23.55 pm.png | 400]]

Every time we see a new prime number, we will create a new go routine with that filter, and hook that up with the generator
Code is in github.

- Generator closes the channel, which creates a domino effect, and the routines that are reading from it will close themselves

The implementation for this is v slow due to a lot of communication happening between the go channels

### Chapter 24: Select
It is a control structure which allows us to work on channels 
It allows us to *multiplex* channels
Select allows any "ready" alternative to proceed among:
- A channel we can read from
- A channel we can write to
- A default action that's always ready

Most often a select runs in a loop so we keep trying
We can put a timeout or "done" channel into the select
- We can compose channels using synchronisation primitives
- Traditional primitives (mutex, conditional variables) cannot be composed

Why?

```go

for i:=0;i<12;i++{
	i1 := <- chans[0]
	i2 := <- chans[1]
}

// for range 12{
// m0 := <- chans[0];
// log.Println("Received",m0)
// m1 := <- chans[1];
// log.Println("Received",m1)
// }
```
In this case even though I can listen to the channels without needing the select statement, I have to keep reading from both channels again and again , even when their rates of sending information might be a bit different.
Instead I wanna pick the channel which is ready and read from it.

Select allows us to read multiple channels at the same time, and whichever is ready first we read from it.


In the commented case, we have to wait for 2 seconds for every read from the channels because one of them is slow.