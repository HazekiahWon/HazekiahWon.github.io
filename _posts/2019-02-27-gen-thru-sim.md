---
title: "Generalization through Simulation:  
Integrating Simulated and Real Data into Deep Reinforcement Learning  
for Vision-Based Autonomous Flight"
categories:
  - paper-notes

tags:
  - [DRL, sim2real]
author_profile: true
---
# Current issues and motivation
Deep learning requires large quantity of data to achieve performance. DRL has the following challenges:  
1. the policy is meant to be used in the real-world settings, but collecting real-world data is expensive.   
2. policy learning is often performed in simulated environments but there are systematic
differences between simulated rendering and the real-world, because some factors are hard to model.

As a result, policy learned using only real-world data easily overfits while policy learned using only simulation data does not transfer to the real-world settings. 

What are the benefits for policy learning with two types of data?  
1. simulation data is of large quantity.  (but fails at learning dynamics)  
2. real-world data better reflects the dynamics. (but fails at its quantity)

These motivates the author to combine the two kinds of data.

# idea
Since the quantity of simulation data facilitates effective learning of visual features, and 
the quality of real-world data facilitates effective learning of the dynamics. The idea is to 
separate the decision making into two parts: 1) perception 2) control.

We expect to learn a perception module that can easily transfer from simulation to real-world, although it is only trained with simulation data. Besides, a control system is required 
to identify the incentives of the task and make decisions (actions). We stick to the real-world 
data to learn this control system.

# model and learning
## the formulation
The states are in the form of image. The action space is continuous. A discounted factor is used for computing cumulative returns. We assume the access to a simulator (infinite amount of simulation data) and a small amount of real-world data.

The learning is divided to two phases -- use simulation data or real-world data.

## the simulation phase
In the simulation phase, our focus is to learn effectively a perception module, in a typical 
deep q-learning style. The author refers the features produced by the perception module as task-relevant features, and compare this with supervised or unsupervised learning of task-agnostic features.

(Personally, I think the q-network is not in use during testing. So the 
q-network acts more like a validator for the perception module, i.e., to see if the perception 
module has come up with essential visual features to support a correct q-prediction (prediction of the cumulative rewards). In other words, if the perception module produces 
irrelevant features, the q-value, based on the given state and action, will be non-sense.) 

Also note that the typical q-learning requires a discrete action space to take the max among 
all possible actions. The author has resorted to two approaches to approximate a solution with stochastic optimization methods.
![]({{site.baseurl}}/assets/images/20190227/model.png)
## the real-world phase
In the real-world phase, the author transfers knowledge from simulation via the perception 
module. However, they claim to fix the parameters of the perception module without further finetuning on real-world data, suggesting a result of overfitting. 

For the control system in the real-world phase, the author sparkly proposes an action-conditioned reward predictor that is meant to model the dynamics, not of the environment but of the reward. 
![]({{site.baseurl}}/assets/images/20190227/pred.png)
(This is surprising but kinda make sense because, the choice of action is guided by the cumulative returns in the q-learning, and the dynamics of environment is inherently reflected by the reward, because the rewards tell us if the following states are good or bad.)

The reward predictor is similar to the q-value in the sense that they both relate with rewards, but differs in the form of rewards. The predictor ingests a state and an action sequence in a future horizon, and outputs a reward sequence of that horizon, while the q-value is the summation of the reward sequence.

The architecture for the predictor is simple. It consists of :
1. a perception module, which is directly set the same as in the simulation phase, as aforementioned.  
2. an action module, learned with the real-world data.  
3. a value module, which combines the hidden vectors given by the perception module and the action module (state and action), learns its parameters on the real-world data, and outputs a single-step reward.   

By the way, the q-network takes a similar architecture as such.

Using such architecture, the predictor can give a reward sequence by adopting an RNN. In other words, the value module takes the features of the same current state and time-varying action **iteratively** to produce a sequence of rewards along the horizon.
![]({{site.baseurl}}/assets/images/20190227/loss.png)
The learning of the predictor is accomplished in a supervised style. We minimize the summation of the squares over the horizon on the real-world dataset. Besides, the predictor is used in a model-based fashion -- with CEM and MPC. CEM fits a distribution which approximates the region of higher region, it is the planner that gives the action. MPC here acts as a controller. It runs the planner to obtain an action sequence but only executes the first at each time step. This fashion has nothing to do with learning.

## questions
However,  currently I am confused about how CEM is combined with the predictor. CEM fits a distribution in the action space, which is the region of higher rewards, and gives action sequence via sampling during testing. So my question is how is the predictor used during testing, since CEM itself already outputs the actions.
![]({{site.baseurl}}/assets/images/20190227/ques.png)