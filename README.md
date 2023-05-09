
# Antilearning
Consider some task X, and a related task Y, with the property that training a model generalise well on task X naturally leads to the model being able to do Y as well. I am interested in training and model on task X in a different way so that task Y is never learned.

Some toy examples which I would like to study:
- X is MNIST classification where the images can be rotated up to D° clockwise, and Y is a MNIST classification were the images are rotated between D° and 360° clockwise.
  - I expect there to be some (smaller) values of D where learning X doesn't naturally generalise to Y, but once a large enough fraction of the circle is learned, the rest probably comes with it.
- X is distinguishing between a red square and a blue triangle, and Y is distinguishing between a red square and a red triangle, or a red square and blue square.
- X is group composition on elements of the alternating group A5, and Y is group composition on elements of the permutation group S5 of which A5 is a normal subgroup.
  - This example has been chosen because we know from [A Toy Model of Universality](https://arxiv.org/abs/2302.03025) that group operations are often learned by the model learning an embedding which converts group elements into a faithful representation of the group. Then the matrices are multiplied together and the unembedding is taken.
  - This group has been chosen because A5's smallest faithful irrep is 3D, and S5's is 4D. Therefore, if the model learns the 4D irrep of A5, it would quickly learn to generalise to S5, simply having to learn the embeddings of the new group elements. On the other hand, if the model learns the 3D irrep of A5, this cannot generalise to S5, meaning that the model would have to learn S5 from scratch.
- X is modular addition modulo $p$ for $p \in \{p_i\}$, and Y is modular arithmetic modulo some other value of p.
  - For a few values of $p$ in the training data, the model is likely to learn [a few separate circuits for modular arithmetic via Fourier Transforms](https://arxiv.org/abs/2301.05217). For enough values of $p$ I expect that eventually the model will learn to generalise to any $p$. I'd be interested to see if this memorisation/generalisation behaviour mirrors the behaviour one level down, when learning modular addition modulo a single $p$. 
  - One upside of this scenario is that I already have some idea of how different ways of learning X influence the ability to learn Y. For example, if X is learned by first doing several epochs with $p_1$ only, then several with $p_2$ only (and a few cases of $p_1$ to prevent catastrophic forgetting) and so on, it seems more likely that the model would learn the $p$-specific circuits only than if the training has a lot of different values of $p$ from the beginning.
  - One downside is that it may be harder to learn general modular addition than any other the task on this list, which may slow down experiments a lot.

Some techniques for influencing model generalisation:
- There are various approaches which involve having access to instances of task Y to train against:
  - eg. 

