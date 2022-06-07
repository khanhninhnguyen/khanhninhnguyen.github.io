---
layout: post
title: "[Reading Group] A Learnable Radial Basis Positional Embedding for Coordinate-MLPs"
date: 2022-06-07

categories: paper-reading reading-group



---
**Authors**: Sameera Ramasinghe and Simon Lucey. University of Adelaide
# Positional embedding (PE)

Conventional MLPs trained with low-dimensional coordinate inputs is shown both experimentally [2, 3] and theoretically [1] to not able to learn **signal with high frequency**.

To learn function with high frequency, **positional embedding** (PE) is often used to project the coordinates to a higher dimensional space. 
{% include figure.html path="assets/img/blog/learn-basis/Screenshot from 2022-06-06 23-42-11.png" class="img-fluid rounded z-depth-1" %}

Image from [1]

**PE in NeRF**:

$$
\gamma(p) = (sin(2^0\pi p), cos(2^0\pi p), ..., sin(2^L\pi p), cos(2^L\pi p))
$$

Where $$\gamma$$ is the mapping from $$\mathbb{R}$$ to higher dimensional space $$\mathbb{R}^{2L}$$. $$p$$ is nomarlized between $$[-1, 1]$$

Example of 1D positional encoding with $$L=10$$ and 100 values
{% include figure.html path="assets/img/blog/learn-basis/pe_example.png" class="img-fluid rounded z-depth-1" %}


**Note: PE in attention mechanism**:
- Positional encoding is used to encode the sequence spatial position. The formula is a bit different but the idea is the same. 
- There are works proposed learned PE for sequential spatial position.

# Contributions
- This paper only deals with PE for learning high frequency in coordinate-MLP
- PE is often needed to be pre-defined and tuned for a given task, which could be sub-optimal.
+ Learned PE with coordinate-MLPs **does not generalize**: They trained on 25% of the pixels/points and predict the rest.
{% include figure.html path="assets/img/blog/learn-basis/not_generalize.png" class="img-fluid rounded z-depth-1" %}
-> This paper propose a method to **learn PE for coordinate-MLPs** that can **better generalize.**

# Idea

MLPs preserve **smoothness** [1, 4]-> **Generalization**.

PE helps the network learn **high frequency details** -> **Overfitting** 


**Smoothness** is crucial to make PE generalizes. 

Given a signal $$f(x): \mathbb{R} \rightarrow \mathbb{R}$$  and the positional embedding $$\Phi(\cdot): \mathbb{R} \rightarrow \mathbb{R}^D$$, the smoothness can be maintained by preserving

$$
\frac{||d\Phi(x)||_2}{|df(x)|} = K, 
$$

Where $$K$$ is constant, thus

$$
\frac{||d\Phi(x)||_2}{|dx|} \propto \frac{||df(x)||_2}{|dx|}, 
$$


$$\|d\Phi(x)\|_2$$ can be written in term of Riemannian metric tensor $$M$$ :

$$
\frac{\sqrt{\text{det}(M(x)}dx)}{dx} \propto \frac{||df(x)||_2}{|dx|}
$$

Where $$\sqrt{\text{det}(M(x))}$$  is the volume element of $$M$$ at $$x$$. Thus:

