# ENV - machine inventory and {BASELINE} environment plan

Written: 2026-09-12. Repo commit at time of writing: `{COMMIT}`
Phase 1 is **read-only inventory + a proposal**. Nothing was installed.

Every line is tagged **[V]** verified by a command run on this machine or a fetch
of the upstream source, or **[I]** inference / reasoning not yet executed.

---

## 1. Machine inventory  [V]

| Item | Value | How verified |
|---|---|---|
| GPU | NVIDIA GeForce RTX 5090, 1 unit | `nvidia-smi --query-gpu` |
| VRAM | 32607 MiB (~31.8 GiB) | `nvidia-smi --query-gpu` |
| Compute capability | **12.0 (sm_120, Blackwell)** | `nvidia-smi --query-gpu=compute_cap` |
| Driver, before 2026-10-20 (history) | 610.57.04 | `nvidia-smi` (E1, 2026-09-21) |
| Driver | 615.71.09 | `nvidia-smi` (E1 re-run, 2026-10-20; results/e1_environment_check_2026-10-20.json, run at `{COMMIT}`) |
| Max CUDA per driver, before 2026-10-20 (history) | 13.3 | `nvidia-smi` header, which the 610 series prints as "CUDA UMD Version" (E1) |
| Max CUDA per driver | 13.4 | `nvidia-smi` header "NVIDIA-SMI 615.71.09              KMD Version: 615.71.09     CUDA UMD Version: 13.4" (E1 re-run, 2026-10-20; results/e1_environment_check_2026-10-20.json, run at `{COMMIT}`) |
| Kernel module flavour, before 2026-10-20 (history) | open (NVIDIA UNIX Open Kernel Module, Dual MIT/GPL), built by DKMS from the open-flavour 610 driver package. 595.84 was the open flavour too | `/proc/driver/nvidia/version`, `modinfo nvidia`, `dpkg-query -W 'nvidia-*'` (E1); `dkms status` lists nvidia/610.57.04 only (2026-09-21) |
| Kernel module flavour | open (Dual MIT/GPL), built by DKMS from the open-flavour 615 driver package | `/proc/driver/nvidia/version`, `modinfo nvidia`, `dpkg-query -W 'nvidia-*'` (E1 re-run, 2026-10-20; results/e1_environment_check_2026-10-20.json, run at `{COMMIT}`) |
| Driver 595 leftovers (unused) | the 595 driver packages: removed on 2026-09-20 but not purged, dpkg state `rc` (config files only). No 595 kernel module is built | `dpkg-query -W -f='${Package} ${Version} ${db:Status-Abbrev}\n' '*595*'`, `awk '$3=="remove" && /-595/' /var/log/dpkg.log`, `dkms status` (2026-09-21) |
| nvcc on PATH | **12.8** (V12.8.93, build cuda_12.8.r12.8) | `nvcc --version` |
| CUDA toolkits present | /usr/local/cuda -> cuda-12, cuda-12.8 | `ls -d /usr/local/cuda*` |
| OS | Ubuntu 24.04.5 LTS, kernel 6.14.0-33-generic, x86_64 | `/etc/os-release`, `uname -r` (E1) |
| CPU | AMD Ryzen 7 9800X3D, 8 cores / 16 threads | `lscpu` |
| RAM | 91 GiB total, 84 GiB available; 95 GiB swap | `free -h` |
| Disk (repo, /) | NVMe SSD, 1.9T total | `df -B1` (E1 re-run, 2026-10-20; results/e1_environment_check_2026-10-20.json, run at `{COMMIT}`) |
| Other disk | second SSD, 1.9T | `df -h` |
| conda | 25.5.1 (Miniconda) | `conda --version` |
| mamba / micromamba | **not installed** | `command -v` |
| Existing envs | 23 (names omitted) | `conda env list` |

Disk is ample for the {DATASET} dataset. No cleanup needed.

