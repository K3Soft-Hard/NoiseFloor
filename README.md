# NoiseFloor

**Iterative Sky-Ratio Masking, Self-Supervised Sensor Denoising & Grain-Floor Synthesizer for Astrophotography**

NoiseFloor separates random sensor read noise and Poisson photon shot noise from faint emission nebulosity and dark absorption dust lanes (LDN). Instead of blurring the whole image uniformly, it isolates the empty sky background, flattens sky mottling down to a controlled natural grain floor, and protects structural filaments with BayesShrink Starlet wavelet thresholding and a self-supervised Blind-Spot U-Net.

> **Note:** This software was designed and developed with the assistance of AI.

---

## Project Structure: No `src/` Folder

This project intentionally has **no `src/` directory, build step, or compiled binaries**. Python is already human-readable source code—every mathematical equation, wavelet filter, neural network layer, CLI command parser, and UI widget lives directly in the root files so you can inspect, audit, and modify the exact logic immediately:

* **`noisefloor_engine.py`** — All signal processing, Starlet wavelets, sky-ratio masking, FITS header/camera detection, StarNet++ CLI orchestration, PyTorch Blind-Spot U-Net, and the `NoiseFloorAPI` / `.nfs` script engine.
* **`noisefloor_ui.py`** — The 4-step OLED dark studio interface, interactive CLI shell, Process Console, and FITS Inspector (run this file to start the app).
* **`NoiseFloor_PixInsight.js`** — Native PixInsight PJSR client script that exports the active view as a 32-bit FITS (with camera headers intact), invokes NoiseFloor, and updates the view in-place with Undo support.
* **`noisefloor_config.json`** — Auto-generated on first run to store launch mode (`cli` or `gui`), StarNet++ paths/strides, pedestal settings, and automatic camera-to-`.pm` sensor profile mappings.

---

## Modular Release System (`E_x`, `U_x`, `P_x`, `U+E+P_x`)

Because the denoising/AI engine, the Tkinter studio UI, and the PixInsight PJSR client are updated independently, releases use a modular versioning scheme:

* **`E_<number>`** — Engine release (`noisefloor_engine.py`)
* **`U_<number>`** — UI & CLI release (`noisefloor_ui.py`)
* **`P_<number>`** — PixInsight PJSR Client release (`NoiseFloor_PixInsight.js`)
* **`U+E+P_<number>`** — Synchronized full-stack release where UI, Engine, and PixInsight Client share the same milestone number (for example, `U+E+P_2`)

### Highest-Version Rule
When downloading releases, always pick the **highest available `E_<number>`**, the **highest available `U_<number>`**, and the **highest available `P_<number>`** independently—even if their version numbers do not match (for example, pair **`E_14`** with **`U_12`** and **`P_2`**). Ensure the active files in your project folder are named:
* `noisefloor_engine.py`
* `noisefloor_ui.py`
* `NoiseFloor_PixInsight.js`

---

## 1. Download & Name the Release Files

1. Go to the **Releases** page on GitHub.
2. Download the latest **`U+E+P_<number>`** release, or download the highest **`E_<number>`**, **`U_<number>`**, and **`P_<number>`** files individually.
3. Place `noisefloor_engine.py` and `noisefloor_ui.py` into a single folder (for example `NoiseFloor/`), and keep `NoiseFloor_PixInsight.js` in that same folder (or inside your PixInsight scripts directory).

---

## 2. WINDOWS Installation & Launch

Open **PowerShell** inside your `NoiseFloor` folder and create an isolated `.venv`:

```powershell
# 1. Create and activate .venv
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# 2. Install core scientific and astro image libraries
python -m pip install --upgrade pip
python -m pip install numpy scipy astropy tifffile pillow
```
*(If PowerShell blocks `Activate.ps1`, run `.\.venv\Scripts\python.exe -m pip ...` directly for every command below.)*

### PyTorch Setup (GPU CUDA vs. CPU-Only)
Only install `torch` (do **not** install `torchvision` or `torchaudio`, which saves ~500 MB):

* **GPU Acceleration (CUDA):** Run `nvidia-smi` to check your `CUDA Version: XX.X`, then install the matching or lower wheel:
  * **CUDA 13.2+:** `python -m pip install torch --index-url https://download.pytorch.org/whl/cu132`
  * **CUDA 12.8 – 13.1:** `python -m pip install torch --index-url https://download.pytorch.org/whl/cu128`
* **CPU-Only (~200 MB, no dedicated GPU needed):**
  ```powershell
  python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
  ```

