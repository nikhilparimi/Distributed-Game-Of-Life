# CSA Coursework: Game of Life

## Task Overview

### Introduction

The British mathematician John Horton Conway devised a cellular automaton named ‘The Game of Life’. The game resides on a 2-valued 2D matrix, i.e. a binary image, where the cells can either be ‘alive’ (pixel value 255 - white) or ‘dead’ (pixel value 0 - black). The game evolution is determined by its initial state and requires no further input. Every cell interacts with its eight neighbour pixels: cells that are horizontally, vertically, or diagonally adjacent. At each matrix update in time the following transitions may occur to create the next evolution of the domain:

- any live cell with fewer than two live neighbours dies
- any live cell with two or three live neighbours is unaffected
- any live cell with more than three live neighbours dies
- any dead cell with exactly three live neighbours becomes alive

Consider the image to be on a closed domain (pixels on the top row are connected to pixels at the bottom row, pixels on the right are connected to pixels on the left and vice versa). A user can only interact with the Game of Life by creating an initial configuration and observing how it evolves. Note that evolving such complex, deterministic systems is an important application of scientific computing, often making use of parallel architectures and concurrent programs running on large computing farms.

Your task is to design and implement programs which simulate the Game of Life on an image matrix.

---

## Stage 1 - Parallel Implementation

In this stage, you are required to write code to evolve Game of Life using multiple worker goroutines on a single machine. Below are some suggested steps to help you get started. You are *not* required to follow them. Your implementation will be marked against the success criteria outlined below.

### Step 1

Implement the Game of Life logic as it was described in the task introduction. We suggest starting with a single-threaded implementation that will serve as a starting point in subsequent steps. Your Game of Life should evolve for the number of turns specified in `gol.Params.Turns`. Your Game of Life should evolve the correct image specified by  `gol.Params.ImageWidth` and `gol.Params.ImageHeight`.

The skeleton code starts three goroutines. The diagram below shows how they should interact with each other. Note that not all channels linking IO and the Distributor have been initialised for you. You will need to make them and add them to the `distributorChannels` and `ioChannels` structs. These structs are created in `gol/gol.go`.

![Step 1](content/cw_diagrams-Parallel_1.png)

You are not able to call methods directly on the IO goroutine. To use the IO, you will need to utilise channel communication. For reading in the initial PGM image, you will need the `command`, `filename` and `input` channels. Look at the file `gol/io.go` for details. The functions `io.readPgmImage` and `startIo` are particularly important in this step.

Your Game of Life code will interact with the user or the unit tests using the `events` channel. All events are defined in the file `gol/event.go`. In this step, you will only be working with the unit test `TestGol`. Therefore, you only need to send the `FinalTurnComplete` event.

Test your serial, single-threaded code using `go test -v -run=TestGol/-1$`. All the tests ran should pass.

### Step 2

![Step 2](content/cw_diagrams-Parallel_2.png)

Parallelise your Game of Life so that it uses worker threads to calculate the new state of the board. You should implement a distributor that tasks different worker threads to operate on different parts of the image in parallel. The number of worker threads you should create is specified in `gol.Params.Threads`.

*Note: You are free to design your system as you see fit, however, we encourage you to primarily use channels*

Test your code using `go test -v -run=TestGol`. You can use tracing to verify the correct number of workers was used this time.

### Step 3

![Step 3](content/cw_diagrams-Parallel_3.png)

The lab sheets included the use of a timer. Now using a ticker, report the number of cells that are still alive *every 2 seconds*. To report the count use the `AliveCellsCount` event.

Test your code using `go test -v -run=TestAlive`.

### Step 4

![Step 4](content/cw_diagrams-Parallel_4.png)

Implement logic to output the state of the board after all turns have completed as a PGM image.

Test your code using `go test -v -run=TestPgm`. Finally, run `go test -v` and make sure all tests are passing.

### Step 5

![Step 5](content/cw_diagrams-Parallel_5.png)

Implement logic to visualise the state of the game using SDL. You will need to use `CellFlipped` and `TurnComplete` events to achieve this. Look at `sdl/loop.go` for details. Don't forget to send a CellFlipped event for all initially alive cells before processing any turns.

Also, implement the following control rules. Note that the goroutine running SDL provides you with a channel containing the relevant keypresses.

- If `s` is pressed, generate a PGM file with the current state of the board.
- If `q` is pressed, generate a PGM file with the current state of the board and then terminate the program. Your program should *not* continue to execute all turns set in `gol.Params.Turns`.
- If `p` is pressed, pause the processing and print the current turn that is being processed. If `p` is pressed again resume the processing and print `"Continuing"`. It is *not* necessary for `q` and `s` to work while the execution is paused.

Test the visualisation and control rules by running `go run .`

### Success Criteria

- Pass all test cases under `TestGol`, `TestAlive` and `TestPgm`.
- Use the correct number of workers as requested in `gol.Params`.
- Display the live progress of the game using SDL.
- Ensure that all keyboard control rules work correctly.
- Use benchmarks to measure the performance of your parallel program.
- The implementation must scale well with the number of worker threads.
- The implementation must be free of deadlocks and race conditions.

### In your Report

- Discuss the goroutines you used and how they work together.
- Explain and analyse the benchmark results obtained. You may want to consider using graphs to visualise your benchmarks.
- Analyse how your implementation scales as more workers are added.
- Briefly discuss your methodology for acquiring any results or measurements.

---

## Stage 2 - Distributed Implementation

In this stage, you are required to create an implementation that uses a number of
AWS nodes to cooperatively calculate the new state of the Game of Life board,
and communicate state between machines over a network.  Below is a series of
suggested steps for approaching the problem, but you are *not* required to
follow this sequence, and can jump straight to implementing the more advanced
versions of the system if you feel confident about it.

