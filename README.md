# Can We Build Navier–Stokes From Scratch?

## A Computational Investigation of Fluid Motion, Vorticity, Viscosity and Concentration

> Can we start from Newton's laws and the incompressibility constraint, construct a numerical Navier–Stokes solver ourselves, validate it against a known solution, and then use that solver as an experimental laboratory to investigate mechanisms related to finite-time blow-up?

This repository is my attempt to answer that question computationally.

This is not a claim to solve the Navier–Stokes Millennium Prize Problem.

Instead, the goal is to build the equation from first principles, turn it into an actual computational system, test whether that system behaves as expected, deliberately investigate its failure modes, and use the resulting laboratory to understand the ideas behind the recent OpenAI construction.

The important part of this project is not just the final simulation.

It is the investigation process.

---

## The Core Idea

The project follows one loop throughout:

**Observation → Question → Hypothesis → Experiment → Evidence → Update**

Rather than beginning with a black-box fluid simulator, I started with the minimum mathematical objects required to describe fluid motion:

```text
Scalar field
     ↓
Vector field
     ↓
Velocity field
     ↓
Spatial derivatives
     ↓
Material acceleration
     ↓
Navier–Stokes equation
     ↓
Discretization
     ↓
Numerical operators
     ↓
Pressure projection
     ↓
Navier–Stokes solver
     ↓
Validation
     ↓
Controlled experiments
     ↓
Failure / numerical limits
     ↓
Connection to finite-time concentration
     ↓
OpenAI construction
```

The objective is to understand every important transition rather than simply calling an existing CFD package.

---

# What I Built

The main artifact is a single computational notebook:

`navier_stokes_first_principles_lab.ipynb`

The notebook progresses from mathematical intuition to an operational 2D incompressible Navier–Stokes laboratory.

It contains:

1. Baseline velocity-field experiments
2. Spatial derivative experiments
3. Material acceleration
4. Vorticity
5. Finite-difference discretization
6. Numerical derivative operators
7. A 2D incompressible Navier–Stokes solver
8. Pressure projection
9. Taylor–Green vortex validation
10. Energy diagnostics
11. Reynolds-number experiments
12. Concentration experiments
13. Numerical-resolution / stability experiments
14. Vortex-stretching intuition
15. Connection to the OpenAI Navier–Stokes construction
16. A final evidence and limitation analysis

The notebook is intended to be read from top to bottom.

It is both the implementation and the research log.

---

# 1. Starting From First Principles

## Observation

A fluid is not represented by tracking one object.

Instead, we describe the state of the fluid at every location.

For velocity:

$$
\mathbf{u}(x,y,t)
=
\begin{bmatrix}
u(x,y,t)\\
v(x,y,t)
\end{bmatrix}
$$

Every point in the domain has a velocity vector.

This gives us a velocity field.

---

## Question

If velocity changes from one location to another, how do we mathematically describe those changes?

---

## Hypothesis

Spatial derivatives should allow us to measure how velocity changes across the domain.

This leads to:

* partial derivatives
* gradients
* divergence
* Laplacian
* vorticity

These quantities eventually become the numerical building blocks of Navier–Stokes.

---

# 2. From Newton's Law to Navier–Stokes

The starting physical idea is Newton's second law:

$$
F = ma
$$

For a fluid, acceleration is more subtle because a fluid particle can accelerate for two reasons:

1. The velocity at its location changes with time.
2. The particle moves into a region where the velocity is different.

The resulting material acceleration is:

$$
\frac{D\mathbf{u}}{Dt}
=
\frac{\partial\mathbf{u}}{\partial t}
+
(\mathbf{u}\cdot\nabla)\mathbf{u}
$$

This produces the incompressible Navier–Stokes equation:

$$
\boxed{
\frac{\partial\mathbf{u}}{\partial t}
+
(\mathbf{u}\cdot\nabla)\mathbf{u}
=
-\nabla p
+
\nu\nabla^2\mathbf{u}
+
\mathbf{f}
}
$$

subject to the incompressibility constraint:

$$
\boxed{
\nabla\cdot\mathbf{u}=0
}
$$

where:

* \(\mathbf{u}\) = velocity field
* \(p\) = pressure
* \(\nu\) = kinematic viscosity
* \(\mathbf{f}\) = external forcing
* \(\nabla p\) = pressure gradient
* \(\nabla^2\mathbf{u}\) = viscous diffusion

The computational interpretation used throughout the project is:

```text
TIME CHANGE + ADVECTION
        =
PRESSURE + VISCOSITY + FORCING
```

Each term eventually becomes an actual computation.

---

# 3. Turning the Equation Into an Algorithm

A differential equation describes continuous behaviour.

A computer works with finite arrays and finite time steps.

So the next question is:

> How do we turn the continuous equation into something a computer can actually execute?

The domain is discretized into a grid.

Spatial derivatives are approximated using finite differences.

For example:

$$
\frac{\partial u}{\partial x}
\approx
\frac{u_{i+1,j}-u_{i-1,j}}{2\Delta x}
$$

and the Laplacian is approximated from neighbouring grid points.

The continuous equation therefore becomes a sequence of numerical operations:

```text
Current velocity
      ↓
Compute derivatives
      ↓
Compute advection
      ↓
Compute viscosity
      ↓
Predict velocity
      ↓
Solve pressure correction
      ↓
Project velocity
      ↓
New divergence-free velocity
      ↓
Repeat
```

This is the central engineering transition of the project.

---

# 4. Pressure Is Not Just Another Term

One of the most important computational ideas in the project is the role of pressure.

For incompressible flow:

$$
\nabla\cdot\mathbf{u}=0
$$

must remain satisfied.

However, the intermediate velocity obtained after advection and viscosity does not necessarily satisfy this condition.

The solver therefore performs a pressure projection.

Conceptually:

```text
Velocity prediction
       ↓
May contain divergence
       ↓
Pressure Poisson equation
       ↓
Pressure field
       ↓
Subtract pressure gradient
       ↓
Divergence-free velocity
```

This turns pressure into a mechanism that enforces the incompressibility constraint.

The project uses a spectral/FFT-based pressure solve on the periodic domain.

---

# 5. Validation: Taylor–Green Vortex

A numerical solver should not be trusted simply because it produces a visually convincing animation.

It needs to be tested against something known.

For this reason, the project uses the Taylor–Green vortex as a validation case.

The analytical velocity field is:

$$
u(x,y,t)
=
U_0\cos(x)\sin(y)e^{-2\nu t}
$$

$$
v(x,y,t)
=
-U_0\sin(x)\cos(y)e^{-2\nu t}
$$

This gives a known solution against which the numerical solution can be compared.

The notebook measures quantities such as:

* relative \(L_2\) error
* maximum velocity error
* divergence
* energy behaviour
* temporal evolution

The purpose is simple:

> Before using the solver as an experimental laboratory, test whether it can reproduce a situation whose behaviour we already understand.

---

# 6. Experiments

The project is structured around questions rather than demonstrations.

## Experiment 0 — Baseline Velocity Fields

### Question

What do different velocity fields actually look like?

### Cases

* Uniform flow
* Radial flow
* Rotational flow
* Shear flow

### Purpose

Build visual intuition for velocity as a field rather than treating velocity as a single number.

---

## Experiment 1 — Spatial Derivatives

### Question

How does local velocity structure appear through spatial derivatives?

### Measurements

* First derivatives
* Gradient behaviour
* Divergence
* Vorticity

### Purpose

Connect the mathematical operators directly to observable structure in the field.

---

## Experiment 2 — Material Acceleration

### Question

Can a fluid particle accelerate even when the velocity field itself is steady?

### Key idea

Yes.

For a steady field:

$$
\frac{\partial\mathbf{u}}{\partial t}=0
$$

but:

$$
(\mathbf{u}\cdot\nabla)\mathbf{u}
\neq 0
$$

can still produce acceleration.

This experiment connects the chain rule and spatial velocity gradients to the nonlinear term in Navier–Stokes.

---

# 7. Experiment 3 — Vorticity

Vorticity is defined as:

$$
\boldsymbol{\omega}
=
\nabla\times\mathbf{u}
$$

In 2D, the relevant component becomes:

$$
\omega_z
=
\frac{\partial v}{\partial x}
-
\frac{\partial u}{\partial y}
$$

It provides a way of measuring local rotational structure.

The experiment investigates how different velocity fields generate different vorticity patterns.

