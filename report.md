### Technical Report

#### **Finite Element Solution for Cardiac Wavefront Diffusion**

### Model

A mathematical model that studies wavefront propagation in the myocardium (cardiac muscle) tissue can be represented by the following eikonal problem:

$$\begin{cases} \nabla \cdot M(x, \nabla u) \nabla u = \frac{\partial u}{\partial t} & \text{in } H \\ - \nabla \cdot M(x, \nabla u) \nabla u + c_0 \nabla u \cdot M(x, \nabla u) \nabla u = m_t & \text{on } \partial H \\ u = 0 & \text{for } t = 0 \end{cases}$$

where \(c_0\) and \(m_t\) are speed and time constants.

\(H\) is the problem domain, in this case, the myocardium of a single ventricle. The system has a unique solution only if the signal's sources are defined by essential conditions, like:

$$u = g \text{ on a region of } H$$

The matrix \(M(x, \nabla u)\) abstracts the bioelectrical features of the myocardium and is a non-linear function of \(\nabla u(x)\) and position \(x\).

These features are essentially two:

1.  **Bidomain:** Two conducting media coexist in the space \(H\): the intracellular medium (myocardial cells and their junctions) and the extracellular medium (interstitial space). These are defined by conductivity tensors \(M_i\) and \(M_e\).
2.  **Fiber Architecture:** To simulate the fiber structure, each point \(x\) is associated with a unit vector \(l_a\) defining the local fiber direction. This allows for defining a local orthogonal system and a matrix \(A\).

Since \(M_i^T = A \cdot M_i \cdot A^*\) and \(M_e^T = A \cdot M_e \cdot A^*\), then \(M(x, \nabla u)\) is a non-linear function of \(M_i\) and \(M_e\). This matrix abstracts the bioelectrical features of the cardiac tissue.

The function \(u(x)\) is the activation time, the time at which the myocardial electrical potential changes from resting to activated at each position. The isosurfaces of \(u(x)\) identify the traveling activation wavefronts.

### Mesh and Fiber Architecture

### Numerical Approximation

This problem is solved using the finite element method with Galerkin's method and linear Lagrangian hexahedral elements. A temporal derivative term is added to the eikonal equation, making it hyperbolic and easier to find an asymptotic solution.

The approximate solution is:

$$\sum_{j=1}^{N_r} w_j(t) f_j + \sum_{s=1}^{N_a} g_s f_s \xrightarrow[t \to \infty]{} u(x) = \lim_{t \to \infty} w(t) \cdot f$$

where \(\{f_j\}\) are shape functions, and coefficients in the second sum are nodal values of Dirichlet conditions.

This leads to a system of non-linear ordinary differential equations:

$$\begin{cases} C \frac{dw}{dt} + A(w)w + m(w) = f \\ w(0) = w_0 \end{cases}$$

where:

*   \(C_{i,j} = \int_{H_r} f_i \cdot f_j dx\); \(i, j = 1, ..., N_r\)
*   \(f_i = \int_{H_r} h_i m(x, \nabla w) dx + \sum_{s=1}^{N_a} \int_{S_n} h_i c_0 g_s f_s d\gamma\); \(i = 1, ..., N_r\)
*   \(A(w)_{i,j} = \int_{H_r} (\nabla f_j)^T M(x, \nabla w) \nabla f_i dx\); \(i, j = 1, ..., N_r\)
*   \(m(w)_i = c_0 \int_{H_r} (\nabla w h_i)^T M(x, \nabla w) \nabla w \cdot \nabla f_i dx\); \(i = 1, ..., N_r\)

A fully explicit discretization is used in this implementation:

$$C \cdot w^{k+1} = C \cdot w^k + \Delta t \cdot f - A(w^k) \cdot w^k - m(w^k)$$

### Algorithm

1.  Build and number the mesh according to the hexahedral reference element.
2.  Define essential conditions based on signal sources.
3.  Start the iterative process with an initial solution hypothesis.
4.  Estimate the gradient of the temporary solution using finite element interpolation.
5.  Modify the gradient at boundaries to ensure boundary conditions.
6.  Calculate upwind gradients and apply them to the non-linear transport part \(m(w)\).
7.  Construct integrals using the finite element method to define coefficients of the linear system.
8.  Solve the linear system.
9.  Compare the temporary solution with the previous one and check the relative residual error. If the approximation is satisfactory, continue; otherwise, iterate back to step 4.
10. Write output files for visualization.

### Upwind

An upwind technique is applied to \(m(w)\) for stabilization:

$$\Gamma_x \nabla u = c_0 \cdot \nabla u \cdot M \cdot \nabla u \geq 0$$

$$\Gamma(x, \nabla u) = \nabla u \cdot \Gamma(x, n) \text{ where } n = \frac{\nabla u}{|\nabla u|}$$

$$\nabla u_{up} \approx \nabla u_{xi} = \sum_{i=1}^{3} [max(D_{ui}^+, 0) + min(D_{ui}^-, 0)]$$

where \(D_{ui}^+\) and \(D_{ui}^-\) are forward and backward finite differences approximating derivatives in Cartesian directions, determined using \(w\) values at each iteration.

### Gradients' Interpolation

Gradient interpolation is implemented using shape function gradients:

$$\nabla w = \sum_{j \in N} w_j \cdot \nabla f_j$$

This causes discontinuities at element boundaries, requiring natural boundary conditions to be enforced. These conditions are estimated at each iteration due to the non-linearity of \(M(x, \nabla u)\).

### Essential References

1.  P. Colli Franzone, L. Guerri, M. Pennacchio & B. Traccardi, ‘Spreading of Excitation in 3-D models of the anisotropic cardiac tissue. II. Effect of fiber architecture and ventricular geometry.’, Math. Biosci. 147: 131-171 (1998).
2.  P. Colli Franzone & L. Guerri, ‘Spreading of excitation in 3-D models of the anisotropic cardiac tissue. I. Validation of the eikonal model.’, Math. Biosci., 113: 145-209 (1993).
3.  S. Osher & J.A. Sethian, ‘Front propagating with curvature-dependent speed: algorithms based on Hamilton-Jacobi formulations.’ J. Comp. Phys. 79: 12-49 (1988).

### Pictures

**Examples of Wavefronts**
