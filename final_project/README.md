# Receivers for binary coherent state discrimination

A pulse of light arrives carrying either $|{+}\alpha\rangle$ or $|{-}\alpha\rangle$, the two states equally likely. Decide which. Because the states are not orthogonal — their overlap is $\langle{+}\alpha|{-}\alpha\rangle = e^{-2|\alpha|^2}$, never zero — no measurement decides correctly every time, and the question is only how small the error probability $P_e$ can be made.

Quantum mechanics sets the floor. The Helstrom bound, the minimum over all measurements permitted by the theory,

$$P_e^{\mathrm{Hel}} = \tfrac{1}{2}\left(1 - \sqrt{1 - e^{-4|\alpha|^2}}\right)$$

is a statement about what is achievable, not about how. This project builds three receivers that are physically realisable with a beam splitter, a local oscillator and a photodetector, and asks how close each of them gets. Throughout, $|\alpha|^2$ is the mean photon number of the full pulse, $\beta$ is a displacement applied before detection, and $P_e$ is the probability of announcing the wrong state averaged over the two equally likely hypotheses.

## The three notebooks

**[`1.homodyne_receiver.ipynb`](1.homodyne_receiver.ipynb)** — Homodyne detection measures the $x$ quadrature and answers according to its sign. The two hypotheses give Gaussian marginals centred at $\pm\sqrt{2}|\alpha|$ with unit-scale width, and the error is the overlapping tail,

$$P_e^{\mathrm{hom}} = \tfrac{1}{2}\,\mathrm{erfc}\!\left(\sqrt{2}\,|\alpha|\right)$$

The notebook derives this twice over: numerically, by building the coherent states in a truncated Fock space with QuTiP, computing their Wigner functions and integrating out $p$ to get the marginals; and analytically, by the formula above. The two agree. The numerical route is the slower and less accurate of the two, and it is there because it makes the truncation and grid choices visible — the Fock cutoff scales with $|\alpha|^2$ and the sampling is fixed at twenty points per unit width, with a normalisation assertion that fires if either is too coarse.

Homodyne is a Gaussian measurement, and its error decays like $e^{-2|\alpha|^2}$ against the bound's $e^{-4|\alpha|^2}$. The gap widens without limit.

**[`2.kenedy_receiver.ipynb`](2.kenedy_receiver.ipynb)** — The Kennedy receiver displaces the incoming state by $\beta$ and counts photons. Choosing $\beta = -\alpha$ nulls one hypothesis exactly, sending $|{+}\alpha\rangle$ to vacuum while $|{-}\alpha\rangle$ becomes $|{-}2\alpha\rangle$; any click then means $H_-$, and silence means $H_+$. Since vacuum never clicks, one hypothesis is never misread, and the error is the probability that $|{-}2\alpha\rangle$ happens to stay silent:

$$P_e^{\mathrm{Ken}} = \tfrac{1}{2}e^{-4|\alpha|^2}$$

This carries the same exponential as the bound, so unlike homodyne it does not fall progressively further behind. But expanding the bound's square root for large $|\alpha|$ gives $\tfrac14 e^{-4|\alpha|^2}$, so Kennedy sits a factor of two above it, forever.

The notebook then asks whether a better $\beta$ closes the gap. Displacing past exact nulling gives up the guarantee that $H_+$ never clicks, in exchange for making $H_-$ click more often, and the optimum trades these off at the root of

$$4ab = \ln\frac{b+a}{b-a}$$

solved by bracketed root-finding, with the bracket opened at the next representable double above $a$ because the expression diverges there. The optimised receiver does help where the two hypotheses are still badly confused — at $|\alpha| = 0.5$ it pulls the error from $1.80$ down to $1.32$ times the bound — but by $|\alpha| = 1.5$ it is back at $2.00$ times the bound and stays there. Whatever is missing is not a better value of $\beta$.

**[`3.dolinar_receiver.ipynb`](3.dolinar_receiver.ipynb)** — The main notebook, described below.

## The Dolinar receiver

What the Kennedy receiver never does is react. Its displacement is fixed before the pulse arrives and is the same at the end as at the beginning, even though by then the detector has been telling it things. The Dolinar receiver chops the pulse into $N$ time slots and lets each slot's displacement depend on every click seen so far, and with that one change it reaches the Helstrom bound exactly.

The notebook develops this in three steps, each of which is written up in full as prose with numbered equations that the code then refers back to line by line.

Chopping alone achieves nothing, and the notebook shows this first because it fails. Conserving energy across the slots gives each slot amplitude $a = |\alpha|/\sqrt{N}$, and running Kennedy independently in every slot returns $\tfrac12 e^{-4|\alpha|^2}$ — the Kennedy result again, the exponential factorising straight back into itself. The calculation never used the fact that slot 4 could have known what slot 3 did.

The click history is then compressed. What the receiver knows is its posterior belief $p = P(H_+ \mid \text{everything observed so far})$, which is a property of the receiver's information and not of the pulse. Carried as the log-odds $\lambda = \ln\frac{p}{1-p}$, Bayesian updating becomes addition, and the confidence $m = \tanh(\lambda/2)$, which runs from $-1$ to $+1$, is the single number the control law needs. This $\lambda$ is a sufficient statistic: two histories arriving at the same $\lambda$ are not merely similar but interchangeable, since from that moment they apply the same displacement, see the same outcome statistics and reach the same final guess.

The displacement that belief implies is then derived, giving the control law

$$\beta^* = -\frac{a}{m}, \qquad m = \tanh(\lambda/2)$$

which nulls the currently favoured hypothesis when the receiver is confident and pushes far outside the two amplitudes when it is not.

### Backward induction

