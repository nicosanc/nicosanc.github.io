---
title: "Deep RL: Reinforce Algorithm"
date: 2023-12-26
categories: [Machine Learning, Reinforcement Learning]
---


## Introduction
My first real introduction into groundbreaking AI was in 2015 when I learned about AlphaGo, and the significance of its victory over the best Go player at the time. 
I was blown away by the fact that a computer somehow managed to develop winning behaviors in a game with an unfathomably large state space, or number of possible arrangements that the pieces can take on the board. 
I’ve been fascinated with the topic ever since and have decided to write a blog series explaining some popular algorithms and their significance in RL’s progression. 
These blogs are tailored mainly for people in STEM fields, since they require some basic understanding of topics like Deep Learning and Calculus. 
I will try to be sparing with technical jargon, and offer explanations that can be understood by people of varying levels of knowledge.

Reinforcement learning has numerous applications, including important use cases in Robotics, Autonomous Driving, and Financial Trading. Teaching a machine optimal behaviors by interacting with dynamic environments can help simulate real-world problems. 
For example, an autonomous vehicle can be trained by simulating millions of interactions on the road with stop signs, pedestrians, and other vehicles. 
Through trial and error, the system begins to learn from its mistakes and fine tunes its behavior to better avoid obstacles and appropriately adjust the vehicle’s velocity. 

Most RL problems can be boiled down to a baseline idea: Develop an optimal decision making process so that the agent’s actions can lead to the best possible outcome.

## Terminology
There’s some terminology that is imperative to understand the algorithms that I’ll be writing about in this series. 

**Agent** - The agent is the internal system we are training. We let the agent interact with the world around it, 
gaining feedback from its actions to slowly get better at making decisions. Think of it as a player in a game who starts playing a game with a blank map, 
and over time it learns to sketch out where it's been and how its actions led it to those new places. 

**Environment** - The environment is essentially the “world” or the external system that the agent interacts with. The environment provides rewards and states as information feedback so the agent can know 
where it stands and how its actions can affect the state of its world. The environment has an intrinsic state transition function which dictates the dynamics of the world, 
or how each interaction changes a state into a new state.

It is important to distinguish between the two different kinds of environments: **Discrete** and **Continuous**

Think of discrete environments as having a finite number of states and actions. A common example is chess, where there are only so many ways that pieces can be arranged at any given moment. 

Continuous environments, on the other hand, have an infinite amount of possible states and actions within a varying range of values. A real world example would be a robotic arm. The robot’s joints and limbs can rotate and extend within a continuous range of values. The topic of Inverse Kinematics helps make sense of how these joint angles are computed to get its end-effector to a desired position in space. 

**States** -  A state of an environment can be thought of as the collective information about the environment and the agent at a specific point in time. At any given point, the agent may leverage its current state to 
make a decision about what the best next state would be and how to get there. It does this by observing which action leads to a next state that maximizes the rewards it’s collecting throughout its journey. 

**Rewards** - Rewards are feedback points given to the agent by the environment every time it acts. The rewards serve as a rating system, dictating how good an action was given the state it was in. 
In the image below, we can see the feedback loop where an agent takes an action in the environment and is presented with a new state and a reward for reaching that state. 

![RL-feedback-loop](/assets/images/rl-feedback.png)

**Experiences** - An experience is the information gathered by an agent after a singular interaction. Each experience can be represented by the tuple (state_t, action_t, reward_t+1, state_t+1) 
which holds the current state, the potential current action, the reward gained from choosing that action, and the state it leads to. 

An episode is the sequence of experiences that start at an initial state and ends at a terminal state. In other words, an episode is an agent’s full collection of interactions from beginning to end, in the environment. 
A terminal state can be the goal of the game (like reaching a specific spot in a map), a fail state (like walking on a trap in a game), or a maximum number of steps. 
It is good practice to end an episode when a maximum number of steps is taken to avoid exploring infinitely, especially early on when the agent’s decision making is no better than random. 
The visual below shows an example of a simple environment’s terminal states, one which is a goal state (diamond) and another which is a fail state (trap). 

![gridworld-img](/assets/images/gridworld.jpg){: .center-image}

**Trajectories** - The history of experiences in a given episode. This means trajectories can be seen as a set of tuples representing these experiences and are usually leveraged by algorithms to calculate the expected cumulative return over many episodes. They map out the actions, states, and rewards of each time step from beginning to end. 

