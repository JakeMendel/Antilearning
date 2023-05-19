
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
  - One upside of this scenario is that I already have some idea of how different ways of learning X influence the ability to learn Y. For example, if X is learned by first doing several epochs with $p_1$ only, then several with $p_2$ only (and a few cases of $p_1$ to prevent catastrophic forgetting) and so on, it seems more likely that the model would learn the $p$-specific circuits only than if the training has a lot of different values of $p$ from the beginning.
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