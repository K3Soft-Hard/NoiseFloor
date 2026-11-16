# NoiseFloor

**Iterative Sky-Ratio Masking, Self-Supervised Sensor Denoising & Grain-Floor Synthesizer for Astrophotography**

NoiseFloor separates random sensor read noise and Poisson photon shot noise from faint emission nebulosity and dark absorption dust lanes (LDN). Instead of blurring the whole image uniformly, it isolates the empty sky background, flattens sky mottling down to a controlled natural grain floor, and protects structural filaments with BayesShrink wavelet thresholding and a self-supervised Blind-Spot U-Net.

> **Note:** This software was designed and developed with the assistance of AI.

---

## Project Structure: No `src/` Folder

This project intentionally has **no `src/` directory, build step, or compiled binaries**. Python is already human-readable source code—every mathematical equation, wavelet filter, neural network layer, and UI widget lives directly in the two root `.py` files so you can inspect, audit, and modify the exact logic immediately:

* **`noisefloor_engine.py`** — All signal processing, Starlet wavelets, parameter optimization, I/O, and the PyTorch Blind-Spot U-Net.
* **`noisefloor_ui.py`** — The 4-step studio interface (run this file to start the app).

---

## Modular Release System (`E_x`, `U_x`, `E+U_x`)

Because the denoising/AI engine and the Tkinter studio UI are updated independently, releases use a modular versioning scheme:

* **`E_<number>`** — Engine release
* **`U_<number>`** — UI release
* **`E+U_<number>`** — Synchronized full-stack release where both Engine and UI share the same milestone number (e.g., `E+U_9`)

### Highest-Version Rule
When downloading releases, always pick the **highest available `E_<number>`** and the **highest available `U_<number>`** independently—even if their numbers do not match (for example, pair **`E_10`** with **`U_8`**). Make sure to rename the two active files in your project folder to **`noisefloor_engine.py`** and **`noisefloor_ui.py`**.

---

## 1. Download & Name the Release Files

1. Go to the **Releases** page on GitHub.
2. Download the latest **`E+U_<number>`** release, or download the highest **`E_<number>`** engine release and highest **`U_<number>`** UI release separately.
3. Place both files into a single folder (for example `NoiseFloor/`) and ensure they are renamed to:
   * `noisefloor_engine.py`
   * `noisefloor_ui.py`

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

## 4. Uninstalling PyTorch or Resetting `.venv`

* **Remove only PyTorch & CUDA libraries:**
  ```bash
  python -m pip uninstall -y torch
  python -m pip cache purge
  ```
* **Wipe the entire virtual environment:**
  * **Windows (PowerShell):** `Remove-Item -Recurse -Force .venv`
  * **Linux:** `rm -rf .venv`
