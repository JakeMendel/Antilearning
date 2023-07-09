
# Antilearning
Consider some task X, and a related task Y, with the property that by default, training a model to generalise well on task X naturally leads to the model being able to do Y as well. Under what conditions could I train the model on task X in a different way so that task Y is never learned, and how would I go about doing that?

## Some reasons why I think this is worthwhile
- It seems likely that in the future, we are going to want to train models to become good at some things (planning, consequentalist reasoning) which may naturally generalise to capabilities that we don't want the model to have (deception, powerseeking). Are there any insights we could hope to get into the nature of planning/deception/etc that would enable us to structure learning the good capabilities in a way that avoids bad capabilities being learned? What kind of differences need to be present between the good and bad tasks for it to be possible to learn just the good, and what tools does the developer have in her arsenal to leverage these differences?
- In reality, it seems unlikely to me that we will be able to identify a difference between the cognition required for planning in general and for powerseeking in particular, because such a difference probably doesn't exist. However, it is probably very useful to have a deeper understanding of what conditions must be satisfied by the good and bad capabilities for it to be possible - even in principle - to structure training to learn only the good.
- I don't expect to find the optimal tools for influencing training to control generalisation. However, I think that there is value in introducing this framing of a problem in alignment to a wider audience, and I hope to introduce toy examples which people can work with and benchmarks to improve on.
- More broadly, I'm optimistic that working on this problem can lead to productive insights into a few Science of Deep Learning questions that seem important for alignment like: What determines when a generalisation forms in a model, and which one?

## Some reasons why it might not be worthwhile
- Results may be very contingent on choices of X and Y. Just like mech interp, there is a question of universality: how much do results generalise to new tasks? I'd be much more excited about tools which limit generalisation to Y if they work across a range of tasks

## Some toy examples I would like to study
The key property a toy example should have is: X and Y should have the property that by default (or at least some of the time) they are learned together, but it's also possible to learn X without learning Y.
- X is MNIST classification where the images can be rotated up to D° clockwise, and Y is a MNIST classification were the images are rotated between D° and 360° clockwise.
  - I expect there to be some (smaller) values of D where learning X doesn't naturally generalise to Y, but once a large enough fraction of the circle is learned, the rest probably comes with it (for a suitable model architecture).
