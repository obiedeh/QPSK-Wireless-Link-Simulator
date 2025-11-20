<!-- Copilot / AI agent instructions for the QPSK-Wireless-Link-Simulator repo -->
# Quick Guide for AI Coding Agents

Purpose: Help an AI agent be immediately productive in this repository by documenting the
project architecture, concrete coding patterns, and developer workflows discovered in the code.

- **Big picture**: This repo implements a baseband QPSK link simulator. Core DSP lives in
  `qpsk_modem.py` (symbol mapping, RRC pulse shaping, matched filter, mod/demod). Experiments
  and visualizations are in notebooks: `main.ipynb`, `channel.ipynb`, and `plots.ipynb`.

- **Data flow (typical run)**: bits -> `bits_to_qpsk` -> `qpsk_modulate` (RRC + upsample) ->
  `apply_channel` (AWGN / Rayleigh in `channel.ipynb`) -> matched filter + downsample
  (`qpsk_demodulate`) -> `qpsk_to_bits` -> BER calculation (in `main.ipynb`).

- **Key files to inspect/edit**:
  - `qpsk_modem.py` — canonical DSP implementations. Use its functions when adding or testing
    algorithms (e.g., `bits_to_qpsk`, `qpsk_modulate`, `rrc_filter`, `qpsk_demodulate`).
  - `main.ipynb` — experiment runner that computes BER curves and calls plotting helpers.
  - `channel.ipynb` — contains `apply_channel` (AWGN and Rayleigh fading); when converting to a
    module, keep the function signature: `apply_channel(tx_signal, snr_db=..., fading=bool)`.
  - `plots.ipynb` — plotting helpers used by `main.ipynb`; modify these notebooks for new
    visualizations.

- **Concrete implementation notes / patterns** (copy these exactly when writing tests or fixes):
  - Gray mapping in `bits_to_qpsk`: mapping pairs -> {00: 1+1j, 01: -1+1j, 11: -1-1j, 10: 1-1j}
    and symbols are normalized by `1/sqrt(2)` to unit average power.
  - Default pulse shape: `num_taps = 8 * sps + 1`, `beta = 0.35`, `sps = 8` (common convention here).
  - RRC filter normalizes energy via `h /= sqrt(sum(h**2))`.
  - Matched filtering assumes perfect timing: `offset = len(h_rrc)//2` and samples taken every `sps`.
  - Flat-fading equalization is scalar division by `channel_coef` in `qpsk_demodulate()`.

- **Repository realities & agent rules**:
  - This repo mixes standalone Python module (`qpsk_modem.py`) with Jupyter notebooks (experiments).
    Many modules referenced in notebooks (e.g., `channel`, `plots`) currently live as `.ipynb` files,
    so changing them for importable code often requires converting a notebook to a `.py` module
    (use `jupyter nbconvert --to script <notebook>` or extract the cell code).
  - Preserve notebook structure: when editing `.ipynb` programmatically, keep existing
    `metadata.id` values for existing cells and add new cells instead of merging into existing ones.
  - When adding a new Python module (e.g., `channel.py`) ensure the function signatures used in
    `main.ipynb` stay compatible (`apply_channel(tx_signal, snr_db, fading)` and returned values
    match what `main.ipynb` expects: typically `(rx_signal, h)`).

- **Developer workflows / commands** (PowerShell / `pwsh.exe`) — practical commands to run or test:
  - Start interactive development: `pip install numpy scipy matplotlib jupyter` then `jupyter lab`.
  - Run notebook headlessly (CI or quick verify):
    `jupyter nbconvert --to notebook --execute main.ipynb --ExecutePreprocessor.timeout=600`
  - Quick module sanity check from shell:
    `python -c "import numpy as np; import qpsk_modem as m; b=np.random.randint(0,2,1000); tx,syms,h=m.qpsk_modulate(b); rx=tx; bits_hat=m.qpsk_demodulate(rx,h,8,len(syms)); print((b[:len(bits_hat)]!=bits_hat).sum())"`

- **Dependencies**: At minimum expect `numpy`, `scipy`, `matplotlib`, and `jupyter` (the README
  mentions a `requirements.txt` but it may be missing — verify and create one if you add/require
  specific package versions).

- **When proposing code changes**:
  - Prefer making algorithmic changes in `qpsk_modem.py` (unit-testable) and keep notebooks
    as orchestration/visualization layers. If you update notebook code, also update or add a
    plain `.py` module so scripts can import the functionality.
  - Include a short, runnable example that exercises the change (use the quick module sanity
    check above as a template).

If anything above is unclear or you want more detail (for example: exact expected return shapes
from `apply_channel`, or a suggested `requirements.txt`), say which area to expand and I will iterate.