**2026-09-21, E1** (results/e1_environment_check.json, run at `{COMMIT}`; its script landed unchanged as scripts/e1_env_check.py at `{COMMIT}`). The driver moved 595.84 -> 610.57.04 and a disk cleanup ran. Driver, Max CUDA per driver, OS, Disk and the kernel module above are E1's measurements (-> 1_stack.now); before the update the first three read 595.84, 13.2 and Ubuntu 24.04.4 LTS, and the flavour was not recorded. The 595-leftovers row comes from the read-only commands it names. GPU, VRAM, compute capability, nvcc and the CUDA toolkit directories were re-read by E1 and are unchanged, and the installed envs match section 6 (-> checks.all_envs_match_env_md); CPU, RAM, the other disk, conda and the env list were not re-measured. The bit-exactness canary held: rebuilding {CANARY_RUN} from its saved intermediate outputs reproduced the saved prediction file on every image (-> 4_canary.rebuild). End-to-end latency moved +1.35% against the previous driver (-> 4_canary.speed.comparison; results/e1_speed_{BASELINE}.json), and METHOD.md's latency reference is re-based on it (Dxxx).

**2026-10-20, E1 re-run** (results/e1_environment_check_2026-10-20.json; Dxxx). The driver moved 610.57.04 -> 615.71.09 (Max CUDA per driver 13.3 -> 13.4) between two phases, outside this repo's sessions. Verdict safe_to_proceed_to_method_step_0 = True, blocking checks failed: []. The script ran as the byte-identical copy outputs/tmp/e1_env_check.py (sha256 equal to scripts/e1_env_check.py), because it resolves the repository as its file's parents[2]; its canary and speed blocks compare the 2026-09-21 canary and speed-baseline runs (not re-run).

---

## 2. What {BASELINE} actually asks for  [V]

Fetched from github.com/{OWNER}/{BASELINE} README:

- Python, PyTorch and CUDA versions: **not specified anywhere in the README.**
- detectron2: **v0.3 "recommended", v0.6 also tested.** Install shown as:
  ```
  git clone https://github.com/facebookresearch/detectron2.git
  git checkout tags/v0.3
  python setup.py build develop
  ```
- Backbone {BACKBONE}; reported {SCORE} {METRIC} on {DATASET} at {FPS} FPS.
- Checkpoints hosted on {HOST} (human is fetching these; see DECISIONS Dxxx).

detectron2 upstream state  [V] (GitHub API):
- Last **release**: v0.6, 2021-11-15. No release in ~5 years.
- `main` last pushed 2026-08-19, repo **not archived**, 591 open issues.

---

## 3. The binding constraint: sm_120 kills the old stack

This is the single fact that decides the plan.

1. [V] The GPU reports compute capability **12.0**.
2. [V] The only nvcc installed is **12.8**; CUDA 12.8 is the first toolkit that can
   emit sm_120 code. (Consistent with what is installed, not independently proven.)
3. [V] PyTorch's `cu128` wheel index starts at **torch 2.7.0**. Versions with a
   cu128 cp311 x86_64 wheel: 2.7.0, 2.7.1, 2.8.0, 2.9.0, 2.9.1, 2.10.0, 2.11.0.
   Verified by listing https://download.pytorch.org/whl/cu128/torch/ .
4. [I] detectron2 v0.3's era is torch 1.5-1.7 / CUDA 10.x-11.0. Those builds contain
   no sm_120 SASS, no PTX that JITs to sm_120, and ship pre-Blackwell cuDNN/cuBLAS.

**Conclusion: reproducing the original stack is not an option here.** A torch 1.x
install would not run slowly or unreliably on this card - it would not execute at
all ("no kernel image is available for execution on the device"). There is no
version of "keep the old stack" that works on a 5090. Porting is not a preference,
it is the only path.

One piece of luck: system nvcc is 12.8 and the torch wheels are cu128, so the
extension compile and the runtime agree on the CUDA minor version.

---

## 4. Where it will actually break  [V - by reading upstream source]

I fetched the specific files rather than relying on recall. Results:

| Check | v0.3 | v0.6 | main |
|---|---|---|---|
| `deform_conv_cuda.cu` includes `THC/THC*` | no | no | no |
| `deform_conv_cuda.cu` uses `c10::optional` / `AT_CHECK` | no | no | no |
| `nms_rotated_cuda.cu`, `ROIAlignRotated_cuda.cu`, `cocoeval.cpp` legacy APIs | none | none | none |
| `setup.py` torch guard | `>= 1.5` | `>= 1.8` | `>= 1.8` |
| `setup.py` python_requires | `>=3.6` | `>=3.6` | `>=3.7` |
| `transform.py:46` default interp | **`Image.LINEAR`** | **`Image.LINEAR`** | `Image.BILINEAR` |