- X is distinguishing between a red square and a blue triangle, and Y is distinguishing between a red square and a red triangle, or a red square and blue square (example lifted from [Concept Extrapolation](https://www.alignmentforum.org/s/u9uawicHx7Ng7vwxA))
- X is group composition on elements of the alternating group A5, and Y is group composition on elements of the permutation group S5 of which A5 is a normal subgroup.
  - Group composition has been chosen because we know from [A Toy Model of Universality](https://arxiv.org/abs/2302.03025) that group operations are often learned by the model learning an embedding which converts group elements into a faithful representation of the group. Then the matrices are multiplied together and the unembedding is taken.
  - This group has been chosen because A5's smallest faithful irrep is 3D, and S5's is 4D. Therefore, if the model learns the 4D irrep of A5, it would quickly learn to generalise to S5, simply having to learn the embeddings of the new group elements. On the other hand, if the model learns the 3D irrep of A5, this cannot generalise to S5, meaning that the model would have to learn S5 from scratch.
- X is modular addition modulo $p$ for $p \in \{p_i\}$, and Y is modular arithmetic modulo some other value of p.
  - For a few values of $p$ in the training data, the model is likely to learn [a few separate circuits for modular arithmetic via Fourier Transforms](https://arxiv.org/abs/2301.05217). For enough values of $p$ I expect that eventually the model will learn to generalise to any $p$. I'd be interested to see if this memorisation/generalisation behaviour mirrors the behaviour one level down, when learning modular addition modulo a single $p$. 
  - One upside of this scenario is that I already have some idea of how different ways of learning X influence the ability to learn Y.
      - If X is learned by first doing several epochs with $p_1$ only, then several with $p_2$ only (and a few cases of $p_1$ to prevent catastrophic forgetting) and so on, it seems more likely that the model would learn the $p$-specific circuits only than if the training has a lot of different values of $p$ from the beginning.
      - If each new $p_i$ is a number it hasn't seen before, this makes it harder to do something other than classification based on this weird token. ie: if
  - One downside is that it may be harder to learn general modular addition than any other the task on this list, which may slow down experiments a lot.
- X is group operation on 2 different groups, and Y is group operation on a single larger group which has both the X groups as subgroups. 

## Some techniques for influencing model generalisation
- There are various approaches which involve having access to instances of task Y to train against:
  1. After training on X, train on a mixture of X and Y where Y has random labels
     - In this case, the model can't do better than random on Y unless it memorises the labels. Therefore the model is not actually incentivised to forget Y, and it would be able to perform task Y in the test set even after memorising false labels for Y in the train set.
  2. After training on X, train on a mixture of X and Y where Y is labelled with a uniform distribution.
     - In this case, the model is incentivised to work out which tasks are X and which are Y, so that it can predict a uniform distribution when the task is Y. However, this may not affect the accuracy much (in the sense of selecting the highest probability guess) because a mixture of guessing correctly and guessing uniformly leads to less confidence but a still correct guess.
  3. After each step in training, give an adversary access to the internal activations and have it try to work out what the correct answer was. Punish the model if the correct answer (in task Y) can be gleaned from the model's activations.
     - In this case unlike previous ones, the model cannot even secretly think about the task. However, it is still possible for the model to implement a circuit that could work on Y, and a classifier early in the model which determines whether to use this circuit.
  4. Train a model specifically to discriminate between X and Y, then later finetune it to have good performance at X and bad at Y. 
- Use knowledge of how networks implement X and Y together (perhaps by reverse engineering a model that does both) to build an interpretability tool to train against.
  - Of course, the dangers of training against interpretability tools are well known. In this case, this isn't too much of a concern because I hope to reverse engineer the circuit after training, but in general this is dangerous. It's possible that if the pressure to learn the fully general circuit isn't large enough, then the pressure towards the more narrow circuitry introduced by training against the interpretability tool may be enough to reliably produce the desired outcome
- Use an understanding of the training setup to influence the outcome. For example:
  - Use early stopping, if the model learns just X initially, but would eventually learn Y
  - Find (preferably in a principled way) a set of hyperparameters, loss function, optimiser or data ordering that influences how the model generalises.

## Related Work
The big one here is Adversarial Robustness: those guys try to make the model generalise to as much as possible, and I'm kinda trying to do the opposite.

# Combining/Generalising Generalisations

I'm interested in the general question of what I call 'generalising generalisations': the phenomenon of existing generalisations in a model being used to develop other generalisations. Examples of this include:
- Combining existing generalising circuits to implement a new generalising capability. For example, suppose I have a model that is good at approximating the functions f(A,B) = F and g(C,D) = G, where A,B,C,D are features of the input. I'd like to study how the model learns to approximate the function h(A,B,C,D) = h(F,G). I expect that this may vary in different architectures eg. if I add extra layers on the end of the original model, then the function will be learned likely by first calculating F and G using existing circuits and combining these. Toy scenarios include:
  - Training a model to detect curves and edges, and then later training it to detect more complicated objects. During training, have some ability to monitor how the curve and edge detectors are used or modified. Alternatively, monitoring how curve and edge detectors arise in training on a more complicated model detector. 
  - To think about: train autoencoder B -> A -> B where dim(A) < dim(B). Then C -> A -> C or C -> B -> A -> B -> C where dim(C) > dim(B). Is there anything in this?
  - Modular addition on primes $p$ $q$, then on $pq$. 
- Modifying an existing circuit to apply it to a new situation.
  - Addition mod $p$ using one set of tokens, then addition mod $p$ using a second set of tokens (thanks Kaarel Hanni for this idea). How does the model learn to use the circuit it already used? ((I seem a bit stuck on modular addition as an idea generator - what other toy tasks are commonly used?))
  - MNIST, then rotated MNIST (not sure if this is actually going to use the original circuitry)

What I'd really like to do eventually is connect something like this to a theory of why models generalise, like Singular Learning Theory or some picture about the number of parameters involved in specifying the circuits, or something about wide basins. It's possible that these questions are not the right ones to ask to do that. But something related I'm thinking of is like: 
- Consider two circuits both implementing modular addition mod $p$, but on different token sets, vs a single circuit and something which converts between tokens from the different sets. Can we link any results about the way models learn these options/convert from one to the other to the number of parameters involved in each setup?

# Feature Identification

## A bunch of questions

- What is the most sensible definition of a feature?
  - Human interpretable directions (I don't like this one)
     - A more causal test than usual for this: can we partition the model into datapoints with destroyed performance when the direction is ablated, and datapoints with normal performance? This would be powerful, because we would have found a direction that is essential to model performance only on a restricted subset of the data.  
  - Allow for maximal information compression
     - Something about sparsity? In the feature basis, the weight matrix is sparse: each feature is only relevant for the activations of a few others.
     - How do we measure information left over? I think Anthropic has something to say about this in a recent paper/ (TO LOOK UP, maybe notes may 23?)
  - Base units of circuits, in the sense that circuits can be more easily identified when features are decomposed
     - Can we actually run an experiment and verify this? Best to start with a model we know the circuits for
     - Can we look at the features in the subspace an attention head is reading from and work out what the attention head is doing?
  - A feature is something we would want to use as a variable when hard coding the model in understandable python code
- How do we know how many features there are in an activation space and how are they arranged? How well do our toy models reflect reality?
  - Superposition/SVD
  - If superposition, a random/homogeneous polytope (low maximum interference) or a tegum product of low dimensional polytopes (low average interference)?
  - Could it be both, eg. superposition in a subspace and the remaining subspace left empty for computation later in the network?
  - How does the structure vary with location in the model? Are Residual streams harder to identify features in than MLPs?
- How do we identify ground truth features?
  - If we use dictionary learning, what is the best autoencoder/other setup?
  - If we use SVD or similar, how do we choose directions in the remaining subspace?
     - PCA produces orthogonal features unlike SVD
     - SVDCCA produces orthogonal features which are similar to feature directions in another layer
  - In synthetic (toy) datasets, are feature recovery techniques robust or do they depend on correctly guessing number of features/ feature orientation/ other?
- Which ideas (if any) should we be carrying over/translating from the field of signal processing and compression?
- How do we verify the features we have found?
  - Related to feature definition: do we check if they are human interpretable, or if they compress model information/make the weight matrix sparse, or if they allow us to read off circuits?
  - Do our methods recover the same features when used multiple times/ on different sets of activations?
  - Do our methods recover the same features when we retrain a layer of the model?
  - Using a sparse autoencoder:
     - do we find the same features when we train the autoencoder to completion twice?
     - how robust are findings to hyperparameters, choice of number of features in total and used per activation
     - if we suggest features by fixing some weights, does the model reject the features?

## Some experiments


- Train a model with $n$ neurons at layer $l$. Then retrain the network, holding all weights fixed except $W_{l-1,l}$ and $W_{l,l+1}$ which are re-initialised. Do small $k$ techniques like SVD find the same features both times (as measured by cosine similarity of features sorted by the canonical ordering of each singular vector set)? What about large $k$ techniques like sparse autoencoding?

- Repeat the first experiment, but use $D>d$ neurons at layer $l$ when retraining.
Now we want the decomposition in the narrower layer to be the same as the decomposition into the top $k$ features in the wide layer (in generalm we expect that the number of features learned in the wide model is likely to be larger than in the narrow model). Does this work better when our decomposition is with a small $k$ technique or a large $k$ technique?

- Repeat the first experiment, but in retraining, reinitialise the weights between more layers. Now we don't expect features to be represented in the same directions as before, but insofar as the \textit{Universality Hypothesis} (\cite{li2016convergent, olah2020zoom}) is true, the same features are likely to be represented in some way or other. Using the technique that worked best in the first experiment, can we identify features we find in the two models with each other?

- Train pairs of (model with ReLU at layer $l$, model with ReLU and a filter which sets to zero all but the top $m$ activations at layer $l$) for a range of values of $d$. If the top-$m$ filter forces features to be aligned to the neuron basis, then the models should only have similar performance when the number of features $k \leq d$. 

- We'd like to improve our confidence that sparse autoencoders have found the real features. If we train the sparse autoencoder twice, do we get the same decomposition of activation vectors? If we fix some weights on the sparse autencoder to hardcode some random features into the model, but we learn their biases, does the model choose to ignore them from its feature set by setting their bias really low so the neurons never activate? 

- Is there something correct in both the large-$k$ and small-$k$ pictures? For instance, instead of doing SVD and also instead of using a sparse autoencoder for feature decomposition, train an autoencoder in which some number $\delta<d$ of neurons have no $L^1$ penalty, and the rest do. The thought is that these $\delta$ features might pick out the top $\delta$ compositional directions, and the rest will be used for capturing the features in superposition. Will we find that the features are more interpretable this way than in either the dense or sparse extreme?



## Why is this important:

Feedback loop of: better understanding of feature encoding -> better toy models -> more refined feature discovery -> repeat. Right now, we have techniques that work somewhat on our toy models, but we need to start using them to identify features in language models, to verify if we have found anything of significance, and to understand the ways in which our toy models and feature identification techniques are limited.

## Related message exchange
It seems possible to me that at least for some tasks, one can find a small number of directions to ablate (maybe even <= 1 per residual stream position for a number of positions) which hurt performance significantly and such that remaining directions do not hurt performance significantly. For instance, maybe there is an "tree" feature whose ablation significantly reduces performance on the task of completing sentences like "The oak had green _"

a related idea: Take a particular task (like indirect object identification), and track some coarse-grained data about non-ablatable directions at various points in the residual stream (and maybe in other model components). See if the places where this data suddenly changes can be used to locate model components which are important for doing the task

Our intuitions notwithstanding, there is a (somewhat involved) proof that linear combinations of ground truth features can be recovered by L1 minimization in Chapter 3 of High-dimensional data analysis with Low Dimensional Models by Wright & Ma.

"how to make these synthetic activation data sets more similar to real model activations" is roughly equivalent to "in which way are real model activations made out of features"

PCA features are uncorrelated — in fact, this seems like something of an argument for using PCA over SVD

this orthogonal basis over other orthogonal ones? one thing that distinguishes is that this is the only one for which the first k\< k directions are also an optimum for feature decomposition with k’ things, but i guess this is not really a strong reason if we assume we know how many features there are, so the above can be ignored, i think a better justification for the svd is sth like if you generated the data by having independent (or maybe uncorrelated is enough, i need to think more) random variables with different variances giving components in particular orthogonal directions, then svd would recover those directions, i think