### Verify & Launch
```powershell
.\.venv\Scripts\python.exe -c "from noisefloor_engine import COMPUTE_INFO; print(COMPUTE_INFO)"
.\.venv\Scripts\python.exe noisefloor_ui.py
```

---

## 3. LINUX Installation & Launch

Open a terminal inside your `NoiseFloor` folder and create an isolated `.venv`:

```bash
# 1. Ensure Tkinter and venv are installed (Debian/Ubuntu-based distributions)
sudo apt update && sudo apt install -y python3-venv python3-tk

# 2. Create and activate .venv
python3 -m venv .venv
source .venv/bin/activate

# 3. Install core scientific and astro image libraries
python -m pip install --upgrade pip
python -m pip install numpy scipy astropy tifffile pillow
```

### PyTorch Setup (GPU CUDA vs. CPU-Only)
Only install `torch` (do **not** install `torchvision` or `torchaudio`, which saves ~500 MB):

* **GPU Acceleration (CUDA):** Run `nvidia-smi` to check your `CUDA Version: XX.X`, then install the matching or lower wheel:
  * **CUDA 13.2+:** `python -m pip install torch --index-url https://download.pytorch.org/whl/cu132`
  * **CUDA 12.8 – 13.1:** `python -m pip install torch --index-url https://download.pytorch.org/whl/cu128`
* **CPU-Only (~200 MB, no dedicated GPU needed):**
  ```bash
  python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
  ```

### Verify & Launch
```bash
./.venv/bin/python -c "from noisefloor_engine import COMPUTE_INFO; print(COMPUTE_INFO)"
./.venv/bin/python noisefloor_ui.py
```

---

## 4. First-Run CLI, Studio GUI & Automation Scripts (`.nfs`)

### Hybrid CLI / GUI Lifecycle
* **First Launch:** NoiseFloor starts in interactive **CLI mode** (`noisefloor> `) and auto-generates `noisefloor_config.json`.
* **Permanent Switch to GUI:** Type **`gui`** at the `noisefloor>` prompt. NoiseFloor saves `launch_mode: "gui"` in `noisefloor_config.json` and launches the visual Studio interface—every future launch will open the GUI automatically unless switched back to CLI in **Settings** (or via `--cli`).

### Process Console & `.nfs` Pipeline Scripts
You can run commands inside the top-bar **Console** in the GUI, in the interactive CLI, or from the terminal using `--cmd` / `--script`:

```text
# Example automated pipeline (.nfs)
select
fits
starnet auto
sensor auto
custom 0.85
log "session_log.txt"
save "denoised_output.fits"
```

* **`select` / `open <file>`** — Pick or load a FITS/TIFF/PNG image and auto-detect camera/telescope headers (e.g., ZWO Seestar S30 Pro / S50 / ASI) to preselect its matching `.pm` sensor profile.
* **`starnet <auto|run|skip>`** — Automatically separate stars via StarNet++ v2 CLI (with reversible MTF stretch for linear FITS) and recombine untouched stars at export.
* **`sensor <auto|default|skip|path.pm>`** or **`train [name]`** — Apply, train, or save named `.pm` sensor profiles in `sensor_models/`.
* **`auto`** | **`custom <0.10–1.00>`** | **`advance <flat, keep, sens, dust, neb>`** — Run denoising and sky-ratio flattening.
* **`generate [out.nfs]`** / **`script <file.nfs>`** — Generate a reusable `.nfs` script from your current session or execute an existing script.

---

## 5. PixInsight Integration (`P_<number>` Client)

1. Open **PixInsight** and go to **`Script -> Feature Scripts...`** -> click **Add**, and select the folder containing **`NoiseFloor_PixInsight.js`** (or run it once via **`Script -> Execute Script File...`**).
2. With an active image window open, launch **`Script -> NoiseReduction -> NoiseFloor`**.
3. Verify the auto-detected paths for your `.venv` Python executable and `noisefloor_ui.py`, select your **StarNet++**, **Sensor AI Profile**, and **Tuning Mode** options, and click **Run NoiseFloor**.
4. PixInsight exports a 32-bit FITS with full camera headers, executes NoiseFloor, and updates your active view in-place with full PixInsight Undo support.

---

## 6. Uninstalling PyTorch or Resetting `.venv`

* **Remove only PyTorch & CUDA libraries:**
  ```bash
  python -m pip uninstall -y torch
  python -m pip cache purge
  ```
* **Wipe the entire virtual environment:**
  * **Windows (PowerShell):** `Remove-Item -Recurse -Force .venv`
  * **Linux:** `rm -rf .venv`