**Returns** - The Return can be thought of as the cumulative reward achieved during an episode. The goal of Reinforcement learning at its core, for most RL problems, is to maximize the return. A basic formulation of the Return is: $$G(t) = \sum_{t}^{T} {\gamma^{t}*r_{t+1}}$$, where gamma is a discount factor that ranges between 0 and 1. When gamma is closer to 0, it means the agent will prioritize more immediate rewards over potential future rewards. 
When gamma is closer to 1, we put high priority on long term rewards. This dictates whether an agent takes an action that seems immediately valuable while ignoring the possibility of something better later on, or an action that may not be immediately useful but leads to high rewards in the long term. Finding the sweet spot for gamma is important for defining the right return function for an RL problem.

**Objective** - The Objective Function is the function we want to maximize. It can be understood in many instances as the expected value of the Return function over many, many episodes. A basic formulation of the Objective Function is $J(\pi_{\theta}) = $The idea is that with enough practice, the agent’s average return will be high enough to cross a certain threshold we deem reasonable. Maximizing the objective function is 
the intrinsic purpose of the agent and to achieve this, it must develop an optimal behavior.

**Policy** - The policy of an agent is its behavior. Think of the policy as the agent’s mind that tells it how to act at any given state. It starts off as basically random, and over time improves its behavior given the returns achieved after many episodes. Our goal is to find an optimal policy that maximizes our objective function. 

**Exploration-Exploitation Tradeoff**- A key concept to consider in RL. Exploration defines an agent’s characteristic to explore states and actions that it has not previously seen before. Exploitation defines an agent’s characteristic to act based on experiences it has already gained. Excess emphasis on exploration can lead to inefficient learning, whereas excess exploitation hinders an agent from learning new and better strategies. It is good practice to deal with this tradeoff using adaptive strategies that adjust according to the environment’s dynamics and the agent’s experiences.

Now that we have a good grasp of the basic concepts of Reinforcement Learning, we can now dive into how agents actually form a policy. There are two main methods to developing a policy: **Value** Based and **Policy** based approaches.

## Learning Approaches

**Value based**  approaches are methods of achieving an optimal policy using a value function. Value functions estimate how “good” it is to be in a given state, where the “goodness” is essentially the expected cumulative return of being in that state. The value function aims to indirectly persuade the agent to make better decisions by providing the value of next states. It’s appropriate to see value based methods as forward thinking, meaning they try maximizing long term rewards by choosing the highest valued actions and states at any given point. 

There are two types of values computed by value functions:

- **State value** (V): The value of being in a specific state, choosing the optimal actions from that state onward
- **Action Value** (Q): The value of taking a particular action in a given state, also proceeded by optimal actions from that state onward

We can see that our policy is not directly learned when we use value functions. Instead we optimize this value function and can assume that an optimal value function will lead to an optimal policy. This means that value based methods indirectly form a policy since the agent’s behavior is based on a computed value function. On the other hand, a direct approach to forming a policy would involve explicitly using the experiences and rewards of a trajectory to directly tweak the behavior. 

## REINFORCE

The algorithm we are exploring in this blog is a **Policy Based** method. In other words, this algorithm directly learns a policy and updates that policy after each episode instead of learning a value function. In the context of Reinforce, our policy can be represented by a set of parameters \theta, where each update of those parameters is a direct update to the policy. This is a popular approach called Policy Gradient. 

Policy Gradient aims to solve the formulation max J(pi theta) = E [R(tau)], by using Gradient Ascent to nudge the policy closer to an optimal approximation. Gradient Ascent is useful because it provides us with the steepest positive rate of change for our function. This means that every time it updates, it will inch us closer to convergence at an optima. To do this, we need a function approximator that works well with complex or nonlinear data. Neural Networks are great function approximators, and many frameworks like PyTorch and TensorFlow allow us to automatically compute gradients and 
fine tune the parameters after each pass. The network will take in a state as input and will output the likelihood of choosing an action given that input. Something to note is that neural network optimizers aim to minimize a function, specifically a loss function. The objective of Reinforce is to maximize our objective function, so we multiply what is being differentiated by -1 to ensure we converge at a maxima. The formulation for parameter updates is as follows:

$$\theta = \theta + \alpha*\nabla_{\theta} J(\pi_{\theta})$$

Where $$\nabla_{\theta} J(\pi_{\theta}) = \sum_{t=0}^{T} {R_{t}(\tau)\nabla_{\theta}log\pi_{\theta}(a_{t}|s_{t})}$$