---

# 8. Experiment 4 — Peak Velocity vs Total Energy

One of the important ideas behind finite-time blow-up is that a solution could potentially develop arbitrarily large pointwise velocity while still having finite total kinetic energy.

The relevant distinction is between:

### Maximum velocity

$$
\|\mathbf{u}\|_\infty
=
\max_{\mathbf{x}}
|\mathbf{u}(\mathbf{x})|
$$

and:

### Total kinetic energy

$$
E
\sim
\int |\mathbf{u}|^2\,dV
$$

These are different quantities.

A field can have a very large peak concentrated into a sufficiently small region without automatically having infinite total energy.

The notebook explores this distinction computationally.

### Important limitation

A toy concentration experiment is not evidence that Navier–Stokes actually develops a singularity.

It only demonstrates the mathematical distinction between:

```text
peak magnitude
```

and

```text
integrated quantity
```

---

# 9. Experiment 5 — Viscosity and Diffusion

The viscous term is:

$$
\nu\nabla^2\mathbf{u}
$$

A simpler way to understand its role is through diffusion:

$$
\frac{\partial q}{\partial t}
=
\nu\nabla^2q
$$

where \(q\) represents a localized quantity such as vorticity.

### Question

What happens when viscosity acts on a concentrated structure?

### Hypothesis

Viscosity should spread and smooth sharp structures.

### Measurements

* Peak vorticity
* Spatial width
* Energy
* Evolution over time

The experiment is deliberately simpler than the full Navier–Stokes system.

Its purpose is to isolate the smoothing mechanism.

---

# 10. Experiment 6 — Nonlinear Transport vs Smoothing

Navier–Stokes contains competing mechanisms.

The nonlinear term:

$$
(\mathbf{u}\cdot\nabla)\mathbf{u}
$$

can transport and rearrange structure.

The viscous term:

$$
\nu\nabla^2\mathbf{u}
$$

smooths structure.

This experiment investigates that competition using a controlled computational model.

The question is:

> What happens when nonlinear transport tries to create or maintain concentrated structure while viscosity tries to smooth it?

This is a toy experiment.

It is not a numerical reproduction of 3D blow-up.

---

# 11. Experiment 7 — Vortex Stretching

Vortex stretching is particularly important because it is inherently associated with three-dimensional flow.

A simplified geometric experiment is used to visualize the idea of stretching a vortex tube.

The conceptual picture is:

```text
Initial vortex tube
       ↓
Tube becomes longer
       ↓
Cross-section changes
       ↓
Vorticity can be amplified
```

The important distinction is:

> This experiment provides geometric intuition for vortex stretching. It is not a 3D Navier–Stokes simulation and does not reproduce the mathematical mechanism in the Millennium Prize problem.

---

# 12. Experiment 8 — Connecting the Experiments to the OpenAI Construction

Only after building the computational intuition do we connect the project to the recent OpenAI Navier–Stokes work.

The conceptual chain discussed in the project is:

```text
Shrinking vortex core
        ↓
Increasingly concentrated velocity
        ↓
Elongated / slender structure
        ↓
Nonlinear momentum transfer
        ↓
Oscillatory pulses
        ↓
Stress cancellation
        ↓
Viscosity controlled
        ↓
Bounded kinetic energy
        ↓
Unbounded L∞ velocity
```

The goal is not to claim that the notebook reproduces this construction.

Instead, each part of the chain is classified as:

1. Something directly demonstrated by our experiments
2. Physical or mathematical intuition supported by the experiments
3. Something claimed or established by the OpenAI construction
4. Something our experiments cannot establish

This distinction is essential.

---

# 13. What the OpenAI Construction Adds

The computational experiments in this repository are intentionally much simpler than the construction described by OpenAI.

The reported construction concerns a 3D incompressible Navier–Stokes solution with:

* positive viscosity
* smooth compactly supported forcing
* bounded kinetic energy
* velocity becoming unbounded in the \(L^\infty\) sense as time approaches a finite time

The construction involves a highly structured mechanism involving a concentrating vortex configuration, oscillatory behaviour and cancellation of singular contributions.

The project therefore treats the OpenAI result as the advanced mathematical target that the experiments help make more understandable.

The notebook does **not** reproduce the proof.