Two things this changes versus the usual folklore:

- **The CUDA sources are clean.** `deform_conv_cuda.cu` is byte-identical between
  v0.6 and main (31785 bytes each) and has no THC / AT_CHECK / c10::optional. The
  classic "old detectron2 won't compile on torch 2.x" failures are largely absent
  from the files DeformConv depends on. The compile is more likely to succeed than
  the reputation suggests.
- **The real blocker is Pillow, not CUDA.** `Image.LINEAR` was removed in Pillow 10.
  On v0.3/v0.6 this is an import-time `AttributeError`, not a subtle bug. Fixed on
  main. Pinning `Pillow<10` clears it in one line.
- Neither setup.py has an **upper** bound on torch or Python, so torch 2.7 / py3.10
  pass the version guards.

Remaining expected breakages, **[I] not yet tested**, in rough order of likelihood:

1. **`torch.load` weights_only default flipped to `True` in torch 2.6.** detectron2's
   `DetectionCheckpointer` calls `torch.load` without `weights_only=False`. A 2021-era
   .pth with pickled numpy objects will raise `UnpicklingError`. Very likely to hit us
   the moment we load the released checkpoint. Fix: patch the call or pass the flag.
2. **numpy 2.x.** detectron2 v0.6 predates it; the compiled extension and pycocotools
   both bind numpy's C ABI. Pin `numpy<2`. Also watch for removed `np.float`/`np.int`
   aliases (gone since numpy 1.24).
3. **`python setup.py build develop` is deprecated** with modern setuptools. Use
   `pip install -e . --no-build-isolation`; the flag matters so it builds against the
   torch already in the env rather than a fresh one pulled into an isolated builder.
4. **`TORCH_CUDA_ARCH_LIST` must be set to `12.0`** before building or nvcc emits the
   wrong architecture and every op fails at runtime.
5. **fvcore / iopath drift.** Pin `fvcore==0.1.5.post20221221`, `iopath==0.1.9`.
6. **Python 3.12 removed distutils** - avoid 3.12 for this env; use 3.10.
7. Assorted `torch.cuda.amp` deprecation warnings - noise, not failures.

---

## 5. Proposed environment (NOT executed)

Env name `{BASE_ENV}`, separate from the Phase-3 `{EVAL_ENV}` env.

```
conda create -n {BASE_ENV} python=3.10
conda activate {BASE_ENV}

pip install torch==2.7.1 torchvision --index-url https://download.pytorch.org/whl/cu128
pip install "numpy<2" "Pillow<10" fvcore==0.1.5.post20221221 iopath==0.1.9 \
            opencv-python-headless pycocotools timm

git clone https://github.com/facebookresearch/detectron2.git third_party/detectron2
cd third_party/detectron2 && git checkout tags/v0.6
export TORCH_CUDA_ARCH_LIST="12.0"
pip install -e . --no-build-isolation
```

**Version choices and why:**

- **torch 2.7.1+cu128** - the *earliest* torch that supports sm_120 [V]. Deliberate:
  detectron2 v0.6 is 2021 code, so the less C++ API drift between it and torch, the
  better. Newer torch buys Blackwell maturity but adds four more years of drift.
  Fallback 2.8.0 if 2.7.1 shows Blackwell instability.
- **detectron2 v0.6, not v0.3** - v0.6 is author-tested, and is 12 months closer to
  torch 2.x than v0.3. Same DeformConv CUDA source as main, so nothing is lost.
- **not `main`** - main fixes the Pillow issue for free, but carries ~5 years of
  Python-API drift away from what {BASELINE} was written against. That drift is the
  expensive failure mode (silent config/registry mismatches); the Pillow issue is a
  one-line pin. Keep main as the fallback if v0.6 will not compile.
- **Pillow<10** - forced by `Image.LINEAR` at transform.py:46 [V].
- **Python 3.10** - v0.6-era code; avoids the 3.12 distutils removal.