\begin{equation}
\label{eq:constraint}
\sqrt{\text{det}(M(x)}dx \propto \frac{||df(x)||_2}{|dx|}
\end{equation}

Intuitively, it means that the **volume of the manifold induced by the PE** should be **proportional** to the norm of the **first-order gradients** (Jacobian) of the output signal.

# Discrete approximation of the manifold
The idea is to build **a graph** $$\mathcal{G}$$ with edges $$\mathcal{E}$$ vertices $$\mathcal{V}$$ to approximate the manifold.
Given an embedding function $$\Phi: \mathbb{R}^N \rightarrow \mathbb{R}^M$$ and a set of coordinates $$\{x\}^L_{i=1}$$. 

- Vertices $$\mathcal{V}$$:  $$\{\Phi(x)\}_{i=1}^L$$
- Edges $$\mathcal{E}$$: between two vertices $$\Phi(x_i)$$ , $$\Phi(x_j)$$ 

$$
w_{ij} = (\rho_i \rho_j)^{-\lambda}\theta(d_{i,j}),
$$

$$
\theta(d_{i,j})= exp(-\frac{d^{2}_{i, j}}{2\epsilon^2}),
$$

where $$d_{i,j} = ||\Phi(x_i) - \Phi(x_j)||_2$$ The larger $$d_{i, j}$$, the smaller $$\theta(d_{i,j})$$. 
$$\sigma, \lambda$$ are hyper-parameters.



The **regularization objective** that enforces the Eq.\ref{eq:constraint} is

$$
\tau(u) = u^T L u
$$

where $$u=[\varphi(x_1), ...,\varphi(x_L)]^T$$, $$\varphi: \mathbb{R}^N \rightarrow \mathbb{R}$$ is an arbitrary smooth function and
$$L$$ is the unnormalized **graph Laplacian** 

$$L = A - D$$

where $$A$$ and $$D$$ are the **adjacency** matrix and **degree** matrix.

# Radial basis embedders

The learned PE is defined as:

$$\Phi(x)=[\phi_1(x), ..., \phi_D(x)]$$

where $$\phi_i(x)=\exp[(-\frac{(x\alpha - t_i)^2}{2\sigma_x^2})$$ . $$t_i$$ and  $$b$$ are constants $$\alpha \in \mathbb{R}^N$$  is constant vector, and the standard deviation $$\sigma_x$$ is coordinate-dependant parameter which is **optimized/learned**.

The final learning objective is 

$$
u^TLu - \lambda ||A||
$$

$$- \lambda \|A\|$$ is added to maintain graph structure and avoid trivial solution. $$A$$ is adjacency matrix.

# Scalability issue 
It requires the computation of $$A \in \mathbb{R}^{N \times N}$$  which is huge with large $$N$$.

By **observating** the following figure, $$\sigma_x$$ approximately depends on $$||\frac{\delta \bar{f}(x)}{\delta x}||_2$$ where $$\bar{f}(x)$$ is the mean value of $$f(x)$$
{% include figure.html path="assets/img/blog/learn-basis/approx.png" class="img-fluid rounded z-depth-1" %}
So they approximate $$\sigma_x$$ using the polynomial:

$$
\sigma_x = \beta_1 + \beta_2 ||\frac{\delta \bar{f}(x_i)}{\delta x_i}||_2+\beta_3 ||\frac{\delta \bar{f}(x_i)}{\delta x_i}||^2_2 + ....
$$

$$\beta_{i=1}^L$$ can be found analytically or using small training set of $$\|\frac{\delta \bar{f}(x_i)}{\delta x_i}\|_2$$, $$\sigma_i$$ pairs.  They compute $$\sigma$$ using small $$N$$ (i.e. 20 in their paper) and use them to find $$\beta$$ 

# Experiments
**Trained vs untrained** $$\sigma$$
{% include figure.html path="assets/img/blog/learn-basis/exp1.png" class="img-fluid rounded z-depth-1" %}
**Compare with RFF used in [17]**
{% include figure.html path="assets/img/blog/learn-basis/exp_rff.png" class="img-fluid rounded z-depth-1" %}

# References
[1] Tancik et al. Fourier Features Let Networks Learn High Frequency Functions in Low Dimensional Domains. NeuRIPS 2020

[2] Mildenhall et al. Nerf: Representing scenes as neural radiance fields for view synthesis. ECCV 2020

[3] Niemeyer et al. Differentiable volumetric rendering: Learning implicit 3d representations without 3d supervision. CVPR 2020

[4] Rahaman et al. On the spectral bias of neural networks. ICML 2019