---

# 14. What Have We Actually Established?

This is the most important section of the project.

The conclusions are deliberately separated into different levels of confidence.

## A. Directly Demonstrated

Things directly measured by the notebook's computational experiments.

Examples include:

* behaviour of specified velocity fields
* numerical spatial derivatives
* vorticity patterns
* material acceleration in controlled fields
* pressure projection behaviour
* numerical Navier–Stokes evolution
* agreement with a known validation solution within measured numerical error
* energy evolution under selected conditions
* effects of viscosity in controlled experiments
* sensitivity to numerical resolution and timestep

These are computational observations.

---

## B. Strong Physical / Mathematical Intuition

The experiments provide intuition for:

* why velocity is a field
* why particles can accelerate through a steady field
* why advection is nonlinear
* why pressure is necessary for incompressibility
* why viscosity smooths structure
* why concentrated velocity does not automatically imply infinite total energy
* why three-dimensional vortex stretching matters
* why nonlinear transport and viscosity can compete

These are useful insights, but they are not mathematical proofs of singularity behaviour.

---

## C. Claims From the OpenAI Construction

The notebook does not independently prove the advanced claims made by the OpenAI work.

Those claims belong to the mathematical construction and its formalization.

They are presented separately from the computational evidence produced here.

---

## D. What This Project Cannot Establish

This project cannot establish:

* existence of a genuine 3D Navier–Stokes singularity
* nonexistence of singularities
* the Millennium Prize solution
* correctness of a mathematical proof
* equivalence between our 2D numerical experiments and the OpenAI construction
* convergence to a singular solution simply because a simulation produces a very large velocity
* that numerical instability represents physical blow-up

These require mathematical analysis beyond the computational experiments in this repository.

---

# 15. Why Numerical Blow-Up Is Not Mathematical Blow-Up

This distinction is critical.

A numerical simulation can fail because of:

* timestep instability
* insufficient spatial resolution
* discretization error
* aliasing
* boundary-condition issues
* floating-point limitations
* an inconsistent numerical operator

Therefore:

```text
Simulation becomes unstable
```

does not imply:

```text
Navier–Stokes becomes singular
```

One of the goals of the project is actually to make this distinction visible by investigating numerical limitations directly.

---

# 16. Numerical Self-Audit

A simulation should not only produce pictures.

The project therefore treats diagnostics as part of the experiment.

Important quantities include:

### Divergence

$$
\nabla\cdot\mathbf{u}
$$

For incompressible flow this should remain close to zero.

### Relative solution error

Comparison against an analytical reference where available.

### Maximum velocity

$$
\|\mathbf{u}\|_\infty
$$

### Kinetic energy

$$
E
\sim
\int |\mathbf{u}|^2\,dV
$$

### Vorticity

$$
\omega
=
\nabla\times\mathbf{u}
$$

### Resolution dependence

Run the same experiment with different grid resolutions.

### Timestep dependence

Repeat experiments with different timesteps.

The point is to ask:

> Is the behaviour physical, mathematical, or merely numerical?

---

# 17. Research Philosophy

The project follows a simple rule:

> Do not trust the simulation because it looks right.

Instead:

```text
Observe
   ↓
Ask a precise question
   ↓
Make a falsifiable hypothesis
   ↓
Build the smallest useful experiment
   ↓
Measure
   ↓
Compare against expectation
   ↓
Find failure modes
   ↓
Update the hypothesis
```

This is more important to me than producing a visually impressive fluid animation.

---

# 18. Research Log

I did not want to approach Navier–Stokes by memorizing another equation.

I wanted to know whether I could actually build the machinery behind it.

At the beginning, even the notation was unfamiliar. A velocity field looked abstract because velocity was usually something I thought about as a number attached to an object.

The first shift was realizing that a fluid is different.

Instead of asking:

> What is the velocity of this object?

I had to ask:

> What is the velocity at every point in space?

That led naturally to vector fields, spatial derivatives and gradients.

Then came the more important surprise.

A particle does not need the velocity field to change with time in order to accelerate.

If the particle moves through a spatially varying field, it can experience acceleration through:

$$
(\mathbf{u}\cdot\nabla)\mathbf{u}
$$

That was the point where the nonlinear term stopped looking like mathematical decoration.