**Decision point I want to flag:** the recommendation above optimises for *fidelity to
the published result* (v0.6 + oldest working torch). If reproducing {PUBLISHED_SCORE} proves
painful, the alternative is detectron2 `main` + torch 2.9, which is a more comfortable
build but puts more distance between us and the number we are trying to hit. Per
CLAUDE.md rule 7, if this passes 2 hours I stop and report rather than grinding.

**Not yet verified:** whether {BASELINE}'s own code (not detectron2's) uses any
detectron2 API that changed between v0.3 and v0.6, and whether the {BACKBONE} + DCN
backbone compiles and runs on sm_120. Both need the actual build, which is Phase 4.

---

## 6. Installed environments - exact versions  [V]

Added 2026-09-15 at repo commit `{COMMIT}`. Sections 1-5 are the Phase 1 plan, written
before anything was installed; this section is the installed state, captured by code from the envs
themselves: `<env>/bin/python -m pip list --format=freeze`, `<env>/bin/python --version`,
`conda list -n <env>`, and `torch.__version__` / `torch.version.cuda` / `torch.backends.cudnn.version()`.
Build steps are read from `outputs/logs/` (gitignored).
The logs are INCOMPLETE: `scikit-learn` is installed in `{BASE_ENV}` and its clones but no log records
installing it. The freeze lists below, not the logs, are the record.

| env | python | built | how (from the logs) | key versions |
|---|---|---|---|---|
| `{BASE_ENV}` | 3.10.21 | 2026-09-13 | `conda create` spec `python=3.10`; torch from `https://download.pytorch.org/whl/cu128`; deps; editable detectron2 built with `TORCH_CUDA_ARCH_LIST=12.0` | torch 2.7.1+cu128, torchvision 0.22.1+cu128, detectron2 0.6, numpy 1.26.4, Pillow 9.5.0, fvcore 0.1.5.post20221221, iopath 0.1.9, pycocotools 2.0.11, timm 1.0.29, scikit-learn 1.7.2; torch runtime CUDA 12.8, cuDNN 90701 |
| `{TOOL_ENV}` | {PYTHON} | {DATE} | conda clone of `{BASE_ENV}` (`--offline`), then `pip install --no-deps {PACKAGE}=={VERSION}` ({DECISION}; script outputs/{PHASE}/envs/{TOOL_ENV}.sh, log outputs/logs/{LOG}; built in {TIME}, inside the session's cap) | as `{BASE_ENV}` plus {PACKAGE} {VERSION}, nothing removed; GPU check: {WHAT_RAN_AND_PEAK_MEMORY}; weights `checkpoints/{TOOL}/{FILE}` (not committed; {BYTES} bytes, sha256 {HASH}; {SOURCE_URL}; {LICENCE}) |

Tool envs are cloned from `{BASE_ENV}` and record what was added and that nothing was removed. A build
that fails, or weights whose download link is dead, still get a row: the error message, the log lines
and the time the link was tried.

No CUDA, cuDNN or compiler package comes from a conda channel in any env; conda supplies python plus
libgcc 15.2.0 / libstdcxx 15.2.0. The CUDA runtime is pip's `nvidia-*-cu12` wheels pulled in by torch.
detectron2's extension was compiled against the system CUDA 12.8 toolkit (section 1: the only nvcc on the
machine); `build_detectron2.log` ends `compiled _C OK` and `DCNv2 forward on sm_120 OK: (1, 8, 32, 32)`.

Requested specs as logged (`Collecting` / `Obtaining` lines with no `(from ...)` parent):

- `{BASE_ENV}`: `torch==2.7.1`, `torchvision`, `numpy<2`, `Pillow<10`, `fvcore==0.1.5.post20221221`, `iopath==0.1.9`, `opencv-python-headless`, `pycocotools`, `timm`, `scipy`, `-e third_party/detectron2`

third_party source trees (gitignored):

