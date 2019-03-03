---
title: "Continuous adaptation via meta-learning in nonstationary and competitive envrionments."
categories:
  - paper-notes

tags:
  - [DRL, meta-learning]
author_profile: true
toc_sticky: true
---
<a name="bd8bc354"></a>
# Current issues and motivation
Most of the works deal with stationary dynamics, however the real-world often presents a nonstationary one, e.g. 
1. changes in the dynamics
1. changes of the objectives
1. multi-agent scenarios

To deal with such nonstationarity, most of the works resort to finetuning to the changed environment, which is not effective because
1. nonstationarity makes interactions to the environment limited, presenting a few-shot regime.
1. RL algorithms are sample-inefficient, and hence incapable of achieving good performance after finetuning.

Formulating the nonstationary dynamics as a sequence of stationary tasks, this work adapt the learning-to-learn framework.
> the agents meta-learn to anticipate the changes in the environment and update their policies accordingly. 


Besides dynamics being nonstationary, they further discuss about the multi-agent scenario.
> Multi-agent environments are nonstationary from the perspective of any individual agent since all actors are learning and changing concurrently.


This paper investigates the problem of continuous adaptation both in nonstationary single-agent and competitive multi-agent setting.

<a name="method"></a>
# method
The problem of continuous adaptation in nonstationary environments immediately puts learning into the few-shot regime: the agent must learn from only limited amount of experience that it can collect before its environment changes.  They re-derive MAML for multi-task reinforcement learning from a probabilistic perspective, and then extend it to dynamically changing tasks.

