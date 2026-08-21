# SCIQIS Summer 2026

Work from DTU's [10387 — Scientific Computing in Quantum Information Science](https://kurser.dtu.dk/course/10387), a three-week intensive course held in Lyngby from 3 to 21 August 2026. The course material it follows is at [qpit/sciqis](https://github.com/qpit/sciqis).

There are two directories here.

[`final_project/`](final_project) is the main body of work: three receivers for telling apart the two coherent states $|{+}\alpha\rangle$ and $|{-}\alpha\rangle$, built in order of increasing capability and ending at the Dolinar receiver, which attains the Helstrom bound — the lowest error probability quantum mechanics permits for this task. That directory has its own README with the physics and the implementation.

[`things_done_on_class/`](things_done_on_class) holds the daily exercise work from the first two weeks, following the course tutorials and demos. It is what the final project was built on top of rather than a result in itself, and it is documented briefly.

If you are here to look at one thing, look at [`final_project/`](final_project).

## Setup

The environment is managed with [uv](https://docs.astral.sh/uv/) and pinned by `uv.lock`, matching the course environment (NumPy, SciPy, Matplotlib, QuTiP, Qiskit, h5py, ipywidgets).

```bash
git clone https://github.com/andyfromko/sciqis_summer_2026
cd sciqis_summer_2026
uv sync
```

Then either launch `uv run jupyter lab`, or point your editor at the `.venv` interpreter that `uv sync` creates.

The final-project notebooks are self-contained and run top to bottom. The Dolinar notebook is the slow one: the error curve is evaluated at $N = 400$ time slots and takes a couple of minutes, and the animation cell at the end takes a few minutes more.