| repo | commit | tag | licence (from the clone's own file) | locally modified | patch in results/ |
|---|---|---|---|---|---|
| detectron2 | `d1e0456` | v0.6 | Apache-2.0 (`LICENSE`) | none | - |
| {BASELINE} | `{COMMIT}` | - | {LICENCE} (`LICENSE`) | `{MODIFIED_FILES}` | `results/patches/{NAME}.patch` |

Licences read 2026-09-18 from the LICENSE file in each clone at the pinned commit above (Dxxx); no
repository is missing one, so no "none found" row is needed. Copyright lines as they stand in those
files: [ one line per repository ]; detectron2 ships the unfilled Apache-2.0 appendix (`Copyright
[yyyy] [name of copyright owner]`, line 189) and names its holder only in per-file headers
(`detectron2/__init__.py:1`, "Copyright (c) Facebook, Inc. and its affiliates."). SHA-256 of each
file, first 16 hex: detectron2 `ceb78ee847e76f67`, {BASELINE} `{HASH}`.

Dxxx's release constraint is unchanged and is now on firmer ground: all are permissive, MIT and
Apache-2.0 both allow redistribution with the notice kept, and {BASELINE} does carry a licence, so
the project does not have to record an unlicensed base model. We still distribute our own code plus
clone-and-patch instructions and never third-party sources.

Patches verified 2026-09-15: applied to a clean `git archive` of the pinned commit, each reproduces
the live third_party files byte-for-byte, and a reverse `--dry-run` of the command below resolves against the
live tree. Reapply after a re-clone, from the repo root: `patch -p1 -d third_party < results/patches/<name>.patch`.
The {BASELINE} patch defaults `{DATA_ROOT_VAR}` to this workstation's absolute data path; set it anywhere else.

### Full pip freeze

<details><summary><code>{BASE_ENV}</code>: one block per env; the others are omitted in this public version</summary>

```text
absl-py==2.5.0
antlr4-python3-runtime==4.9.3
anyio==4.15.1
appdirs==1.4.4
black==21.4b2
certifi==2026.7.22
click==8.5.0
cloudpickle==3.1.2
contourpy==1.3.2
cycler==0.12.1
detectron2==0.6
exceptiongroup==1.3.1
filelock==3.32.3
fonttools==4.65.0
fsspec==2026.7.0
future==1.0.0
fvcore==0.1.5.post20221221
grpcio==1.83.1
h11==0.16.0
hf-xet==1.6.0
httpcore==1.0.9
httpx==0.28.1
huggingface_hub==1.31.0
hydra-core==1.3.6
idna==3.19
iopath==0.1.9
Jinja2==3.1.6
joblib==1.6.0
kiwisolver==1.5.1
Markdown==3.10.3
MarkupSafe==3.0.3
matplotlib==3.10.9
mpmath==1.3.0
mypy_extensions==1.1.0
networkx==3.4.2
numpy==1.26.4
nvidia-cublas-cu12==12.8.3.14
nvidia-cuda-cupti-cu12==12.8.57
nvidia-cuda-nvrtc-cu12==12.8.61
nvidia-cuda-runtime-cu12==12.8.57
nvidia-cudnn-cu12==9.7.1.26
nvidia-cufft-cu12==11.3.3.41
nvidia-cufile-cu12==1.13.0.11
nvidia-curand-cu12==10.3.9.55
nvidia-cusolver-cu12==11.7.2.55
nvidia-cusparse-cu12==12.5.7.53
nvidia-cusparselt-cu12==0.6.3
nvidia-nccl-cu12==2.26.2
nvidia-nvjitlink-cu12==12.8.61
nvidia-nvtx-cu12==12.8.55
omegaconf==2.3.1
opencv-python-headless==4.11.0.86
packaging==26.3
pathspec==0.12.1
Pillow==9.5.0
pip==26.2.1
portalocker==4.3.0
protobuf==7.36.1
pycocotools==2.0.11
pydot==4.0.1
pyparsing==3.3.2
python-dateutil==2.9.0.post0
PyYAML==6.0.3
regex==2026.9.10
safetensors==0.8.0
scikit-learn==1.7.2
scipy==1.15.3
setuptools==83.0.0
six==1.17.0
sympy==1.14.0
tabulate==0.10.0
tensorboard==2.21.0
tensorboard-data-server==0.7.2
termcolor==3.3.0
threadpoolctl==3.6.0
timm==1.0.29
toml==0.10.2
torch==2.7.1+cu128
torchvision==0.22.1+cu128
tqdm==4.70.1
triton==3.3.1
typing_extensions==4.16.0
Werkzeug==3.1.8
wheel==0.47.0
yacs==0.1.8
```

</details>