It represented something physical.

From there, the project became an engineering problem.

The equation had to become code.

Derivatives had to become finite differences.

Pressure had to become a computational correction.

The incompressibility constraint had to become something measurable.

And the resulting solver had to be tested rather than trusted.

The most useful mindset shift was realizing that failure is also data.

If a simulation becomes unstable, the first question should not be:

> Did Navier–Stokes blow up?

It should be:

> What exactly failed?

That distinction becomes especially important when studying a mathematical problem where the difference between extremely large velocity and genuine finite-time singularity matters.

The final goal was therefore not to produce a "fluid simulation."

It was to build a laboratory in which I could ask better questions.

---

# 19. Conceptual Map

The entire investigation can be summarized as:

```text
Newton's Law
     │
     ▼
Velocity Field
     │
     ├──────────────► Spatial Derivatives
     │                     │
     │                     ├── Gradient
     │                     ├── Divergence
     │                     └── Laplacian
     │
     ▼
Material Acceleration
     │
     ▼
Navier–Stokes Equation
     │
     ├── Advection
     ├── Pressure
     ├── Viscosity
     └── Forcing
     │
     ▼
Numerical Discretization
     │
     ▼
Computational Solver
     │
     ▼
Pressure Projection
     │
     ▼
Validation
     │
     ├── Taylor–Green vortex
     ├── Energy
     └── Divergence
     │
     ▼
Experimental Laboratory
     │
     ├── Vorticity
     ├── Viscosity
     ├── Reynolds number
     ├── Concentration
     ├── Nonlinear transport
     └── Numerical limitations
     │
     ▼
3D Vortex Stretching
     │
     ▼
Finite-Time Concentration
     │
     ▼
OpenAI Construction
     │
     ├── Shrinking vortex core
     ├── Slender structure
     ├── Nonlinear momentum transfer
     ├── Oscillatory pulses
     ├── Stress cancellation
     ├── Bounded kinetic energy
     └── Unbounded L∞ velocity
```

---

# 20. Important Questions That Remain

The experiments leave several questions open.

### 1. What exactly prevents a concentrated structure from becoming singular?

### 2. How does vortex stretching quantitatively compete with viscous dissipation in 3D?

### 3. Under what conditions can nonlinear amplification overcome viscous smoothing?

### 4. How can a velocity become arbitrarily large while kinetic energy remains bounded?

### 5. What role does the geometry of the vortex play?

### 6. How do oscillatory structures produce the cancellations required by the advanced construction?

### 7. How much of the observed behaviour is robust under changes in resolution and timestep?

### 8. What numerical diagnostics are sufficient to distinguish physical concentration from numerical instability?

### 9. What additional structure is required to move from a computational experiment to a rigorous mathematical argument?

### 10. Can the mechanisms be simplified enough to make their interaction experimentally measurable without destroying the essential mathematics?

These are better questions than simply asking whether a simulation "looks turbulent."

---

# 21. Possible Next Experiments

The current laboratory can be extended in several directions.

## Experiment A — Energy Budget Decomposition

Measure the contribution of:

* advection
* pressure
* viscosity
* forcing

to the total energy evolution.

The goal would be to verify numerically which terms contribute to global energy change.

---

## Experiment B — Resolution Convergence

Run the same experiment at:

```text
N = 32
N = 64
N = 128
N = 256
```

and measure:

* divergence
* \(L_2\) error
* \(L_\infty\) error
* energy
* vorticity

The question:

> Does the observed behaviour converge as resolution increases?

---

## Experiment C — Timestep Stability Map

Systematically vary:

$$
\Delta t
$$

and determine where the solver transitions from stable to unstable.

This would turn numerical stability itself into a measurable object.

---

## Experiment D — Vorticity Concentration

Track:

$$
\|\omega(t)\|_\infty
$$

alongside:

$$
E(t)
$$

and the characteristic core size.

The question:

> Can concentration increase strongly while global energy remains controlled?

---

## Experiment E — 3D Minimal Vortex-Stretching Model

Construct a minimal computational model that isolates vortex stretching without pretending to solve the full 3D Millennium problem.

The purpose would be to measure how stretching changes vorticity amplification.

---

# 22. What Makes This Project Different?

The project is not primarily about:

