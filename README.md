# deepbots

Deepbots is a simple framework which is used as "middleware" between the
[Webots](https://cyberbotics.com/) robot simulator and Reinforcement Learning
algorithms. When in comes to Reinforcement Learning the [OpenAI
gym](https://gym.openai.com/) environment has been established as the most used 
interface between the actual application and the RL algorithm. Deepbots is a 
framework which follows the OpenAI gym environment interface in order to be 
used by Webots applications. 

## How it works

First of all let's set up a simple glossary:

+ `Supervisor`: Webots use tree structure to represent the different entities
  of the world. Supervisor is that entity which has access to all entities of
  the world while has not mass or any "physical" property. For example,
  supervisor knows the exact position of all entities in the world.
  Additionally, Supervisor has children nodes, one of them is the Supervisor
  Controller.
  
+ `Supervisor Controller`: is this python script which is responsible about
  supervisor. For example, in Supervisor Controller script we can access the
  distance between two entities in the world. 

+ `Robot`: is this entity in tree that represent a robot in the world. Robot may
  have sensors as children entities. Another children of Robot is the Robot
  Controller. In addition, robots has active components like motors, joints etc.
  For example, [epuck](https://cyberbotics.com/doc/guide/epuck) and
  [TIAGo](https://cyberbotics.com/doc/guide/tiago-iron) are robots. 
  
+ `Robot Controller`: The Robot Controller is a python script which is
  responsible for Robot's movement and sensors. With Robot
  Controller it is feasible to observe the world and act accordingly. 
  
+ `World`: Is the root entity which contains all entities/nodes. For example,
  world contains the supervisor and robot entity as well as objects which might
  be included in the scene. 
  
+ `Environment`: The Environment is the interface as described by the OpenAI gym.
  The Environment interface has the following methods:
  
  + `get_observations()`: Return the observations of the robot. For example, metrics from
    sensors, a camera image etc.

  + `step(action)`: Each time step, the agent chooses an action, and the
     environment returns the observation, the reward and the state of the
     problem (done or not). 

  + `get_reward(action)`: The reward that the agent receives as result of their
     action.
        
  + `is_done()`: whether it’s time to reset the environment again. Most (but
     not all) tasks are divided up into well-defined episodes, and done being
     True indicates the episode has terminated. For example, if a robot has
     the task to reach a goal, then the done might be happen when the robot
     "touch" the goal. 
        
  + `reset()`: Used to reset the world to the initial state.

In order to set up a task in Deepbots is necessary to understand intention of
the OpenAI gym shared environment. According to OpenAI gym documentation, the
framework follows the classic “agent-environment loop”. "Each time step, the
agent chooses an action, and the environment returns an `observation` and a
`reward`. The process gets started by calling `reset()`, which returns an initial
`observation`." 

<p align="center">
    <img src="./doc/img/agent_env_loop.svg">
</p>

Deepbots follows this exact same agent-environment loop with only difference
that the agent, which is responsible to choose the action, runs on supervisor
and the observations are received by robot. The goal of deepbots framework is to
hide this communication to the user, especially to those who are familiar with
the OpenAI gym. More specific, `SupervisorEnv` is the interface which is used
from Reinforcement Learning algorithms and follows the OpenAI Gym environment.
Deepbots frameworks provides different levels of abstraction according to the
user need. On the other hand, goal of the framework is to provide different
wrappers for a great amount of robots. Currently, the communication between
`Supervisor` and `Robot` is achieved via `emitter` and `receiver`. 

<p align="center">
    <img src="./doc/img/deepbots_overview.png">
</p>

Precisely, on one hand `emitter` is the entity, which is provided by webots,
that broadcasts a message to the world. On the other hand, `receiver` is
the entity that is used to receive the messages came from the world.
Consequently, the agent-environment loop is transformed accordingly. Firstly,
Robot uses their sensors to retrieve observation from the worlds and in turn
uses the emitter component to broadcast those observation. Secondly, Supervisor
receives the observations via receiver component and in turn, agent uses them to
choose a action. It should be noted that the observation agents uses might be
extended from the supervisor. For example, a model uses lidar sensors from the
robot but also the euclidean distance between robot and an object. As it is
expected, the robot does not know the euclidean distance, only supervisors
does. 

<p align="center">
    <img src="./doc/img/workflow_diagram.png">
</p>

### Abstraction Levels

Deepbots framework have been created mostly for educational proposals. The aim
of the framework is to make easier Deep Learning implementations in webots.
Specifically, we can consider deepbots as an wrapper among webots and OpenAI gym
interface. For this reason there are multiple levels of abstraction. For
example, user can choose if they want to use CSV emitter/receiver or if they
want to make a from "scratch" implementation. In the top level of the
abstraction hierarchy is the `SupervisorEnv` which is the OpenAI gym interface.
Below that level are the actual implementation. Those implementation aims to
hide the communication between `Supervisor` and `Robot`. On the other hand, the
`Robot` has also different abstraction levels. According to their needs, user
can choose either to process from scratch the messages which is receiver from
the supervisor or use the existing implementations.


