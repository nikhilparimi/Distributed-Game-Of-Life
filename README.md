# Distributed-Game-Of-Life

## Overview

A distributed, concurrent, multi-node system which simulates Conway's Game Of Life on a 2D board; utilises a broker _(which evenly distributes work amongst a variable number of worker nodes stored on AWS Cloud ec2 medium instances)_ to display it _(as an image matrix)_ using SDL2.

---

## General structure:

![Step 6](content/cw_diagrams-Distributed_6.png)

**Aside: Reducing coupling between the "Local Controller" and the "GOL workers" is desirable, which is why we decided to compartmentalise these 2 components.**

To initiate communication, the `Local Controller` connects to the broker machine via `RPC`. This allows the `Local Controller` to start the game by calling the main `Broker` method, which returns the final game state once it is finished.

Likewise, the `Broker` connects to the `GOL workers`. It is then able to give them slices of the game world and ask them to return the result of iterating on it.

**Note: It is fine to have the Broker and Local Controller running on the same machine to get around firewall / port forwarding issues**

---

## Technologies used:
- GoLang
- SDL2
- AWS Cloud (EC2)
- Delve (https://github.com/go-delve/delve)
- XLaunch (to start the server remotely via SSH)
- PowerShell
- WSL2
- VSCode
- IntelliJ

---

## Project Brief:

**The project brief is located in: [PROJECT_BRIEF.md](PROJECT_BRIEF.md)**