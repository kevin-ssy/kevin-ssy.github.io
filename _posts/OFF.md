---
title: 'Understand and Using Optical Flow Guided Feature in the Wild'
date: 2018-7-20
permalink: /off/
tags:
  - research
comments: true
---

### Aim of this blog

This blog is to discuss and summarize the tricks and observations we got about the project, [Optical Flow Guided Feature(OFF)](https://github.com/kevin-ssy/Optical-Flow-Guided-Feature). OFF is a fast motion representation that could be easily embeded into any kinds of video based deep learning framework. Here we will not replicate the contents that we have illustrated in the paper. What we want to discuss here is some details that we did not mention before, and also some new tricks and observations that we have found after the paper has been accepted.

### Component Analysis for OFF

OFF is quite simple. There are only two components in OFF -- the spatial and temporal gradients. The spatial gradient, which could be easily obtained by a sobel operation, and the temporal gradient, which is equal to the feature difference, are fast to compute and encoded in a same equation with optical flow. We stack these two components together and feed them into a tiny ResNet. We did not analyze the contribution that each branch made to the final performance. Here we report it in the following table on the dataset UCF-101 split1. The notation *OFF_T* represents the performance when only temporal gradient is applied in the OFF.

| Modality | Accuracy |
|       :-:       |  :-:  |
|  RGB+OFF_T(RGB) | 89.8% |
|   RGB+OFF(RGB)  | 90.5% |

The above results give us a obvious conclusion: the temporal gradients play a key role in improving the performance. Though the spatial part is less important, it still provides 0.7% gain to the performance. This phenomenon is not hard to understand, as the CNN itself already holds the capability to extract the gradient information from the RGB images, the spatial network can definitly capture such information from the raw RGB frames, and thereby we remove the saptial branch away from the OFF Unit when the input modalities are motion representations like optical flow or RGB difference. According to our experiment, the removal of the spatial branch for temporal descripting modality will not result in an obvious drop in performance, but the speed could be much faster. If you want to quickly implement the OFF into your own framework, it would be better for you to test whether the temporal branch works in your architecture.

### Dealing with Overfitting Problem.

As UCF-101 and HMDB-51 are both relative small dataset, we spent lots of time struggling with the overfitting problem. Addition to the tricks including fix-BN and large-ratio dropout applied in TSN. We removed all the BN layers in our OFF sub-networks and also add large dropout on the spatial branch of the OFF unit to keep the network away from overfitting.

### Cosine Learning Schedule

Originally, we use ```multistep``` learning policy to train the model. However, again according to the overftting problem, the final convergence does not always mean the model has reach its global optimal, thereby we need to use the ```early stop``` strategy to solve this problem. However, it is unrealistic for us to test the whole dataset all the time along with the training process. This means that we need to test lots of models we obtained from the training offline to make sure on which iteration we can get the best, which is time-consuming. To solve this problem, we use ```cyclic cosine learning schedule``` as a new strategy for training. In this case, we can test the full test set everytime a cycle terminates, which allows us to monitor the fitting situation during training. According to our observation, the cosine learning schedule will not lead to any improvement in performance comparing with the multi-step policy.
