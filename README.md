# EEG Benchmark Tool

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

**An open-source Python script for automated EEG signal quality assessment, focusing on key performance metrics.**

This tool provides a standardized and transparent method for evaluating the quality of EEG signals, crucial for comparing different electrode types, acquisition systems, or processing pipelines. It calculates fundamental quality metrics using the MNE-Python library, adhering to established signal processing principles (inspired by resources like "Analyzing Neural Time Series Data: Theory and Practice" by Mike X Cohen).

The primary goal is to offer researchers and developers a reliable way to quantify EEG signal integrity through objective measurements.

## Key Features

*   **Automated Quality Metrics:** Calculates essential EEG performance indicators: SNR, CMRR Proxy (Line Noise), and Alpha Power Ratio.
*   **Quantify Signal Integrity:** Assess signal clarity, noise rejection, and physiological modulation detection.
*   **MNE-Python Powered:** Built on the robust and validated MNE library.
*   **Standardized & Transparent:** Follows established methods; open-source under GPLv3.
*   **Interactive Notebook:** Provided as a Jupyter Notebook for easy execution and exploration.
*   **Built-in Demonstration:** Includes MNE sample data for immediate testing.
*   **Adaptable for Your Data:** Easily configurable to analyze your own EEG recordings.
*   **Visual Results:** Generates comparison plots of the calculated metrics.

## Understanding the Metrics

This benchmark focuses on three fundamental indicators of EEG signal quality:

1.  **Signal-to-Noise Ratio (SNR) (dB):**
    *   **What it is:** Measures the power of the desired brain signal relative to the power of background noise within specific frequency bands.
    *   **Calculation:** Calculated as `10 * log10(Power in Signal Band / Average Power in Noise Bands)`. The signal band is typically set to a physiologically relevant range (e.g., Alpha: 8-13 Hz), while noise bands are often adjacent ranges assumed to contain primarily noise (e.g., Theta: 4-7 Hz, Beta: 14-30 Hz).
    *   **Why it's important:** A high SNR indicates that the brain activity of interest is clearly distinguishable from noise, which is essential for reliable analysis (e.g., detecting events, analyzing frequency content, building BCIs). A low SNR means the signal is obscured by noise.
    *   **Interpretation:** **Higher dB values are better.** A significant drop in SNR suggests increased noise contamination.

2.  **CMRR Proxy (Line Noise Power) (uV²/Hz):**
    *   **What it is:** An indirect assessment of the system's ability to reject common-mode interference, specifically focusing on ubiquitous power line noise (50 Hz or 60 Hz). It measures the noise *before* targeted filtering (like a notch filter) is applied.
    *   **Calculation:** Calculated as the average Power Spectral Density (PSD) within a narrow frequency band centered around the line noise frequency (e.g., 59.5-60.5 Hz for 60Hz noise), computed on data that has been band-pass filtered but *not* notch filtered.
    *   **Why it's important:** Line noise is a major artifact. A low power value in this band *before* notch filtering suggests the electrode setup and amplifier inherently possess good Common Mode Rejection Ratio (CMRR), minimizing this interference at the source. High power indicates susceptibility to environmental noise.
    *   **Interpretation:** **Lower values are better.** This metric reflects the "raw" contamination level before specific line noise removal.

3.  **Alpha Power Ratio (Unitless):**
    *   **What it is:** Compares the EEG power in the alpha frequency band (typically 8-13 Hz) between a baseline/resting state (commonly "eyes closed") and an active/attentive state (commonly "eyes open").
    *   **Calculation:** Calculated as `(Average Alpha Band Power during "Eyes Closed" condition) / (Average Alpha Band Power during "Eyes Open" condition)`. Power is derived from the PSD calculated over epochs corresponding to each condition.
    *   **Why it's important:** Detects the "alpha blocking" phenomenon – a fundamental aspect of brain activity where alpha oscillations are typically stronger during relaxed wakefulness with eyes closed and suppressed during visual input or mental effort. A system that reliably captures this modulation (ratio > 1) demonstrates good sensitivity to underlying physiological changes.
    *   **Interpretation:** **Values significantly greater than 1 are generally better,** indicating successful detection of alpha modulation. Noise or poor electrode contact can obscure the alpha rhythm, pushing the ratio closer to 1 or even below it. *(Note: In the sample data demonstration, time segments are used as proxies for eyes open/closed states, limiting the physiological interpretation for that specific run).*

## Demonstration Output

The notebook demonstrates the metric sensitivity by comparing the results from the original sample data to a version with added artificial noise:

