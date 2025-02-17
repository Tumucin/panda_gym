# Self-Collision Aware Reaching and Pose Control in Large Workspaces using Reinforcement Learning
![](https://github.com/Tumucin/DeepRL-Pose-Control/blob/PoseControlConda/panda_gym/robots.gif)
## Table of Contents: 
- [ABOUT](#about)
- [CITATION](#citation)
- [INSTALLATION](#installation)
- [USAGE](#usage)
- [AGENTS](#agents)
- [TRAINING](#training)
  - [CASE1](#case1)
  - [CASE2](#case2)
  - [CASE3](#case3)
  - [CASE4](#case4)
  - [Curriculum Learning](#curriculumLearning)
    - [Curriculum Learning without Considering Self-Collisions](#curriculumLearningwithoutConsideringSelf-Collisions)
    - [Curriculum Learning with Considering Self-Collisions](#curriculumLearningwithconsideringSelf-Collisions)
  - [Reward Function Design](#rewardFunctionDesign)
- [EVALUATION](#evaluation)
- [SWITCHING](#switching)
- [PRE-TRAINED MODELS](#preTrainedModels)
## ABOUT
- This repository contains the official implementation of the research paper titled **"Self-Collision Aware Reaching and Pose Control in Large Workspaces using Reinforcement Learning"**. You can find the paper [here](https://github.com/Tumucin/DeepRL-Pose-Control).
- The codebase is developed and tested using Python, along with the following libraries:
  - [stable_baselines3](https://github.com/DLR-RM/stable-baselines3)
  - [PyBullet](https://github.com/bulletphysics/bullet3)
  - [panda-gym](https://github.com/qgallouedec/panda-gym)

The development and testing were performed on an Ubuntu 18.04 system.
## CITATION
If you find this study useful, please cite using the following citation:
```bibtex
 @article{...,
   title = {Self-Collision Aware Reaching and Pose Control in Large Workspaces using Reinforcement Learning},
   author = {$author$},
   year = {2023},
   booktitle = {$booktitle$},
   url = {https://github.com/Tumucin/DeepRL-Pose-Control},
   pdf = {$pdf$}
 }
```
## INSTALLATION
- Clone this repository to your local machine
```setup
git clone https://github.com/Tumucin/DeepRL-Pose-Control.git
cd DeepRL-Pose-Control/panda_gym
conda env create environment.yml
conda activate DeepRL-Pose-Control
cd ..
bash install.sh
```
## USAGE

To use this package, follow these steps to update the necessary directory paths in the code and configuration files:
- Modify line 22 and line 24 in the **"panda_reach.py"** file, located within the **"DeepRL-Pose-Control/panda_gym/envs/panda_tasks"** directory, to match the folder locations specific to your setup.
- Navigate to the **"DeepRL-Pose-Control/panda_gym/envs/configFiles"** directory. Within this directory,  update the paths specified in the first six lines of each config file (ex:Agen5_Panda.yaml) to reflect the correct directory locations for your configuration.

## AGENTS
- **"Agent1:"** Direct RL training without the pseudo-inverse component and curriculum scheduling. This agent is referred to as the Learning Baseline.
- **"Agent2:"** Curriculum training of RL without the pseudo-inverse component.This agent is trained as the combination of **"Agent1:"** + Curriculum Learning. This agent is also referred to as the Learning Baseline.
- **"Agent3:"** Both the pseudo-inverse and the RL output is used without curriculum scheduling. This agent is referred to as the Hybrid agent that we proposed.
- **"Agent4:"** Both the pseudo-inverse and the RL output is used with curriculum scheduling. This agent is trained as the combination of **"Agent4:"** + Curriculum Learning. This agent is referred to as the Hybrid agent that we proposed.
- **"Agent5:"** Raw pseudo-inverse joint velocities, treated as a traditional baseline. It is important to note that this agent is not trained since there is no learning part in it. Referred to as the Traditional Baseline, it serves as a benchmark for comparison.

## QUICK EVALUATION 
To verify that the PyBullet simulator runs correctly, you should first execute the following command for Agent5.
Note that the code expects you to provide a pre-trained model for Agent5. However, Agent5 does not actually use this pre-trained model. Since Agent5 lacks a learning component, it functions as a traditional baseline utilizing the pseudo-inverse method.

```
python3 trainTest.py --expNumber 1 --configName "Agent5_Panda.yaml" --render True
```
## TRAINING
There are several possible combinations to train the agents. We will systematically examine each combination to gain a better understanding.
- **"CASE 1:"** The agent does not consider orientation at the target pose and neglects self-collisions. This implies that the episode continues even in the presence of collisions between the links.
- **"CASE 2:"** The agent considers orientation at the target pose but does not take self-collisions into account.
- **"CASE 3:"** The agent does not account for orientation at the target pose but actively considers self-collisions. In this case, the episode is terminated if a collision occurs between the links.
- **"CASE 4:"** The agent considers both orientation at the target pose and self-collisions.

The trained model, log files, and information about failed samples will be saved to the directory specified in the corresponding YAML files. Metric results will be recorded and saved in the metrics$expNumber$.txt file, where expNumber corresponds to the experiment number.
### CASE1
**[No Collision, No Orientation]**\
The training and evaluation procedures for Agents 1, 2, 3, and 4 are similar. The following command line uses Agent1 as an example. Please note that using the training command below performs both training and evaluation. Evaluation is conducted with 1000 random initial robot configurations and random target poses.
Please ensure that, for this setup, the values of the variables "addOrientation" and "enableSelfCollision" in the config YAML file are both set to False.
To train **"Agent1"**, which functions as a Learning Baseline without the Pseudo-inverse module, follow these steps:
```setup
# No Orientation, no collision
python3 trainTest.py --mode True --expNumber 1 --configName "Agent1_Panda.yaml"
```
### CASE2
**[No Collision, Orientation]**
- For this time, set the **"addOrientation"** variable in the config yaml files to True to consider orientation at the target pose. Additionally, set the **"enableSelfCollision"** variable to False.
```setup
# Orientation, no collision
python3 trainTest.py --mode True --expNumber 1 --configName "Agent1_Panda.yaml"
```
### CASE3
**[Collision, No Orientation]**
- For this instance, configure the **"addOrientation"** variable in the config yaml file to False to exclude orientation at the target pose. Additionally, set the **"enableSelfCollision"** variable to True.
```setup
# Orientation, no collision
python3 trainTest.py --mode True --expNumber 1 --configName "Agent1_Panda.yaml"
```
### CASE4
**[Collision, Orientation]**
- To incorporate both orientation and self-collision considerations during training, set the **"addOrientation"** and **"enableSelfCollision"** variables in the config yaml files to True.
```setup
# Orientation, collision
python3 trainTest.py --mode True --expNumber 1 --configName "Agent1_Panda.yaml"
```
### Curriculum Learning
#### Curriculum Learning without Considering Self-Collisions
Agents 2 and 4 employ curriculum learning during training. We move on to the next region if the mean absolute error in position and the mean norm of the joint velocities at the final state are below thresholds if collisions are not taken into account. The following command line setups the curriculum learning:
```setup
python3 trainTest.py --mode True --expNumber 1 --configName "Agent2_Panda.yaml" --maeThreshold 0.05 --avgJntVelThreshold 0.15 --evalFreqOnTraining 3000000 --testSampleOnTraining 500
```
In this configuration, the training progress is evaluated every 3 million training steps, utilizing 500 samples for evaluation. If the mean absolute error in position is less than 5 cm and the mean norm of the joint velocities is less than 0.15 rad/s, the training process proceeds to the next region.
#### Curriculum Learning with Considering Self-Collisions
In this configuration, episodes are terminated when there is a collision between the links. It's crucial to understand that the mean absolute error and mean norm of joint velocities metrics are not meaningful in this context. From this reason, we move on to the next workspace after a certain training timstep. Thus, the number of evaluation frequency on training is fixed when collision are taken into account. Since **"maeThreshold"** and **"avgJntVelThreshold"** thresholds can not exceed 10, they are set to 10. These thresholds can be adjusted to other values, such as 15 or 20. This adjustment ensures the next curriculum region will be added to the current workspace after 3 millions of training time-step.  
```setup
python3 trainTest.py --mode True --expNumber 1 --configName "Agent2_Panda.yaml" --maeThreshold 10 --avgJntVelThreshold 10 --evalFreqOnTraining 3000000
```
### Reward Function Design
It is possible to change the parameters of the reward function. This project uses the following reward function type to train an agent:
$`r(s, a)= \exp \left(-\lambda_{pos}\|(\delta x_{})\|^2\right)  -\frac{\lambda_{velocity}\|\dot{\theta}\|}{1+\|\delta x_{}\|} - \lambda_{coll}c + \exp \left(-\lambda_{ori}\beta_q^2\right)`$

To modify the reward function, you can make adjustments using the following arguments:
```setup
python3 trainTest.py --mode True --expNumber 1 --configName "Agent1_Panda.yaml" --lambdaErr 100.0 --velocityConstant 0.1 --orientationConstant 50 --collisionConstant 10.0
```
Please keep in mind that the provided code assumes that the algorithm will account for self-collisions and orientation. As mentioned earlier, ensure to set the **"addOrientation"** and **"enableSelfCollision"** variables in the config YAML files to True accordingly.

## EVALUATION
After completing the training procedure, you can evaluate the trained models to obtain metric results using the PyBullet simulator. The evaluation process includes using 1000 random initial robot configurations and random target poses. The trained model, log files, and information about failed samples will be saved to the directory specified in the corresponding YAML files. Metric results will be recorded and saved in the metrics$expNumber$.txt file, where expNumber corresponds to the experiment number as explained in the [TRAINING](#training) section.
```setup
python3 trainTest.py --expNumber 1 --configName "Agent1_Panda.yaml" --render True
```
## SWITCHING
We switch to raw pseudo-inverse control when the Euclidean distance between the end effector and the target position is below a threshold (i.e. only utilize $`\theta_{pinv}`$). It is important to note that this is only employed during inference and not during training.
If you want to implement the switch modification, set the **"switching"** variable in the config yaml files to True. 

## PRE-TRAINED MODELS
No Collision             |  Collision
:-------------------------:|:-------------------------:
![noCollision](https://github.com/Tumucin/DeepRL-Pose-Control/blob/main/panda_gym/noCollisionPretrained.png)  |  ![Collision](https://github.com/Tumucin/DeepRL-Pose-Control/blob/main/panda_gym/CollisionPretrained.png)

The figure above illustrates the pre-trained models corresponding to different cases and experiment numbers. These pre-trained models have been automatically downloaded into the repo during the installation process, as described in the [INSTALLATION](#installation) section. All metric results have been documented in the paper. To evaluate the performance of the pre-trained model, simply execute the following command:
```setup
# Model 891 uses hybrid agent (Agent4) without considering self-collisions and orientation
# Make sure to set the 'addOrientation' and 'enableSelfCollision' flags to False
python3 trainTest.py --expNumber 891 --configName "Agent4_Panda.yaml" --render True
```

```setup
# Model 1082 employs the Hybrid Agent (Agent4) while taking into account both self-collisions and orientation considerations
# Make sure to set the 'addOrientation' and 'enableSelfCollision' flags to True
python3 trainTest.py --expNumber 1082 --configName "Agent4_Jaco7dof.yaml" --render True
```