This may seem a bit convoluted but the idea is that the gradient of our objective function is equal to the average of many trajectories’ returns multiplied by the gradient of the log probabilities of sampling an action ‘a’ from a state ‘s’. The reason we work with gradients of probabilities is because our model wants to boost the likelihood of choosing actions that lead to high returns while lowering the likelihood of sub-optimal actions. We can assume that the gradients of the log likelihood are weighted by the return, so Gradient Ascent will tell the network how to update the parameters depending on 
how high or low the return was for a sampled action. The logarithm of the probability is used to stabilize the computed values since the gradients can sometimes be very small numbers. Logarithms also make computation simpler by turning products into summations. 

The Reinforce Algorithm differs from other policy gradient methods because it leverages Monte Carlo sampling. This means the algorithm only updates its parameters at the end of an episode. In other words, it computes the return using an entire trajectory rather than updating after each experience. This can expose the model to a wide range of possible combinations of state/action pairs but has some flaws that have to be acknowledged. 

Learning from entire episodes will inherently lead to a high variance for reasons like: (1) Randomness associated with choosing actions from a probability distribution, (2) Environments with stochastic transitions from one state to another, (3) Possibility of initial and goal states varying in different episodes - like different levels in a game. A common approach to lowering variance is to subtract the computed return by some type of baseline value. This helps normalize the rewards to stabilize the scale of each return. This can be the average over many returns, or it can be a value function. Approaches 
like Actor-critic, which combine policy and value based methods, compute an optimal value function to subtract from the return during updates. Another set back that Reinforce faces is low sampling efficiency. The model only trains off the trajectory it uses for a single episode so it does not hold a history of previous iterations. The trajectories are discarded once the return is computed and are not revisited. 

Another important factor to consider, as with many deep learning approaches, is finding an optimal learning rate for our policy network. Choosing a high learning rate can speed up convergence, but it can lead to overshooting the optima or even oscillating back and forth causing instability. On the other hand, a low learning rate can provide stability, but can lead to much slower convergence and even getting stuck on local maximas. Finding that golden learning rate is imperative. 

Finally, it is worth noting that policy gradient approaches are inherently explorative. This is due to the stochastic nature of these models, since the agents’ actions are sampled from a probability distribution. Reinforce’s exploitation comes from its gradual increases in probability for actions that lead to higher cumulative rewards. 

## Summary

Reinforcement learning is still relatively young as a field so there is plenty of room for growth and innovation. The exciting progress at companies like OpenAI, Google Deepmind, Anthropic, and Cohere are concurrent proof of the capabilities of not only Reinforcement Learning, but the field of Artificial Intelligence as a whole. I firmly believe that spending time learning about Reinforcement Learning in AI can provide valuable insight to how intelligent systems (like ourselves) can learn to process information. There are many complex, dynamic systems out there to explore so I recommend taking a look at the 
sources that helped me write this blog. Hopefully you can be inspired enough to become a new tech pioneer. Also, do not hesitate to leave any feedback by emailing me directly and adding the blog post in the subject area. Any and all critiques are welcome as they’ll help me to not only understand the material better but also to improve the quality of future blogs. I am working on adding comment sections with disqus, in the meantime. Thank you for reading!

## Cool Resources

Lil’log, a blog written by Lilian Weng who is a software lead at OpenAI for AI Safety. Her blog is full of awesome information, including an in depth look at Reinforcement Learning.
[Lil'Log learning notes](https://lilianweng.github.io/)

Deep Learning by Ian Goodfellow,Yoshua Bengio, Aaron Courville. This textbook gives a great rundown of the math and theory behind Neural Networks, and their different applications.
[Deep learning Book](https://www.deeplearningbook.org/)

Foundations of Deep Reinforcement Learning: Theory and Practice by Laura Graesser and Wah Loon Keng. This textbook focuses on the different popular Reinforcement Learning algorithms and provides coding implementations at the end of every chapter. 
They use the SLM Lab library in their implementations. 
[Foundations of Deep RL](https://slm-lab.gitbook.io/slm-lab/publications-and-talks/instruction-for-the-book-+-intro-to-rl-section)

Hugging Face Deep Reinforcement Learning Course. This course is what recently got me hooked on the topic again. It offers explanations of popular algorithms that are easy to digest. It also provides hands-on coding sections where you implement the algorithms using popular libraries like Gym, Stable Baselines, and CleanRL. 
You receive a Hugging Face certificate once you complete the course so I highly recommend it!
[Hugging Face Deep RL Course](https://huggingface.co/learn/deep-rl-course/unit0/introduction)


