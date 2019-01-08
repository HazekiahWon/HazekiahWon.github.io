---
title: "Lecture 6 Actor-Critic"
categories:
  - lecture-notes

tags:
  - DRL
  - CS294-112
  - notes
author_profile: true
---
This is a half-way-done lecture note for [CS294-112 Lecture 6](http://rail.eecs.berkeley.edu/deeprlcourse/static/slides/lec-6.pdf).

# p4
recapping policy gradients.

the gradient is computed on a sampling estimate of the original objective. The estimate is averaged across n trajectories and each T time steps.

'reward to go' is the sum over the return of a trajectory starting from time t, the inner sigma's variable.
# p5
If we interpret $\hat{Q}_{i,t}$ as the estimate of expected return of a trajectory starting with $(s_{i,t},a_{i,t})$, this estimate is obviously not accurate, because this is estimated on a single trajectory.

In this sense, we recover the **Q-function** given $(s_{t},a_{t})$. The above term is an unbiased estimate of Q-function but has high variance because the sampling consists of only one trajectory (i).
# p6
recall that a way to reduce variance is to add a baseline, and a typical choice for this baseline is to use the average return under current policy, equivalent to the concept of the value-function.

So with such a **value-function baseline**, we reduce the original 'reward to go' term to the **advantage function**.
# p7
now we know that the final objective that policy gradients use, is still a single-sample high-variance, though, unbiased estimate of the advantage term.
# p8
now we think of how to get a good estimate of V or Q or A.
As Q and A can all be represented by V, we first consider fitting the value function.

note that here, to represent Q with V, a single sample approximation is involved, make it unbiased but with high variance.
# p9-10
**policy evaluation** evaluates how good is a policy, but does not change the policy.
**Monte Carlo evaluation** is the same as what policy gradient does. Although the true form of this is, for any $s_t$, a multiple-sample estimate should be applied to estimate $V(s_t)$, this is impractical, because this means to sample several trajectories from the prescribed state $s_t$ and the simulator cannot be controlled to rollback or rollout to this state. 

This does not mean there will only be one trajectory in total. There are multiple trajectories to be sampled for the simulator, just that for any state in any trajectory, there is only one sample for the 'reward to go' -- the simulator does not afford to sample multiple times for every state in one trajectory.

There is variance loss since the 'reward to go' still follows a single-sample estimate, but this variance loss is still smaller compared with the case using one trajectory overall. The evidence is that although the same state cannot be sampled more than one trajectory, states across trajectories are similar, which approximates the case where for any state multiple 'reward to go' can be estimated.

Although the estimate used for the reward to go is the same with policy gradients, we use this estimate to fit the value function, not to compute gradients for the policy update.

The regression for the value function follows the ordinary supervised style.

To reiterate, the training data composed of multiple trajectories with oneway forward reward records, no rollback is involved.
# p11
This slide introduces **bootstrapped estimate**.

The only difference is that the regression target is not the sum over the real trajectory rewards. It kind of uses a difference equation, only $(s_t,a_t)$ is needed, the subsequent cumulative rewards is given by $V(\cdot)$ at the next state.

This form enjoys lower variance but is biased.

**I suddenly find that this form is the same as what we approximate for the Q-function based on the value-function.** As to the Q-function, $E_{s'}[V(s')]$ is replaced with directly $V(s')$. While for the bootstrapped estimate of the value-function, $E_{a}[r(s,a)+E_{s'}[V(s')]]$ is replaced with $r(s,a)+V(s')$.

# p12
examples of TD-Gammon and AlphaGo

# p13
batch-mode actor-critic algorithm:
1) generate samples (run the policy)
2) fit a model (i.e., the value function), via bootstrapped estimate or Monte Carlo Evaluation
3) evaluate the advantage function based on the fitted value-function.
4) improve the policy via gradient descent on the objective with the estimated (and fixed) advantage.
5) back to 1) except convergence.

The regression for the value function uses a supervised MSE loss with gradient descent.

# p14
If the episode length has no upper bound, a discount factor on rewards is required otherwise the value can get infinitely large.

For the bootstrapped estimate in a differential form, plug the discount factor before $\hat{V_\phi^\pi}(s')$.  The intuition for this is that getting rewards sooner is better. A typical value of $\gamma$ is 0.99.

This is like adding a dead state from which no real state and reward can be reached, with a probability of $1-\gamma$.

# p15
As mentioned previously, the Q-function is the same as the value-function regression target by approximation in the form, so when evaluating the advantage, there is $\gamma$ to plug before the estimated $V(s')$.

Next question is about where to plug the discount factor for the Monte Carlo policy gradients.

We have two options:
1) plug it in the 'reward to go' term as our intuition suggests.
2) derive the primitive formula again, with $\gamma$ plugged before each reward.

# p16
Although option 2 is the natural and rigid derivation, option 1 is used in practice because there is no such discount for remote future in option 1 as in option 2.

Option 1 is also seen as an approximation to the policy gradients of the expected return without discount. The discount is only added in the 'reward to go' term to reduce the variance of the remote future that may be noisy or uncertain.

# p17
Batch mode actor-critic with discount is almost the same as before, except that the discount plugged to the advantage estimate and the regression target.

Online actor-critic differ from the previous algorithm as follows:
1) the samples generated consist indeed of one transition
2) the regression for the value-function should use bootstrapped estimate only because no real data for the future trajectory is known in this case.



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MTQ2NjY1MzAsLTIwODg3NDY2MTJdfQ
==
-->