# Getting started with SADKAT on macOS (Python 3.13)

This guide takes you from a fresh Mac to running the SADKAT notebooks. It uses conda (Miniconda) for the Python environment, and covers both classic Jupyter and VS Code for running notebooks. Each step is short; copy the commands into Terminal in order.

By the end you will have your own fork of the group repository on GitHub, cloned to your Mac, a `sadkat` conda environment running Python 3.13, the `sadkat` package installed in editable mode (so edits to `src/sadkat/*.py` are picked up without reinstalling), and the eight notebooks `01_solvents.ipynb` … `08_everything.ipynb` built and running.

---

## 1. Prerequisites

**Command-line developer tools** (provides `git` and compilers). If you have not installed them before:

```bash
xcode-select --install
```

**Miniconda.** If `conda --version` already works in Terminal, skip this. Otherwise download the macOS installer for your chip (Apple Silicon = arm64, Intel = x86_64) from <https://docs.conda.io/en/latest/miniconda.html>, run it, then close and reopen Terminal. Check with:

```bash
conda --version
```

**An editor (optional but recommended).** [VS Code](https://code.visualstudio.com/) with the *Python* and *Jupyter* extensions from Microsoft. You can also use plain Jupyter in the browser; both routes are covered in step 7.

**A GitHub account**, and a way for `git` on your Mac to sign in to it. The simplest is the GitHub CLI:

```bash
brew install gh        # needs Homebrew (https://brew.sh); or download from https://cli.github.com
gh auth login          # choose GitHub.com → HTTPS → "Login with a web browser"
```

`gh auth login` also sets up `git` to use your GitHub login, so `git push` works without extra passwords or tokens.

---

## 2. Fork the repository and get the code

You will work in **your own fork** of the group repository, `grif-lab/sadkat-acrg`. A fork is your personal copy on GitHub: you can push to it freely, and when you have something to share you open a *pull request* back to the group repository.

### 2a. Fork on GitHub

Either on the website: go to <https://github.com/grif-lab/sadkat-acrg>, click **Fork** (top right), keep the owner as your own account and the name as `sadkat-acrg`, and click **Create fork**. Then clone it:

```bash
mkdir -p ~/src
cd ~/src
git clone https://github.com/<your-username>/sadkat-acrg.git
cd sadkat-acrg
git remote add upstream https://github.com/grif-lab/sadkat-acrg.git
```

Or do all of that in one step with the GitHub CLI:

```bash
mkdir -p ~/src
cd ~/src
gh repo fork grif-lab/sadkat-acrg --clone
cd sadkat-acrg
```

(`gh repo fork --clone` adds the `upstream` remote for you.)

### 2b. Check your remotes

```bash
git remote -v
```

You should see two remotes:

| Remote | Points to | Used for |
|---|---|---|
| `origin` | `github.com/<your-username>/sadkat-acrg` | your fork: you push your work here |
| `upstream` | `github.com/grif-lab/sadkat-acrg` | the group repo: you pull updates from here |

### 2c. Work on a branch

Keep `main` as a clean copy of the group repo and do your project work on a branch:

```bash
git switch -c fyp-reactive-uptake
```

Everything below assumes you are in the repository root, `~/src/sadkat-acrg`.

---

## 3. Create the Python 3.13 environment

```bash
conda create -n sadkat python=3.13 -y
conda activate sadkat
python --version          # should print Python 3.13.x
```

You need to run `conda activate sadkat` in every new Terminal window before working on SADKAT. Your prompt shows `(sadkat)` when it is active.

---

## 4. Install the dependencies

```bash
pip install --upgrade pip
pip install numpy scipy matplotlib pandas chemicals jupytext jupyter ipywidgets
```

What each one is for: `numpy`, `scipy`, `pandas` and `matplotlib` are the numerical and plotting stack; `chemicals` supplies thermophysical property data; `jupytext` converts the Python sources into notebooks; `jupyter` and `ipywidgets` run the notebooks and the interactive GUI in `05_gui`. (`setup.py` lists all of these except `jupyter` and `ipywidgets`, which the code also needs.)

`tkinter` is also used (for the file-save dialog in the GUI). Conda's Python includes it; check with:

```bash
python -c "import tkinter; print('tkinter OK')"
```

---

## 5. Install SADKAT itself (editable)

```bash
pip install -e .
```

The `-e` ("editable") flag links the installed package to `src/sadkat/`, so any change you make to the source is used next time you import it (after restarting the notebook kernel). This differs from the README, which suggests `pip install --user .`; editable is better when you are developing the code.

Check the install:

```bash
python -c "from sadkat.droplet import *; print('sadkat imports OK')"
```

If this fails with `ValueError: mutable default <class 'numpy.ndarray'> for field velocity is not allowed`, see [Troubleshooting](#mutable-default-valueerror) — the fix is a four-line change.

**Optional: register the environment as a named Jupyter kernel.** This makes it easy to pick the right Python in both Jupyter and VS Code:

```bash
python -m ipykernel install --user --name sadkat --display-name "Python 3.13 (sadkat)"
```

---

## 6. Build the notebooks

The notebooks are generated from the Python source files; they are not stored in git (`*.ipynb` is in `.gitignore`).

```bash
python make-notebooks.py
```

You should see each notebook being built followed by its table of contents, and eight files appear in the repository root:

| Notebook | Built from | What it covers |
|---|---|---|
| `01_solvents.ipynb` | `solvents.py` | Solvent data structures, Kelvin effect, water and alcohol properties |
| `02_solutes.ipynb` | `solutes.py` | Solutes and solution properties |
| `03_gas.ipynb` | `gas.py` | The surrounding gas / environmental conditions |
| `04_droplet.ipynb` | `droplet.py` | The droplet model and a first simulation from raw Python |
| `05_gui.ipynb` | `gui.py` | Interactive GUI for setting up and running simulations; parameter sweeps |
| `06_benchmarking.ipynb` | `benchmarking.py` | Comparisons with Kulmala, Su, EDB and FDC data; sensitivity analysis |
| `07_equations.ipynb` | `equations.py` | Appendix A: the full set of model equations |
| `08_everything.ipynb` | all of the above | Everything in one notebook (equations appendix at the end) |

**Rebuild whenever you change a source file.** Each notebook contains its own copy of the code from its `.py` file, so an edit to `droplet.py` only shows up in `04_droplet.ipynb` (and `08_everything.ipynb`) after you rerun `python make-notebooks.py`.

---

## 7. Open and run the notebooks

The notebooks load data and a matplotlib style file using paths relative to the repository root (e.g. `notebookStyle.mplstyle`, `src/sadkat/kulmala_data.npy`, `src/sadkat/EDB data for benchmarking/`). Always open them from the repository root, where `make-notebooks.py` puts them.

### Option A: VS Code

1. `code ~/src/sadkat-acrg` (or *File → Open Folder…* and choose `sadkat-acrg`).
2. Open e.g. `04_droplet.ipynb`.
3. Click **Select Kernel** (top right) → *Python Environments* (or *Jupyter Kernel*) → choose **sadkat** / **Python 3.13 (sadkat)**.
4. *Run All*.

### Option B: Jupyter in the browser

```bash
conda activate sadkat
cd ~/src/sadkat-acrg
jupyter notebook
```

A browser tab opens; click a notebook to open it. If you registered the kernel in step 5, choose *Kernel → Change kernel → Python 3.13 (sadkat)*.

### A sensible first run

Start with `04_droplet.ipynb` (section 3.2 runs a droplet simulation from plain Python), then `05_gui.ipynb` for the interactive version. `06_benchmarking.ipynb` is the longest and slowest; run it once you know the basics work.

---

## 8. Day-to-day workflow

```bash
cd ~/src/sadkat-acrg
conda activate sadkat
# edit files in src/sadkat/
python make-notebooks.py      # regenerate notebooks after editing
```

Then restart the kernel in your open notebook (VS Code: *Restart*; Jupyter: *Kernel → Restart*) and rerun. Because the package is installed in editable mode, you never need to rerun `pip install -e .` unless `setup.py` changes.

### Saving your work to your fork

Commit little and often, and push to your fork so your work is backed up on GitHub:

```bash
git add src/sadkat/droplet.py          # add the files you changed (notebooks are ignored by git)
git commit -m "Short description of what changed and why"
git push -u origin fyp-reactive-uptake   # first push of the branch; afterwards just: git push
```

### Getting updates from the group repo

When the group repository gets new commits, bring them into your fork:

```bash
git switch main
git pull upstream main        # update your local main from grif-lab
git push origin main          # keep your fork's main in step
git switch fyp-reactive-uptake
git merge main                # bring the updates into your branch
```

### Sharing your work (pull request)

When you want your changes reviewed or merged into the group repository, push your branch and open a pull request against `grif-lab/sadkat-acrg`:

```bash
git push
gh pr create --repo grif-lab/sadkat-acrg --base main --fill
```

or, on GitHub, open your fork and click **Contribute → Open pull request**.

---

## 9. Troubleshooting

### Mutable default ValueError

```
ValueError: mutable default <class 'numpy.ndarray'> for field velocity is not allowed: use default_factory
```

Python 3.11 and later refuse dataclass defaults that are mutable, such as numpy arrays. The original SADKAT code predates this. The fix is three edits:

- `src/sadkat/common.py`: change `from dataclasses import dataclass` to `from dataclasses import dataclass, field`
- `src/sadkat/gas.py` (in `Environment`) and `src/sadkat/droplet.py` (in `UniformDroplet`): change each
  `velocity: np.array=np.zeros(3)` / `position: np.array=np.zeros(3)` to
  `velocity: np.array = field(default_factory=lambda: np.zeros(3))` (and the same for `position`)

Then rerun `python make-notebooks.py` and restart the kernel. As a bonus, each droplet and environment now gets its own zero vector rather than all of them sharing one array.

### `ModuleNotFoundError: No module named 'sadkat'` (or `numpy`, `jupytext`, …)

The notebook or Terminal is using a different Python from the `sadkat` environment. In Terminal, run `conda activate sadkat` and check `which python` points into `.../envs/sadkat/`. In VS Code or Jupyter, reselect the **sadkat** kernel. If `sadkat` itself is missing, rerun `pip install -e .` from the repo root with the environment active.

### `FileNotFoundError` for `notebookStyle.mplstyle` or benchmarking data

The notebook's working directory is not the repository root. Make sure you are opening the notebooks that `make-notebooks.py` wrote into `~/src/sadkat-acrg/`, not copies moved elsewhere. In a notebook you can check with `import os; os.getcwd()`.

### `ModuleNotFoundError: No module named '_tkinter'`

Your Python was built without Tk. This does not happen with conda's Python; if you are using Homebrew's Python instead, run `brew install python-tk` and recreate the environment.

### The GUI's "save" file dialog hangs or does nothing

The save button opens a native Tk file dialog from inside the notebook kernel, which can misbehave on macOS (it may open behind other windows or hang). Check for a hidden window (Cmd-Tab / Mission Control). If it stays stuck, restart the kernel and save results from code instead: `droplet.complete_trajectory(trajectory)` returns a pandas DataFrame, so `history = droplet.complete_trajectory(trajectory); history.to_csv('results.csv')` works.

### Widgets in `05_gui` show as text instead of controls

`ipywidgets` is not installed in the kernel's environment. Run `pip install ipywidgets` with the `sadkat` environment active, then reload the notebook.

### `make-notebooks.py` fails with `No module named 'jupytext'`

`pip install jupytext` with the `sadkat` environment active.

### GitHub won't let me fork (the Fork button is greyed out or goes to an existing repo)

GitHub allows only one fork of a repository family per account. If you have already forked the original `tranqui/sadkat` (or any other fork of it), GitHub treats `grif-lab/sadkat-acrg` as the same family and sends you to your existing fork. Either delete or rename that old fork on GitHub and try again, or ask your supervisor. If `grif-lab/sadkat-acrg` is private, you also need to be given access to it before you can fork it.

### `git push` asks for a password or says permission denied

Run `gh auth login` again (step 1), and check that `origin` points to **your** fork, not `grif-lab` (`git remote -v`). You can fix a wrong `origin` with `git remote set-url origin https://github.com/<your-username>/sadkat-acrg.git`.

### A note on filename case

`benchmarking.py` loads `src/sadkat/kulmala_rh_list.npy`, but the file on disk is `kulmala_RH_list.npy`. This works on macOS because its file system ignores case by default, but it would fail on Linux (e.g. a cluster). Worth fixing if you ever run the benchmarks elsewhere.

---

## Quick reference

```bash
# one-off setup
xcode-select --install
brew install gh && gh auth login
mkdir -p ~/src && cd ~/src
gh repo fork grif-lab/sadkat-acrg --clone
cd sadkat-acrg
git switch -c fyp-reactive-uptake
conda create -n sadkat python=3.13 -y
conda activate sadkat
pip install numpy scipy matplotlib pandas chemicals jupytext jupyter ipywidgets
pip install -e .
python -m ipykernel install --user --name sadkat --display-name "Python 3.13 (sadkat)"
python make-notebooks.py

# every session
cd ~/src/sadkat-acrg
conda activate sadkat
python make-notebooks.py      # after editing src/sadkat/*.py
jupyter notebook              # or open the folder in VS Code
git add <files> && git commit -m "..." && git push   # save work to your fork
```
