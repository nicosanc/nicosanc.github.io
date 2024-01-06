---
title: "Deep Learning: "Introduction to Feed-Forward Neural Networks"
date: 2024-01-05
categories: [Machine Learning, Deep Learning]
---
## Introduction
Feed Forward Neural Networks, also known as Multi-Layer Perceptrons (MLPs), are a class of Artificial Neural Networks composed of sequential layers of neural units. 
These MLPs are structured as acyclic, directed, and fully connected graphs, extending from the input layer to the output layer.  Between these, one or more hidden layers exist, performing computations on the data at each node. 
Neural Networks excel as function approximators, capturing complex patterns in data, primarily due to each node's ability to perform non-linear transformations.
This non-linearity is crucial; if nodes only executed linear transformations, each layer would essentially capture identical information, rendering the network no more effective than a single-layer perceptron. 
Hence, MLPs learn hierarchically, with each sequential layer abstracting higher-level features from the patterns recognized by previous layers. In Deep Learning, 'depth' refers to the number of hidden layers in the model, while 'width' denotes the number of nodes
in a given layer. As the network becomes deeper and wider, its capacity to extract unique values from large datasets expands, attributable to the increased number of parameters (weights and biases) in its architecture. 

Each node in a Feed Forward Network is interconnected through weighted links, which are adjusted based on the model's output error. Gradient-based techniques, such as backpropagation, utilize Gradient Descent to modify these weights. In this context, gradients are the partial derivatives of the Loss Function with respect to the model's parameters. These derivatives indicate the direction and rate of change required to iteratively minimize the Loss Function. The Loss Function is a crucial metric that measures the error between the model's predicted output and the actual target values. Since the ultimate goal is to closely approximate a target function, Gradient Descent guides the model towards the point of minimum loss, known as the minima. This process involves gradually shifting the model parameters in the direction that steadily reduces the loss, thereby honing the model's accuracy in making predictions. The size of the steps taken at each adjustment is dictated by another pivotal metric called a **learning rate**. Too large a learning rate can lead to overshooting the minima and oscillating, while too low a learning rate leads to slow convergence or getting stuck on a local minimum. It’s important to clarify that it takes many iterations, called epochs, each iteration bringing the model closer to the desired accuracy. 

## Overfitting and how it’s combatted 

