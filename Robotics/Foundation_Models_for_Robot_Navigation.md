# Foundation Models for Robot Navigation

**-- NAVER LAB europe**

Robotics has traditionally addressed navigation with mapping and planning based on SLAM (simultaneous localization and mapping). Our research is based on a different *Ansatz*, namely to learn <u>a foundation model for navigation with end-to-end agents trained on large amounts of heterogeneous data</u>. We want a robot to work out of the box in its new environment without any special preparation, <u>no prior scans or existing external localization system.</u>

## Towards a Foundation Model for Robot Navigation

**Difficulty with robotic agents training**: need to be **trained on diverse 3D scenes or navigation episodes** both of which are relatively hard to come by.

==> solution: Training in simulation (can perform training without manual supervision of real robot hardware and avoiding the risk of breaking material in the early stages of training.)

However, **sim2real gap**: the difference between simulated environments and real physical environments. These differences have multiple aspects:

- *visual differences*
- *collision models and haptics*: differences in agent dynamics, in environment dynamics and in agent-environment intearctions.

To learn agents which perform robustly in real environments, we deploy our trained models to real robots and evaluate them in challenging setups on a daily basis. We research improving navigation robustness and speed in real environments by:

- reliably simulating realistic agent behavior during training
- leveraging large-scale geometric foundation models
- designing and implementing efficient training losses.

## Reliably simulating realistic agent behavior during training

Previous attempts to address the sim2real gap have focused on only the visual gap (as in Figure 1) and, although progress has been made by training on large amounts of data and performing domain randomization during training, it’s come at the cost of navigation speed. Typical agents are trained to take small steps and/or to stop for observations, avoiding the problem of simulating realistic agent dynamics during training.

In our work [[1](https://openaccess.thecvf.com/content/CVPR2024/html/Bono_Learning_to_Navigate_Efficiently_and_Precisely_in_Real_Environments_CVPR_2024_paper.html)] and [[6](https://europe.naverlabs.com/research/publications/reasoning-in-visual-navigation-of-end-to-end-trained-agents/)], we target training agents capable of navigating swiftly in real environments and to reliably model agent dynamics, we minimize the sim2real gap not only in sensing, but in actuation. Our agent predicts velocity commands while in motion, resulting in a navigation speed of 0.7m/sec. The resulting delay between sensing and actuation requires the agent to anticipate motion, which it learns in simulation from a realistic motion model. We create a second-order dynamical model from real robot rollouts and expert actions, which approximates the physical properties of the real robot.

## Leveraging large-scale geometric foundation models

The perception module of a navigating robot needs to detect navigable space and obstacles, exits and goals and estimate its relative pose with respect to them. All of these subtasks require a certain form of visual reasoning. The detection of free navigable space, obstacles and exits boils down to taking decisions based on appearance cues but the detection of goals requires solving a partial matching task, which in essence is a challenging visual correspondence problem. 

## Designing and implementing efficient training losses.

Learning good representations of the scene within which the robot navigates is important, but being able to actually determine how useful a representation actually is can lead to greater efficiency. You can compare it to walking into an apartment which you explore and observe then being blindfolded and told to go to a specific place (eg. *“the table at the right corner of this room”*). If you’re able to get there without bumping into anything, then the mental representation of the scene you’ve kept in your memory has been very useful. This is the premise behind our new training method for navigation [[4](https://openreview.net/forum?id=8HCARN2hhw)] where the objective is to assist an end-to-end trained agent in learning a compact and useful representation of the history of observed images through an auxiliary loss. While known auxiliary losses for robotics are often based on learning some form of reconstruction of the scene, we raise the question of whether these representations should be based on precise reconstruction and whether an excess amount of precision during learning in simulation will potentially lead to an overzealous agent, whose excess capabilities do not transfer well to the real world.

We’ve therefore been learning a latent spatial representation with the goal of avoiding spending training signals on learning to explicitly reconstruct the scene in unnecessary detail. The representation built by the agent through the integration of visual observations over time while navigating is passed to a blind auxiliary agent (mole), which is trained to use it as its sole information to navigate to a batch of intermediate subgoals (see Figure 5). Optimizing over the success of the blind auxiliary agent leads to an actionable representation and it can be done independently of the downstream task. We’ve shown that this leads to a significant performance gain when the agent is deployed from simulation to a real environment.

## Summary

Our research focuses on building a foundation model for navigation by carefully compromising between increasing training data and improving models, algorithms and inductive biases to learn robust and efficient models from the amount of data we already have available; hybrid training in simulation and training in the real world. Ongoing work focuses on further improving scaling properties, acquiring and including real data and learning from web-scale robotics data. This requires cross-embodiment and multi-task agents [[5](https://openaccess.thecvf.com/content/CVPR2024/html/Marza_Task-Conditioned_Adaptation_of_Visual_Features_in_Multi-Task_Policy_Learning_CVPR_2024_paper.html)] to integrate heterogenous sources.
