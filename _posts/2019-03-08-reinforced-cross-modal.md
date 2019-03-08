---
title: "Reinforced Cross-Modal Matching and Self-Supervised Imitation Learningfor Vision-Language Navigation"
categories:
  - paper-notes

tags:
  - [DRL, VLN, cross-modal, CV, NLP]
author_profile: true
toc_sticky: true
---
<a name="2daa7c03"></a>
# The task and the challenges
Vision-Language Navigation is the task of navigating an agent inside real environments by following natural language instructions. The target should be inferred from the instructions, and the navigation process is defined in the instructions, so the agent must align what it sees and what the instructions suggest.


![VLN.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551764795755-1e2ea8ac-9e52-44c0-86ee-a328f563791e.png#align=left&display=inline&height=484&name=VLN.PNG&originHeight=484&originWidth=661&size=440373&status=done&width=661)

There are three challenges:
1. the cross-modal grounding  

    In the local scene, the agent needs to ground the instruction, i.e., identify the specific part of the instruction that inform the agent an act. And also, the agent needs to match the whole instruction with the visual trajectory in the global temporal space.
2. the ill-posed feedback
    
    The agent should both follow the instruction to act and reach the final target. However, the 'success' reward is provided only when the agent reaches the target position, making the feedback ill-posed if without additional rewards, i.e., multiple solutions leads to the success reward.
3. the generalization problem
    
    The agent's learned policy should also generalize to unseen environments.

In this work, roughly speaking, they give solutions to all of the above by:
1. use attention between visual clues and textual clues, and design a matching critic that measures the correspondance of the trajectory and the instruction.
1. combine the relative distance reward and matching critic as intermediate rewards.
1. a self-supervised imitation learning approach that replay its own best experience.

<a name="dd2f4022"></a>
# The approach
<a name="858ec7dd"></a>
## 1. problem formulation

| state | panoramic view, image patches from multiple viewpoints |
| --- | --- |
| action | orientation (heading+rotation) |
| reward | success if reaches a ball around the target location |
| goal | a target location indicated by the instruction |

The mission is to reach a target location indicated by a given instruction, from a given initial state, and execute actions that follows the instruction.

<a name="965a1b65"></a>
## 2. model
<a name="6c3f7ad3"></a>
### 2.0 model overview

![ov.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551782411219-51569101-d6d7-4189-bcb4-f294008f9a60.png#align=left&display=inline&height=309&name=ov.PNG&originHeight=309&originWidth=597&size=36413&status=done&width=597)<br />Given the initial state and the natural language instruction (a sequence of words), the reasoning navigator learns to perform a sequence of actions, which generates a trajectory τ, in order to arrive at the target location starget indicated by the instruction. The navigator interacts with the environment and perceives new visual states as it executes actions. The Matching critic forms an additional reward signal during reinforcement learning. 
<a name="db6c7142"></a>
### 2.1 cross-modal reasoning navigator
![actor.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551766205766-2cccdd1e-02db-4e19-8a46-3f57a27634c2.png#align=left&display=inline&height=444&name=actor.PNG&originHeight=444&originWidth=663&size=106609&status=done&width=663)<br />The first module is the trajectory encoder that is structured as an LSTM. It takes in the trajectory as a sequence of state-action pairs, and produces a history context vector at each time steps, which is supposed to relate with the visual memory.<br />![hist.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551766597241-1aca74d7-6a2d-4273-8577-d3a8c3f51fe2.png#align=left&display=inline&height=65&name=hist.PNG&originHeight=65&originWidth=370&size=6134&status=done&width=370)<br /><br />As for the inputs, the action is the action vector, while what is supposed to represent the state is a soft-attention-weighted visual vector. This vector is obtained by attending the multiple visual viewpoints to the last history context vector.<br />
![vis_atn.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551766648287-a9599c9e-1b5c-4230-8d49-7e22fb8114b9.png#align=left&display=inline&height=144&name=vis_atn.PNG&originHeight=144&originWidth=504&size=18032&status=done&width=504)<br />The attention is computed in the form of dot-product attention, that is, linearly transforms both visual clues and history context, and dot product to compute the attention score, which is then used as the unnormalized attention weight for the visual clues.

The history context enables memory of the visual contexts that recognizes the current status. On top of this, they further propose to identify which words or sub-instructions to focus on next. Via a language encoder LSTM (pretrained?), they first obtain textual features for each word in the instructions, and then compute a soft-attention-weighted textual context, which is conditioned on the history context (the visual memory).<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/278890/1551767133698-5651be85-b88b-453b-90dc-9763201f0c66.png#align=left&display=inline&height=51&name=image.png&originHeight=64&originWidth=372&size=9744&status=done&width=298)

On top of this textual context, they further compute a soft-attention-weighted visual context, by attending multiple visual clues to the textual context.<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/278890/1551767315465-1d4b4102-deb9-4cb3-a764-01db1532165a.png#align=left&display=inline&height=56&name=image.png&originHeight=70&originWidth=427&size=11060&status=done&width=342)

Finally to propose an action, the three context vectors: 1) history context, 2) textual context, 3) visual context are concatenated, with each navigable direction embedding to calculate a probability distribution over each direction via a bilinear dot product.<br />![act.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551767627958-369f1254-1fa2-40fe-9b2f-1d8ae242f177.png#align=left&display=inline&height=70&name=act.PNG&originHeight=70&originWidth=550&size=12174&status=done&width=550)<br />Note that the navigable direction embedding is actually the action embedding that concatenates: 1) the viewpoint image feature from that direction, 2) orientation: heading and elevation, which gives 4-d vector.<br />(So the action space is discrete??)

To summarize:
1. the multiple viewpoints (visual features) are soft-attention-weighted w.r.t. the last history context
1. the history context is produced by the trajectory encoder, representing trajectory memory
1. the textual context is obtained by soft-attention of textual features w.r.t. the history context
1. the visual context is obtained by soft-attention of visual features w.r.t. the textual context
1. the action is the soft-attention-weights of action embeddings w.r.t. the three contexts.

<a name="b9dc276d"></a>
### 2.2 cross-modal matching critic
![cri_model.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551769196930-99af4db7-92bf-49ba-8595-3041b4f27f98.png#align=left&display=inline&height=246&name=cri_model.PNG&originHeight=246&originWidth=628&size=28110&status=done&width=628)<br /><br />To encourage the global matching of the executed trajectory to the instruction, an intrinsic reward is proposed. This is done by an attention-based seq2seq language model, which encodes the trajectory and computes the probability distributions of generating each word of the instruction. This part is pre-trained with a instruction-trajectory dataset.<br />![cri.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551769110991-1c8c09be-a1a7-41f5-97ba-fe21efa99cad.png#align=left&display=inline&height=82&name=cri.PNG&originHeight=82&originWidth=395&size=8261&status=done&width=395)<br />(is each word independently computed?)

<a name="3f429ded"></a>
### 2.3 Learning
**phase 1: imitation learning**<br />Warmstart the agent a relatively good policy with a supervised MLE loss (behavioral cloning loss), which maximizes the cross entropy between the ground truth action and the one given by the policy.<br />![imi.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551770531995-c1893d69-7491-4084-83d3-c08127dd9cf7.png#align=left&display=inline&height=59&name=imi.PNG&originHeight=59&originWidth=299&size=6294&status=done&width=299)<br />

**phase 2: reinforcement learning**<br />The reward in this phase is defined two-sourced. 
1. extrinsic reward

a) the relative navigation distance: the reduced distance to the target after executing the action under the state.<br />![image.png](https://cdn.nlark.com/yuque/0/2019/png/278890/1551770724837-714454bb-ef4d-418e-a730-6c297f4663ca.png#align=left&display=inline&height=42&name=image.png&originHeight=53&originWidth=636&size=12892&status=done&width=509)<br />b) success criterion: counted as success if the distance to the target is within a threshold.<br />![suc.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551770839301-aa4b12aa-e2be-46fc-b899-22fcb633a6bf.png#align=left&display=inline&height=51&name=suc.PNG&originHeight=51&originWidth=377&size=7167&status=done&width=377)<br />Note that the second one is only activated in the last step of a trajectory. They use the cumulative version with a discount factor (decrease the importance of future rewards)<br />![dis_cum.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551771047428-f5eae7b3-8c1e-41cf-bcca-e3cdefe686f1.png#align=left&display=inline&height=150&name=dis_cum.PNG&originHeight=150&originWidth=604&size=17998&status=done&width=604)
1. intrinsic reward

exactly the probability given by the critic, called cycle-reconstruction reward.

With the two types of rewards, the RL loss is defined as an expectation of advantages over the trajectories given by the policy.<br />![rllos.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551771399530-3c3d44b9-146d-4824-b636-4903e63acaed.png#align=left&display=inline&height=109&name=rllos.PNG&originHeight=109&originWidth=647&size=17884&status=done&width=647)<br />delta is a hyperparameter, b is a baseline to reduce variance. (but not sure how to get the baseline, they seem to fit a value function given the trajectory encoder's hidden state). Based on REINFORCE can this loss's gradient be computed.<br />![polgrad.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551771777984-bb6f3fce-36d5-4562-ba46-ac965bbcdd11.png#align=left&display=inline&height=52&name=polgrad.PNG&originHeight=52&originWidth=352&size=6666&status=done&width=352)

**phase 3: self-supervised imitation learning**<br />They further discuss about the scenario where the agent is allowed to explore unseen environments. The agent generates demonstrations by itself (based on the policy learned with past experience). Those solution with the best cycle-reconstruction reward is defined as demonstration and stored in a replay buffer.<br />![sil.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551772632045-f2df4b88-f6bb-45b7-9b8e-55cdd4cc82aa.png#align=left&display=inline&height=66&name=sil.PNG&originHeight=66&originWidth=326&size=6521&status=done&width=326)<br />This is the ordinary behavioral cloning loss that maximizes the likelihood of the demonstrations. However it is confusing that the author claims the above is equivalent to the following.<br />
![sil2.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551772742671-07ccc0aa-c5ba-4366-8095-954096debf71.png#align=left&display=inline&height=53&name=sil2.PNG&originHeight=53&originWidth=324&size=6453&status=done&width=324)<br />This seems to be reduced from the previous RL loss, which is not supposed to use as the loss for imitation learning.

<a name="Experiments"></a>
# Experiments
There is a Room2Room dataset for VLN in real 3D environments, which consists of 7189 paths, 21567 human-annotated instructions, with an average length of 29 words.

The dataset is split into training, seen validation, unseen validation and test sets. Two scenarios is designed for evaluation:
1. testing scenario.

    test the agent in unseen environments in a zero-shot fashion. (no prior exploration on the test set)
1. lifelong learning scenario.

    allow the exploration in unseen environments but without ground truths.<br />

The training pipeline:
1. resnet-152 as image feature extractor (without finetuning)
1. GloVe word embeddings as the language encoder (with finetuning during training)
1. pretrain a matching critic with human demos (instr-traj dataset)
1. warm start the policy via imitation learning
1. switch to RL
1. (optional) self-supervised imitation on unseen environments.

The VLN challenge uses five metrics: 
1. Path Length (PL), 
1. Navigation Error (NE), 
1. Oracle Success Rate (OSR), 
1. Success Rate (SR), 
1. Success rate weighted by inverse Path Length (SPL).

![metrics.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551777771878-6b23df4f-767d-4cdf-95e7-0bfc390fbee9.png#align=left&display=inline&height=165&name=metrics.PNG&originHeight=165&originWidth=664&size=49773&status=done&width=664)

<a name="2f4b7894"></a>
## 1. Comparison with SOTA
![res1.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551777895569-8b341286-9f46-4255-8088-f5bb3450bc38.png#align=left&display=inline&height=405&name=res1.PNG&originHeight=405&originWidth=680&size=81580&status=done&width=680)<br />
<a name="9330bffd"></a>
## 2. Ablation

![res2.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1551777920348-1d4b034e-f91d-401e-b70c-a5a5f0511fec.png#align=left&display=inline&height=227&name=res2.PNG&originHeight=362&originWidth=1188&size=92857&status=done&width=746)

Update:<br />Our final conclusion is that the baseline is strong enough, making the proposed modules not that effective. As seen in the table, decrementally reducing module does not affect the performance significantly. Another conjecture is that the dataset is still relatively small, making the model overfit.