![EEG Benchmark Comparison Plot](https://github.com/zavesone/EEG-Benchmark-Tool/raw/main/output.png)
*(Example plot showing how metrics change with added noise. Note the log scale for Line Noise Power).*

## Usage

This benchmark tool is provided as a Jupyter Notebook (`.ipynb`) file.

**Prerequisites:**

*   **Python 3.x**
*   **Jupyter Environment:** Jupyter Notebook or JupyterLab. Install via pip if needed:
    ```bash
    pip install jupyterlab notebook
    ```
*   **Required Python Libraries:**
    ```bash
    pip install mne numpy matplotlib scipy
    ```

**Running the Benchmark (Demonstration with Sample Data):**

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/zavesone/EEG-Benchmark-Tool.git
    cd EEG-Benchmark-Tool
    ```
2.  **Launch Jupyter:**
    ```bash
    jupyter lab  # Recommended
    # or
    # jupyter notebook
    ```
3.  **Open the Notebook:** In the Jupyter interface, open `EEG_Signal_Quality_Benchmark.ipynb`.
4.  **Execute the Cells:** Run the cells sequentially (e.g., using `Shift + Enter`). The MNE sample data will be downloaded automatically if needed.
5.  **View Results:** Observe the output generated by the cells, including the printed metric values and the comparison plot at the end.

**Configuration:**

Modify the `config` dictionary within the notebook cell near the beginning to customize the analysis:

*   `channels_to_analyze`: **Crucial:** Update with channel names from *your* data.
*   `filter_bp_low`, `filter_bp_high`: Band-pass filter settings.
*   `line_noise_freq`: Set to 50 or 60 Hz.
*   `snr_signal_band`, `snr_noise_bands`: Bands for SNR calculation.
*   `alpha_band`: Band for Alpha Power Ratio.
*   `alpha_test_conditions`: **Important:** Adapt to match how conditions (e.g., eyes closed/open) are defined in *your* data (event markers or time ranges).
*   ... and others (see comments in the notebook).

## Using Your Own Data

Adapt the notebook to analyze your EEG recordings:

1.  **Modify Data Loading:**
    *   In the notebook, find the cell loading the sample data (`mne.io.read_raw_fif`...).
    *   Comment out or delete those lines.
    *   Add code to load *your* file using the appropriate `mne.io.read_raw_*` function (e.g., `read_raw_edf`, `read_raw_bdf`). Use `preload=True`.
    *   **Example (EDF):**
        ```python
        # Comment out or remove sample data loading lines

        # Add code to load your data:
        # your_eeg_filepath = 'path/to/your/data.edf' # <-- SET YOUR FILE PATH
        # try:
        #     raw_orig = mne.io.read_raw_edf(your_eeg_filepath, preload=True)
        #     print(f"Successfully loaded your data from: {your_eeg_filepath}")
        # except FileNotFoundError:
        #     print(f"Error: File not found at {your_eeg_filepath}. Please check the path.")
        # except Exception as e:
        #     print(f"An error occurred loading {your_eeg_filepath}: {e}")

        # Continue with channel selection and preprocessing...
        ```

2.  **Update Configuration (`config` dictionary):**
    *   Set `channels_to_analyze` to match your channel names.
    *   Adjust `line_noise_freq`.
    *   Correctly define `alpha_test_conditions` based on your experimental protocol (using event markers or precise time segments). If using events, you'll need to modify the alpha calculation cell to use `mne.find_events`.
    *   Verify all other parameters (`sfreq` if needed, filter settings, PSD parameters).

3.  **Run the Adapted Notebook:** Execute the cells. Skip or remove the noise addition section if desired.

## Contributing

Contributions are welcome! Please feel free to fork the repository, create a feature branch, and submit a pull request for improvements or bug fixes.

## License

This project is licensed under the **GNU General Public License v3.0** (or later). This means you are free to run, study, share, and modify the software. If you distribute modified versions, you must also distribute them under the GPLv3 (or later) license.

**Please ensure you include the `LICENSE` file** (containing the full GPLv3 text) in your repository when distributing this code. The full text can be found at: [https://www.gnu.org/licenses/gpl-3.0.txt](https://www.gnu.org/licenses/gpl-3.0.txt)

## Disclaimer

This tool is for research and development purposes only. It is **not a medical device** and is not intended for clinical diagnosis. The calculated metrics offer insights into signal quality but must be interpreted considering the specific recording context, data quality, and chosen configuration. The demonstration using sample data serves as a functional example and may not directly reflect performance with all EEG hardware or experimental conditions.
