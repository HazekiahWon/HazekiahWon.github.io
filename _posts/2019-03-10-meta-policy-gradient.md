---
title: "Learning to explore via meta-policy gradient"
categories:
  - paper-notes

tags:
  - DRL
  - meta-learning
  - DDPG
  - exploration
  
author_profile: true
toc_sticky: true
---
<a name="bd8bc354"></a>
# Current issues and motivation
DDPG (deep determinstic policy gradient) is an actor-critic off-policy method that
1. learns both a value function and a parameterized policy,
1. extending the discrete q-learning to continuous action space
1. and uitilizing a replay buffer to expolit past experience and thus improve sample-efficiency

One issue for continuous action space is about exploration:
1. the common practice of epsilon-greedy is only opt for discrete action space
1. continuous action space is high-dimensional, making exploration more sophisticated.

Current solution for this issue is to use an uncorrelated Gaussian or a correlated Ornstein-Uhlenbeck process, which has drawbacks of being sub-optimal or misspecified and its hyper-parameters hard to tune.

Their solution suggests learning an additional exploration policy, that is on-policy, based on the improvement of the policy (and the value function) as the reward, and optimized via policy-gradient.

<a name="Approach"></a>
# Approach
<a name="3e8439cb"></a>
## 1. DDPG
It is not feasible to apply q-learning in continuous action space because of the hardship of computing the max over continuous actions. Actor-critic approaches resort to a neural network to propose one action, which is supposed to be the action-argmax of the q-function.

More specifically, DDPG maintains a determinstic policy that ingests a state and outputs an action, which is the optimal one that maximizes another q-network, which ingests a state and an action and outupts the estimate of the total reward.

During training, the q-network, same as prior deep q-learning methods, optimizes the distance between current q-value and a target value of a given state-action pair. The target value is obtained by Bellman equation.<br />![bellman.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552118985882-1f41cd05-a210-489c-9e9a-8cac7eda8712.png#align=left&display=inline&height=51&name=bellman.PNG&originHeight=51&originWidth=429&size=11881&status=done&width=429) <br />(Bellmain Equation)<br /><br />![tv.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552119022467-c3c21b2c-8386-4f8f-a5a2-07ea9d1fcbea.png#align=left&display=inline&height=28&name=tv.PNG&originHeight=28&originWidth=341&size=8180&status=done&width=341)<br /><br />![qupd.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552119030268-a68ed6da-1aad-4582-925b-f6660d54958a.png#align=left&display=inline&height=89&name=qupd.PNG&originHeight=89&originWidth=438&size=13045&status=done&width=438)

As to the policy network, the objective is directly maximizing the Q-value given a state and the action suggested by the policy network.<br />![polobj.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552119260716-09def7d4-385e-4ecc-b9b5-1efb3ac4a56c.png#align=left&display=inline&height=46&name=polobj.PNG&originHeight=46&originWidth=434&size=13483&status=done&width=434)<br />Here mu is used to denote the policy network because the policy network can be considered as outputing an action distribution just that it is wrapped with a dirac-delta.<br />![dirac.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552119410182-fbf7a138-9502-4557-85d0-c6c8f97efda5.png#align=left&display=inline&height=34&name=dirac.PNG&originHeight=34&originWidth=226&size=5735&status=done&width=226)<br />Via REINFORCE, we analytically obtain the policy gradient as:<br />
![polgrad.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552119458517-766c776e-f0ac-4c41-ac38-84d3ecc91428.png#align=left&display=inline&height=58&name=polgrad.PNG&originHeight=58&originWidth=528&size=14776&status=done&width=528)

As an off-policy algorithm, a critical factor to consider is exploration. In common practice, data into the replay buffer is obtained by adding noise to the deterministic action given by the policy.

<a name="1bb52aba"></a>
## 2. Learning to explore
This work proposes an adaptively learned exploration strategy which adopts the MAML framework.

Suppose there is an additional policy network, who itself is parameterized distribution, whose action is to sample from itself, and whose reward is the policy improvement. Similar to MAML, the algorithm is three-stepped.
1. Simulate. 

    (run the current actor pi to compute a total reward R of a trajectory)<br />sample data D0 from the current (exploration-policy's) distribution.<br />update the current actor pi to pi' using the sampled data D0.
1. Estimate.

    run the updated pi' and obtain data D1, and compute the total reward R'.<br />compute policy improvement delta=R'-R
1. Optimize.
    
    solve for the exploration-policy that maximizes the policy improvement, via policy gradient.
    
![algo.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552120612162-37d98be8-74de-4756-8be8-a1752577d33e.png#align=left&display=inline&height=677&name=algo.PNG&originHeight=677&originWidth=629&size=202139&status=done&width=629)

Next I would briefly describe how is the policy gradient for the exploration-policy derived. The objective is as follows.<br />![expobj.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552120727456-33d8971d-4e0e-48ea-8e49-56e5cbf5852b.png#align=left&display=inline&height=97&name=expobj.PNG&originHeight=97&originWidth=316&size=13836&status=done&width=316)<br />Then compute the policy-gradient.<br />![expgrad.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552120763883-55a4c25b-3664-4f78-b49f-05c16e73bab2.png#align=left&display=inline&height=59&name=expgrad.PNG&originHeight=59&originWidth=533&size=14752&status=done&width=533)<br />The probability of generating transition tuples can be expanded.<br />![trans.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552120891963-8dc91601-8855-43f7-b4e3-98f81a845589.png#align=left&display=inline&height=91&name=trans.PNG&originHeight=91&originWidth=485&size=15165&status=done&width=485)<br />Taking the log, the transition function is independent of the parameters, so this term is reduced to the following.<br />![simp.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552120864731-948f3a16-7049-428e-a193-d8d4e52eba4d.png#align=left&display=inline&height=99&name=simp.PNG&originHeight=99&originWidth=477&size=14976&status=done&width=477)

For the specific form of the exploration policy, the author proposes three variants:
1. a zero-mean noise with variance parameters to optimize is applied directly to the deterministic action
    ![expp1.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552121150262-c5ce2294-dba6-49e9-9bc0-b187817043f3.png#align=left&display=inline&height=34&name=expp1.PNG&originHeight=34&originWidth=257&size=7174&status=done&width=257)
2. a Gaussian distribution with mean represented by a neural network and a variance parameter
    ![expp2.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552121268551-af987457-e9c9-4f2e-b19c-68afd78e6770.png#align=left&display=inline&height=37&name=expp2.PNG&originHeight=37&originWidth=184&size=6292&status=done&width=184)
3. adding more information about the current Q-function or the Bellman residual error, on the basis of the second variant.

<a name="Experiments"></a>
# Experiments
Q: What is the arch of the independent exploration policy network?

The experimental results have shown that the second variant of the exploration policy is sufficiently good, with the third adding not significant performance increase.

<a name="5a46583d"></a>
# Update: comments
Meta in this paper is not what we understand and expect. In essence, I think this work proposes a loss function for the learning of an additional exploration policy, just that the loss function adopts a similar pipeline of MAML.