<a name="dbb9620d"></a>
## 1. a probabilistic model
The task is defined as a stationary dynamics, a MDP.<br />
![taskf.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551420046239-4649ad63-d8c4-45a7-a992-e0ffd197a9be.png#align=left&display=inline&height=183&name=taskf.PNG&originHeight=213&originWidth=870&size=78745&status=done&width=746)<br />The K-shot meta-learning for RL in a stationary dynamics (a task) is defined as follows.<br />
![adapt.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551420294068-8be3767a-dc85-4f57-8c74-3b70a3ced63c.png#align=left&display=inline&height=191&name=adapt.PNG&originHeight=221&originWidth=865&size=78939&status=done&width=746)<br />For a sampled task, its current policy is parameterized by theta. Sample k trajectories from the theta-policy under the current dynamics (task), and finally perform the gradient update. This update can be seen as an estimation (simulation) for the adaptation step during testing. After the update, we obtain phi-policy which should exhibit better performance.

Meta-learning aims to optimize for theta, from which the adaptation is performed and achieves good performance. So the next thing is to measure the performance of phi-policy -- the loss -- and differentiate it w.r.t. theta.<br />
![meta-loss.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551420764625-5a32500c-18cd-4c69-9a9c-188ef672e553.png#align=left&display=inline&height=114&name=meta-loss.PNG&originHeight=131&originWidth=860&size=38230&status=done&width=746)<br />The where-statement appears complicated because, the meta-objective is recast from 
![](https://cdn.nlark.com/yuque/__latex/3880a6e4e439e6a5081bec492868ac06.svg#card=math&code=L_T%28%5Ctau_%5Cphi%29&height=25&width=49) to ![](https://cdn.nlark.com/yuque/__latex/de492df6f2e15c005cf07996b4ea92c0.svg#card=math&code=L_T%28%5Ctheta%29&height=24&width=41). Lets make an anatomy of this term. tau-theta is first sampled, and then used as the conditionals for ![](https://cdn.nlark.com/yuque/__latex/3880a6e4e439e6a5081bec492868ac06.svg#card=math&code=L_T%28%5Ctau_%5Cphi%29&height=25&width=49). This is because, phi is obtained via a gradient update using tau-theta, and then tau-phi is sampled from the phi-policy. For the expectation, T is a single Monte-Carlo sample, tau-theta is k sample, and tau-phi be a single sample (?).

This can be better illustrated by the following expansion.<br />![expans.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551422298575-ff33db13-ed2a-4400-ac42-534afc154f7b.png#align=left&display=inline&height=278&name=expans.PNG&originHeight=326&originWidth=874&size=76758&status=done&width=746)<br /> 
With the log-likelihood trick used in policy gradient, the gradient can be obtained as follows.<br />![grad.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551422411522-42c97f52-a4d7-4aa7-8603-52705d9b4db2.png#align=left&display=inline&height=82&name=grad.PNG&originHeight=82&originWidth=693&size=14336&status=done&width=693)<br />
This expected loss can also adopt the optimization used in TRPO and PPO.

<a name="be6748dc"></a>
## 2. continuous adaptation via meta-learning
Under nonstationary dynamics, the task distribution is defined by the environment changes, and the tasks are sequentially dependent.
> Hence, we would like to exploit this dependence between consecutive tasks and meta-learn a rule that keeps updating the policy in a way that minimizes the total expected loss encountered during the interaction with the changing environment. 


They approach of meta-learn to anticipate the changes in the environment is based on the assumption that trajectories from current dynamics contain some information about the next dynamics.

The new objective for nonstationary dynamics is formulated as follows. The dynamics is modelled as a markov chain, where each task (stationary dynamics) is only dependent on the former one.

![nobj.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551423504077-7bc7ff56-3c31-4f1d-a7b2-9610a8ebb656.png#align=left&display=inline&height=91&name=nobj.PNG&originHeight=91&originWidth=369&size=7412&status=done&width=369)<br />
The loss is now defined between two consecutive tasks, as our final target is to adapt to the new dynamics with trajectories with the previous one from the former dynamics.
![consloss.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551423685027-ee006f81-549d-4868-a913-bdde1f53f7b4.png#align=left&display=inline&height=50&name=consloss.PNG&originHeight=50&originWidth=702&size=11153&status=done&width=702)<br />
The difference between this loss and the former one is that, phi-policy is the one for the next dynamics, based on the current dynamics and its theta-policy.

The k-shot multi-step adaptation from theta to phi is described as follows.<br />
![mult.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551423956660-b9b6847e-1c13-4673-a580-eb6bba056c5c.png#align=left&display=inline&height=129&name=mult.PNG&originHeight=129&originWidth=615&size=20025&status=done&width=615)<br />
Note that, when we plug this to the loss, both theta and alpha are the targets to optimize, i.e., learn the adaptive step size for each step.<br />
![grad2.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551424101267-56018f92-1384-434e-b31f-e8a29404af04.png#align=left&display=inline&height=103&name=grad2.PNG&originHeight=114&originWidth=826&size=20056&status=done&width=746)
> Note that even though the policy parameters, φi, are sequentially dependent (Fig. 1b), in (6) we<br />always start from the initial parameters, θ. Hence, optimizing LTi;Ti+1(θ) is equivalent to truncated backpropagation through time with a unit lag in the chain of tasks.

This suggests using a same starting point for multiple tasks to adapt from. This is based on empirical observations that a sequential update (from ![](https://cdn.nlark.com/yuque/__latex/83c4a0f64b5c308332e71b11e874d4cc.svg#card=math&code=%5Cphi_i&height=24&width=15) to ![](https://cdn.nlark.com/yuque/__latex/a3471b1c08790ffcaf9d397a5c886d15.svg#card=math&code=%5Cphi_%7Bi%2B1%7D&height=24&width=30)) tends to diverge.<br /><br /><br />
![theta-or.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551426620184-127b26da-12bf-4380-bd02-7aa58f43533d.png#align=left&display=inline&height=145&name=theta-or.PNG&originHeight=145&originWidth=417&size=11002&status=done&width=417)
> Note that computing adaptation updates requires interacting with the environment under πθ while computing the meta-loss, LTi;Ti+1, requires using πφ, and hence, interacting with each task in the sequence twice. 

This is also confusing because, from the above equations, we have to sample from both theta and phi and the intermediate policy parameters.

<a name="meta-train"></a>
### meta-train

![meta-train.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551427359107-91066f89-e405-48f3-b001-64d5173fc670.png#align=left&display=inline&height=431&name=meta-train.PNG&originHeight=431&originWidth=431&size=85941&status=done&width=431)
<a name="meta-test"></a>
### meta-test
the old dynamics is not usually available for the theta-policy to sample trajectories, so they use the current policy under current dynamics to sample trajectories via the importance sampling.<br />
![imp.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551429544427-55d55035-2870-4426-9b08-a0325ef94a48.png#align=left&display=inline&height=72&name=imp.PNG&originHeight=72&originWidth=678&size=13658&status=done&width=678)<br />
![meta-test.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551435251998-6dd9f3e6-124b-4a19-94b6-f7641d8c063c.png#align=left&display=inline&height=265&name=meta-test.PNG&originHeight=265&originWidth=429&size=52381&status=done&width=429)