* using a CFD library
* generating pretty flow animations
* reproducing a textbook derivation
* claiming a new mathematical theorem

The project is about building understanding through computation.

The important transition is:

```text
Equation
   ↓
Assumption
   ↓
Algorithm
   ↓
Measurement
   ↓
Failure
   ↓
Investigation
   ↓
Evidence
```

That makes the notebook closer to a computational research lab than a conventional tutorial.

---

# 23. Project Structure

```text
navier-stokes-first-principles-lab/
│
├── navier_stokes_first_principles_lab.ipynb
│
└── README.md
```

The notebook intentionally contains the full investigation so that a reader can follow the intellectual progression in one place:

```text
intuition
   ↓
equation
   ↓
discretization
   ↓
solver
   ↓
validation
   ↓
experiments
   ↓
failure modes
   ↓
investigation
   ↓
OpenAI connection
   ↓
conclusions
```

---

# 24. Tools

The project was developed using:

* Python
* NumPy
* Matplotlib
* Google Colab
* Git
* GitHub

The numerical methods are implemented directly rather than relying on a high-level CFD solver.

---

# 25. Running the Project

The easiest way to explore the project is to open:

```text
navier_stokes_first_principles_lab.ipynb
```

in Google Colab or another Jupyter environment.

Run the notebook from the beginning so that the experiments build on the definitions and numerical operators introduced earlier.

---

# 26. Reproducibility

The experiments are intended to be reproducible from the notebook.

Where possible, experiments expose parameters such as:

* grid resolution
* timestep
* viscosity
* simulation duration
* initial velocity
* forcing

This allows the reader to modify one variable and observe how the resulting system changes.

That is important because a single simulation run is only one observation.

A better experiment asks whether the observation survives controlled changes.

---

# 27. A Note on the Navier–Stokes Millennium Prize Problem

The Navier–Stokes existence and smoothness problem asks fundamental questions about whether sufficiently smooth initial conditions for the 3D incompressible Navier–Stokes equations necessarily remain smooth for all time, among other equivalent formulations of the problem.

This repository does not attempt to provide a mathematical solution.

The computational work here is intentionally narrower:

> Build enough of the mathematical and computational machinery to experimentally understand the mechanisms involved, then compare that intuition with the advanced construction reported by OpenAI.

Any statement about proving or disproving the Millennium Prize problem would require a level of mathematical rigor far beyond what a numerical notebook can provide.

---

# 28. Final Takeaway

I started with an equation that looked intimidating:

$$
\frac{\partial\mathbf{u}}{\partial t}
+
(\mathbf{u}\cdot\nabla)\mathbf{u}
=
-\nabla p
+
\nu\nabla^2\mathbf{u}
+
\mathbf{f}
$$

The goal was not to memorize it.

The goal was to unpack it.

What does velocity mean?

What does a derivative mean here?

Why does a moving particle accelerate?

Why is advection nonlinear?

Why does pressure appear?

Why does pressure have to enforce incompressibility?

Why does viscosity smooth?

Why can peak velocity and total energy behave differently?

Why does three-dimensional vortex stretching matter?

And finally:

> What would a computational experiment actually allow me to say about a problem as deep as Navier–Stokes regularity?

The answer is not a proof.

But it is a working laboratory.

And that laboratory makes the difference between:

```text
"I know the Navier–Stokes equation."
```

and:

```text
"I can construct it, test it, break it,
measure what happened, and explain what
the evidence does and does not establish."
```

---

## Status

**Project type:** Computational research / first-principles investigation

**Primary artifact:** `navier_stokes_first_principles_lab.ipynb`

**Focus:** 2D incompressible Navier–Stokes

**Validation:** Taylor–Green vortex

**Core themes:**

* velocity fields
* material acceleration
* nonlinear advection
* pressure projection
* viscosity
* vorticity
* energy
* concentration
* Reynolds number
* numerical stability
* vortex stretching intuition
* finite-time blow-up intuition
* OpenAI Navier–Stokes construction

**Important limitation:** This project does not prove or disprove the Navier–Stokes Millennium Prize Problem.

---

## Author

**Tarun**

Built as a first-principles computational investigation into one of the most difficult problems in mathematical physics.

The objective was not to make the problem look simple.

It was to make my understanding less shallow.
