---
title: "Gridworld Environment"
date:  2020-02-15
layout: single
classes: wide
categories: env
---

All code for this project is stored in <https://github.com/ArikVoronov/Machine-Learning/Reinforcement_Learning/Envs>
## Background and Motivation
Gridworld is one of several environments I've coded for testing with various Reinforcement Learning (RL) algorithms.  
All my environments keep a similiar synthax to the openAI Gym envirnments - reset, step... etc.  
This is a **discrete** environment both in state and action space, so it's possible to use it with discrete RL algorithms.  
There's a gridworld environment in the Gym library, but I like to have a complete understanding and control over the environment, so I made my own version.


## Rules
![Gridworld]({{site.baseurl}}/assets/env/gridworld/Gridworld.png "Gridworld"){: .small .align-center}
The environment is made of adjacent cells, the agent must make a series of discrete movements {up,right,down,left} to reach a goal cell from some starting cell.  The agent can't move through borders or walls.

## Interface
The environment parameters, functions and state are contained in the GridWorldEnv object. An instance will be refered to as **env** in this documentation

### Parameters
- env.ns - row x col - Number of possible states: 
- env.na - 4 - Number of available actions 

### Variables (input/output)
- state: OneHot vector with the the agent's current cell "hot" - list [1xns]  
- action: Choose to move in 1 of 4 directions:   `0:up ; 1:right ; 2:down ; 3:left`  
- reward: The agent gets a -1 reward for every step (this is motivation to reach the goal in as few steps as possible).
- done: boolean, True if the step sequence is done (agent reached the goal, in this case)

### Functions
- env.reset() - resets the player to the starting cell, returns **state**
- env.step(action) - takes one step in the action direction, returns **state,reward,done** 
- env.Render(gameDisplay,qDraw) - renders the current state unto a pygame.display object (gameDisplay). If qDraw=True, also draws the direction of the maximal Q (taken from the RL learner)

## Additional nodes
1. Set the RandomGoal parameter to True to randomize the goal location (otherwise always at the bottom right)
2. In the same link provided above, there's a helpful **RunEnv** function, which runs and renders any tailored environment, including GridWorld.  
Input:
	- runs: the number of times to run the environment (runs untill done, then resets)
	- env: an instance of the Gridworld class
	- agent: a function which accepts state then returns and action (both fit the expected env state/action)
3. Running the main file shows an example of a "random walk" agent going through the grid (this may take a while, it's drunk)


## Code
As mentioned, the code can be found in the link in the beginning of this post, and it is mostly self explanatory.
