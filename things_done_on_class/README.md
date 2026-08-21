# Class work

Daily exercise work from the first two weeks of SCIQIS 2026, following the tutorials and exercises in [qpit/sciqis](https://github.com/qpit/sciqis). These are working notebooks kept as they were written, not polished writeups — the polished work is in [`../final_project`](../final_project).

The course moved from Python tooling and NumPy into quantum circuits, then into continuous-variable states and QuTiP, and finally into real experimental data. The notebooks follow that arc.

**Circuits.** `quantum_cirquit.ipynb` builds a three-qubit simulator from nothing but NumPy — states as column vectors, gates as matrices, multi-qubit operators assembled with Kronecker products — and runs a ladder ansatz of `Ry` rotation layers alternating with a CNOT ladder. `qiskit_practice.ipynb` builds the same circuit in Qiskit with `AerSimulator` and `Statevector`, as a cross-check that the hand-rolled version is right and as a look at what a real framework does differently.

**Visualisation.** `ladder_ansatz.ipynb` is the [improve-my-plot](https://github.com/qpit/sciqis/blob/2026/exercises/improve-my-plot.md) exercise, taking the global parity $\langle Z^{\otimes n}\rangle$ of that ansatz as a function of the rotation angle $\theta$ and drawing it five different ways, to see which choices actually make the structure legible. `making_nice_graph.ipynb` is the same data in a shared-axis grid over qubit count and layer depth, with derivatives and zero crossings pulled out. `Visualisation.ipynb` follows the course demo on amplitude and phase modulation, using animations and `ipywidgets` sliders.

**Continuous variables and QuTiP.** `cv_tutorial.ipynb` is my submission to the [tutorialize continuous variables](https://github.com/qpit/sciqis/blob/2026/exercises/tutorialize_continuous_variables.md) exercise, building CV states from the ground up with $\hbar$, $m$ and $\omega$ kept explicit rather than set to one, so the dimensions stay visible. `qutip_cavity_qed_tutorial.ipynb` works through the course QuTiP notebook from basic operators up to cavity QED. `SHO.ipynb` is a scratch file.

**Real data.** `analyse-squeezing-data.ipynb` is the largest of these, re-analysing measured data from a squeezed light source: reading LeCroy scope traces and HDF5 files, estimating spectra with `welch` and `periodogram`, and fitting with `curve_fit` and `least_squares`. `kitten-tomography.ipynb` generates a cat-like state by a beam splitter and a photon click, reconstructs it, and compares the model against the state actually obtained.