Evaluating the receiver means evaluating the whole $N$-slot recursion, and this is where the project's computational weight sits.

Written out literally, $P_e$ is a sum over histories. A history $h = (o_1,\dots,o_N)$ is the full record of what the detector did, it occurs with probability $P(h)$, and it leaves the receiver holding a final belief from which the guess costs $\min(p_h, 1-p_h)$. Nothing about

$$P_e = \sum_h P(h)\,\min(p_h,\,1-p_h)$$

is approximate. The trouble is that there are $2^N$ terms, and the branching is genuine rather than bookkeeping: feedback makes one slot's statistics depend on what earlier slots showed, so the tree does not factor. At the $N = 400$ used for the final curve this is not a large number, it is an impossible one.

Sufficiency is what rescues it. Branches that merge need not be carried separately, so instead of following individual paths one tabulates a function of $\lambda$ and lets every path arriving there share the entry. The tree becomes an axis. Concretely, define the value function $V_k(\lambda)$ as the lowest error probability still reachable when the belief is $\lambda$ and $k$ slots remain,

$$V_0(\lambda) = \min(p,\,1-p), \qquad
V_k(\lambda) = \min_\beta \sum_{o} \Big[pL_+ + (1-p)L_-\Big]\, V_{k-1}\!\left(\lambda + \ln\frac{L_+}{L_-}\right), \qquad
P_e = V_N(0)$$

where $L_\pm$ is the likelihood of outcome $o$ under each hypothesis and the answer is read at $\lambda = 0$ because the two hypotheses start equally likely. The index $k$ counts slots remaining rather than slots elapsed, so the recursion runs against the direction in which the experiment is lived — the pulse arrives at $V_N$ and departs at $V_0$, while the computation starts at $V_0$ and climbs. It has no choice about this: evaluating a slot requires knowing what each of its outcomes is worth, which is a statement about the future, and the only layer whose future is empty is the last one.

The cost drops from $2^N$ to $N$ passes over a grid of $n_\lambda$ belief values, which is what makes $N = 400$ and a two-hundred-point sweep in $|\alpha|$ tractable at all, and what makes the animation over $N$ possible.

### Two implementations

`dolinar_error_theory` uses the closed-form policy $\beta^* = -a/m$ directly, so the minimisation over $\beta$ disappears and the control axis collapses into the belief axis. The displacements are computed once before the loop, since how many slots remain changes what a slot is worth but not what it should do.

`dolinar_error_sopt` keeps the same recursion but chooses $\beta$ by search over a grid of candidates. This is exactly optimal for the discretisation in use rather than for continuous time, and it is the reference the closed form is judged against — at moderate $N$ the difference is real, because the closed form is the continuous-time answer.

### Numerical care

Three choices decide whether the answer is right rather than merely plausible, and each is documented in the notebook alongside the number that justifies it.

The belief grid has to resolve one slot's evidence. The smallest step a slot can produce is $4a^2 = 4|\alpha|^2/N$, reached once the receiver is confident. If the grid spacing exceeds that, $\lambda$ and its update land in the same cell, interpolation returns the same value for both, and the slot is silently recorded as having taught nothing — no error is raised, just a wrong number. `_auto_n_lam` inverts the requirement, choosing $n_\lambda$ so that one step spans ten cells: $9001$ points at $|\alpha| = 1, N = 100$, and $36001$ at $N = 400$. Refining ten times further moves the result by less than one part in $10^4$.

The candidate displacements have to be dense where they matter. Once the receiver is confident, $|\beta^*| = a/|m|$ approaches $a$ from above, so writing candidates as $|\beta| = a + s$ with $s$ spaced geometrically from zero puts the resolution exactly there and places exact nulling itself on the grid. Uniform spacing of the same size is measurably worse: at $|\alpha| = 1, N = 60$ it returns $4.91\times10^{-3}$ where the geometric grid returns $4.75\times10^{-3}$.

The magnitude of $\beta$ has to be capped. The receiver starts at $\lambda = 0$, where $m = 0$ and $-a/m$ is infinite, and a finite slot cannot absorb an infinite displacement the way continuous time can. The single-shot Kennedy optimum is used as the ceiling, which introduces no free parameter since the physics already supplies that value, and the band of beliefs where the clip is active narrows as $N$ grows. Running with `cap=False` shows why it is needed: $\beta$ diverges in the first slot, both hypotheses are driven far above one photon, both click with certainty, the likelihood ratio is one, and the belief never moves at all.

### Results

The final figure puts all four receivers on one axis against the bound, over $|\alpha| \in [0.05, 2]$. Homodyne falls away fastest. Ideal and optimised Kennedy both settle at twice the bound. The Dolinar curve at $N = 400$ lies on the bound — drawn thick and semi-transparent with the bound laid over it as a thin marked line, since otherwise the two hide each other.

The last cell animates the Dolinar curve over the number of slots, from $N = 1$, which is a single optimised Kennedy shot, up to $N = 400$. Each of the 48 frames is one full backward induction. The frame values are every $N$ up to 12 and geometric after that, because the curve does nearly all of its moving in the first few slots while the cost of a frame grows like $N$. It takes a few minutes to precompute.

## Note on collaboration

The final project was done as a team of three with [Mariana](https://github.com/marianabagulho) and Inês; the joint repository is [`marianabagulho/final_project_Mariana_Ines_Andrew`](https://github.com/marianabagulho/final_project_Mariana_Ines_Andrew).

We each built all three receivers independently and then compared, so the versions here are my own rather than my share of a split. What is in this directory is that independent line of work. For the final presentation I took the Dolinar receiver, and the backward-induction implementation described above is the part I developed and presented.
