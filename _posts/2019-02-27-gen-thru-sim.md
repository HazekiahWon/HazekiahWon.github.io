---
title: "Generalization through Simulation:  
Integrating Simulated and Real Data into Deep Reinforcement Learning  
for Vision-Based Autonomous Flight"
categories:
  - paper-notes

tags:
  - [DRL, sim2real]
author_profile: true
toc_sticky: true
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
[//]:![]({{site.baseurl}}/assets/images/20190227/model.png)
![](https://hazekiahwon.github.io/assets/images/20190227/model.png)
## the real-world phase
In the real-world phase, the author transfers knowledge from simulation via the perception 
module. However, they claim to fix the parameters of the perception module without further finetuning on real-world data, suggesting a result of overfitting. 

For the control system in the real-world phase, the author sparkly proposes an action-conditioned reward predictor that is meant to model the dynamics, not of the environment but of the reward. 

[//]:![]({{site.baseurl}}/assets/images/20190227/pred.png)
![](https://hazekiahwon.github.io/assets/images/20190227/pred.png)

(This is surprising but kinda make sense because, the choice of action is guided by the cumulative returns in the q-learning, and the dynamics of environment is inherently reflected by the reward, because the rewards tell us if the following states are good or bad.)

The reward predictor is similar to the q-value in the sense that they both relate with rewards, but differs in the form of rewards. The predictor ingests a state and an action sequence in a future horizon, and outputs a reward sequence of that horizon, while the q-value is the summation of the reward sequence.

The architecture for the predictor is simple. It consists of :
1. a perception module, which is directly set the same as in the simulation phase, as aforementioned.  
2. an action module, learned with the real-world data.  
3. a value module, which combines the hidden vectors given by the perception module and the action module (state and action), learns its parameters on the real-world data, and outputs a single-step reward.   

By the way, the q-network takes a similar architecture as such.

Using such architecture, the predictor can give a reward sequence by adopting an RNN. In other words, the value module takes the features of the same current state and time-varying action **iteratively** to produce a sequence of rewards along the horizon.

[//]:![]({{site.baseurl}}/assets/images/20190227/loss.png)
![](https://hazekiahwon.github.io/assets/images/20190227/loss.png)
The learning of the predictor is accomplished in a supervised style. We minimize the summation of the squares over the horizon on the real-world dataset. Besides, the predictor is used in a model-based fashion -- with CEM and MPC. CEM fits a distribution which approximates the region of higher region, it is the planner that gives the action. MPC here acts as a controller. It runs the planner to obtain an action sequence but only executes the first at each time step. This fashion has nothing to do with learning.

They do not transfer the q-network, claiming that low-data regime is not fit for q-learning, and supervised learning of the predictor is more stable.

# experiemts

|item | value|
|-----|------|
|task| collision avoidance and navigation, 4min max duration.|
|object| NAV(crazyflie).|
|action| yaw rate (forward speed, height constant).|
|state| 4 grey-scale images (optical flow and height sensor, monocular camera).|
|simulation| Gibson simulator provides 3d-scanned environment, the quadrotor as point mass camera.|
|reward| -1 for collision, 0 for no collision.|
|horizon | 12 step = 4 Hz = 3 seconds into the future.|
|simulation data | 16 instances of q-learning with each a different environment. Up to 17m in total.|
|reality data | the simulation-learned q-network run for one hour, 14k in total.|

![experiment results][exp]

Note that the time until collision is represented in median with middle fifty range.
It is shown that:
1. 2 is worse than 1, maybe because q-learning overfits to reality data.
2. 3 is slightly better than 2, maybe because perception module generalize better if it does not finetune.
3. 4 shows progress, can serve as a baseline for ACRP.
4. 5 is better than 4, maybe because task-agnostic visual feature generalize.
5. 6 is better than 5, maybe because vae produces more accurate feature
6. 7 is better than 6, maybe because task-specific helps the task a lot more.

# questions
## 1. CEM with reward-predictor
However,  currently I am confused about how CEM is combined with the predictor. CEM fits a distribution in the action space, which is the region of higher rewards, and gives action sequence via sampling during testing. So my question is how is the predictor used during testing, since CEM itself already outputs the actions.

[//]:![]({{site.baseurl}}/assets/images/20190227/ques.png)
![](https://hazekiahwon.github.io/assets/images/20190227/ques.png)
## 2. learning of the predictor in low-data regime
A potential issue of the approach is how to ensure the accuracy of the predictor. Imagine a state occurs during testing but is not experienced by the real-world data, can the predictor also predict well. If not, the optimal control may definitely degrades.

## 3. why would the reward predictor better than q-learning with small amount of real-world data?
> However, Q-learning
used with deep neural networks typically only works with
large amounts of data [40], and can be noticeably unstable in
low-data regimes due to the Bellman bootstrap update, while
the action-conditioned reward prediction approach is more
stable because it reduces to standard supervised learning

This claims q-learning is unstable in low-data regimes, 
because we use the current Q-value to estimate a target value for the Q-function to optimize towards.
![](https://hazekiahwon.github.io/assets/images/20190227/bellman.PNG)

In this sense, the reward predictor is better because the target reward is right available in the real-world data.
However, I don't think this validate the issue in 2.

## 4. why would the reward predictor be better than an ordinary dynamics model?
> The action-conditioned reward predictor is oftentimes
advantageous compared to model-based methods when the
robot state s is high-dimensional—such as in this work, in
which we consider the state of the NAV to be the current
image because we do not have access to the underlying state
information—because learning dynamics models for image
sequences, while possible [41], is difficult because it requires 
predicting a complex and high-bandwidth sensory signal.

The point made here is that a dynamics model where images serve as the 
observations is hard to learn. This is correct, but is it possible to 
learn a dynamics model that define visual features as observations?

So the justification is not sufficient in my opinion.

## 5. is the visual feature so easily transferable from the simulation that finetuning is not even required?
> we will initialize the weights of the real-world policy’s visual
perception layers (Fig. 2 top) to the values of the visual
perception layers from the Q-function learned in simulation
(Fig. 2 bottom), and hold these perception layers fixed during
real-world policy training. Although these layers could be
further fine-tuned using the real-world data, we decided to
hold these layers fixed to prevent the real-world policy from
overfitting to the training data.

This is assuming that the observations in simulation are plausible to ones in real-world, 
which does not naturally hold true. Is the simulator (Gibson) so powerful as to mimic the 
real-world observations?

## 6. what is the definition of dynamics? Why is it true that simulation has unrealistic dynamics?
> Many of
the prior works in this area have focused on differences that
are irrelevant to the task; nuisance factors, such as variations
in visual appearance, to which the optimal policy should
be invariant. Such nuisance factors can be eliminated by
regularizing for invariance. However, some aspects of the
simulation, particularly in terms of the dynamics of the robot,
differ from reality in systematic ways that cannot be ignored.
This is especially important for small-scale aerial vehicles,
where air currents, drift, and turbulence are significant. In
principle, this mismatch can be addressed by fine-tuning
models trained in simulation on real-world data. However, as
we will show in our experiments, naïve fine-tuning with small,
real-world datasets can result in catastrophic overfitting.

A counter example, the variations in visual appearance, is used to illustrate that policy should 
be invariant w.r.t. such factors. The dynamics is claimed to be part of the systematic differences, 
which makes it hard for the policy to transfer from simulation to reality, e.g., air currents, drift, turbulence.

## 7. how is the learning with such sparse reward effective?
An explanation for the reward predictor being accurate is the reward function is quite simple.
However, if the reward being -1 most of the time (collision), how is it possible for the policy learning, 
like, get any useful training signal?

[exp]: {{site.baseurl}}/assets/images/20190227/exp.PNG