### Step 1

![Step 1](content/cw_diagrams-Distributed_1.png)

Begin by ensuring you have a working single-threaded, single-machine implementation. You should be able to test your serial code using `go test -v -run=TestGol/-1$` and all tests should pass.

Separate your implementation into two components. One component, the local controller, will be responsible for IO and capturing keypresses. The second component, the GOL Engine, will be responsible for actually processing the turns of Game of Life. You must be able to run the local controller as a client on a local machine, and the GoL engine as a server on an AWS node.

Start by implementing a basic controller which can tell the logic engine to evolve Game of Life for the number of turns specified in `gol.Params.Turns`. You can achieve this by implementing a single, blocking RPC call to process all requested turns.

Test your implementation using `go test -v -run=TestGol/-1$` *on the controller*.

### Step 2

![Step 2](content/cw_diagrams-Distributed_2.png)

You should report the number of cells that are still alive *every 2 seconds* to the local controller. The controller should then send an `AliveCellsCount` event to the `events` channel.  

Test your implementation using `go test -v -run=TestAlive` *on the controller*.

### Step 3

![Step 3](content/cw_diagrams-Distributed_3.png)

The local controller should be able to output the state of the board after all turns have completed as a PGM image. 

Test your implementation using `go test -v -run=TestPgm/-1$` *on the controller*.

### Step 4

![Step 4](content/cw_diagrams-Distributed_4.png)

Finally, the local controller should be able to manage the behaviour of the GoL engine according to the following rules: 

- If `s` is pressed, the controller should generate a PGM file with the current state of the board.
- If `q` is pressed, close the controller client program without causing an error on the GoL server. A new controller should be able to take over interaction with the GoL engine.
- If `k` is pressed, all components of the distributed system are shut down cleanly, and the system outputs a PGM image of the latest state.
- If `p` is pressed, pause the processing *on the AWS node* and have the *controller* print the current turn that is being processed. If `p` is pressed again resume the processing and have the controller print `"Continuing"`. It is *not* necessary for `q` and `s` to work while the execution is paused.

Test the control rules by running `go run .`.

### Step 5

![Step 5](content/cw_diagrams-Distributed_5.png)

Split up the computation of the GoL board state (from the GoL server) across
multiple worker machines (AWS nodes).  You will need some means of distributing
work between multiple AWS machines and gathering results together in one place
while avoiding any errors in the collected board state.  Try to design your
solution so it takes advantage of the possible *scalability* of many worker
machines.

Make sure to keep the communication between nodes as efficient as possible. For example, consider a halo exchange scheme where only the edges are communicated between the nodes.

### Step 6

![Step 6](content/cw_diagrams-Distributed_6.png)

Reducing coupling between the "Local Controller" and the "GOL workers" is desirable. To initiate communication, the "Local Controller" connects to the broker machine via RPC. This allows the "Local Controller" to start the game by calling the main "Broker" method, which returns the final game state once it is finished. Likewise, the "Broker" connects to the "GOL workers". It is then able to give them slices of the game world and ask them to return the result of iterating on it.

**Note that it is fine to have the Broker and Local Controller running on the same machine to get around firewall / port forwarding issues**

### Success Criteria

- Pass all tests.
- Output the correct PGM images.
- Ensure the keyboard control rules work as needed.
- At minimum, the controller and the Game of Life engine should be separate
  components running on different machines (as per Step 2 above) and
communicating.
- To fully satisfy the criteria your implementation should use multiple AWS nodes efficiently.

*There is __no need__ to display the live progress of the game using SDL. However, you will still need to run a blank SDL window to register the keypresses.*

### In your report

- Discuss the system design and reasons for any decisions made. Consider using a diagram to aid your discussion.
- Explain what data is sent over the network, when, and why it is necessary.  
- Discuss how your system might scale with the addition of other distributed
  components.
- Briefly discuss your methodology for acquiring any results or measurements.
- Identify how components of your system disappearing (e.g., broken network
  connections) might affect the overall system and its results.

## Extensions

Below are suggested extensions. They vary in difficulty. There are many other possible extensions to Game of Life. If you'd like to implement something that isn't an option below you're welcome to do so, but please speak to a lecturer first.

### Parallel Distributed System

![Extension 1](content/cw_diagrams-Extensions_1.png)

Add parallel workers within each distributed AWS Node.

Analyse the performance of your new solution and compare it with your previous implementation. Use various provided PGM images and analyse the effect on performance in context of the image size.

### SDL Live View of Distributed Implementation

Instead of showing a blank SDL window in your local controller, add support for a Live View, in a similar way to the parallel implementation. Try to keep your implementation as efficient as possible.

Analyse the performance of your new solution and compare it with your previous implementation. Quantify and explain the overhead (if any) added by the Live View.

### Fault Tolerance

Add fault tolerance to your Distributed Implementation.

In your report, explain the design of your fault tolerance mechanism. Conduct experiments to verify the effectiveness of your fault tolerance approach.

### Memory Sharing

Redesign your parallel implementation to use pure memory sharing. Replace *all* channels with traditional synchronisation mechanisms (mutexes, sempahores, condition variables). We recommend first replacing any channels used between the workers and the distributor. Then remove channels linking the distributor with the IO and with SDL. You should still keep them as seperate goroutines. Your solution must be free of deadlocks and race conditions.

Analyse the performance of your new solution and compare it with your previous implementation. Explain any differences observed.

---

## Viva

You will be required to demonstrate your implementations in a viva. This will include running tests as well as showing PGM image output and working keyboard control.

As part of the viva, we will also discuss your report. You should be prepared to discuss and expand on any points mentioned in your report.

