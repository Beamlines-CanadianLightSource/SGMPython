# SGMAnalysis

A Python package for analyzing and visualizing data from the SGM beamline. It provides tools for both single-energy **Map Scans** and multi-energy **Stack Scans**, with advanced features for XEOL (X-ray Excited Optical Luminescence) and chemometric analysis (PCA/K-Means).

## Installation

```bash
pip install .
```

## Features

- **Efficient Data Handling**: Lazy loading of high-volume SDD, MCC, and XEOL binary files.
- **Integrated Metadata**: Automatically parses HDF5 scan metadata and coordinates.
- **Interactive Dashboards**: Jupyter-ready Plotly widgets for real-time ROI selection and linked spatial-spectral analysis.
- **Advanced Analysis**: PCA and K-Means clustering to identify chemically distinct phases.
- **Robust Preprocessing**: Built-in handling for detector glitches, dead time, and "jagged" snake scans (roll correction).
- **Comprehensive Exports**: Single-method CSV exports including I0, TEY, and integrated XEOL intensities.

---

## 1. Map Scans (Single Energy)

A `MapScan` is used for analyzing a single spatial map recorded at a fixed energy.

```python
from sgmanalysis import MapScan

# Load the scan
scan = MapScan('path/to/my_map_scan.h5')

# Generate a quick overview of all detectors
# channel_roi: [start_bin, end_bin] of the SDD spectrum
scan.plot_overview(channel_roi=(80, 101), roll_shift=-6)

# Access raw data
sdd1_data = scan.get_sdd_data('sdd1')  # 2D array: (pixels, bins)
xeol_data = scan.xeol_data             # 1D array: XEOL spectrum
```

---

## 2. Stack Scans (Energy Series)

A `StackScan` manages a series of maps taken at different excitation energies (e.g., an XANES stack).

```python
from sgmanalysis import StackScan

# Load the stack
stack = StackScan('path/to/my_stack_scan.h5')

# Plot summary with pre-edge/post-edge comparison
stack.plot_summary(
    channel_roi=(20, 40), 
    map_roi=[-1.5, 1.5, -0.5, 0.5], # Spatial ROI [x1, x2, y1, y2]
    roll_shift=-6,
    xeol_roi=(100, 200)             # ROI for XEOL energy dependence
)

# Export summary data to CSV
# Automatically includes I0 (ch1), TEY (ch2), and XEOL total intensity
stack.export_csv(
    "summary_data.csv",
    channel_roi=(20, 40),
    mcc_channels=[3, 4],            # Optional: include extra MCC channels
    xeol_roi=(100, 200)
)
```

---

## 3. XEOL (Optical Luminescence)

The package provides specialized tools for XEOL analysis.

```python
from sgmanalysis import plot_xeol

# Quick plot of a XEOL spectrum from an H5 scan or a .bin file
plot_xeol('path/to/scan.h5')

# Access XEOL data for a specific energy in a stack
# XEOL data is stored in a dictionary: {energy: 1D_spectrum_array}
available_energies = list(stack.xeol_data.keys())
spectrum = stack.xeol_data[available_energies[0]]
```

---

## 4. PCA & K-Means Clustering

Identify different chemical phases by clustering pixel spectra across all energies in a stack.

```python
# 1. Perform the analysis
# Combine data from all 4 detectors for maximum sensitivity
results = stack.analyze_pca_kmeans(
    detector_names=['sdd1', 'sdd2', 'sdd3', 'sdd4'], 
    channel_roi=(20, 40), 
    n_clusters=3
)

# 2. Visualize and save
# Saves spatial data to 'phase_analysis.csv' and 
# cluster spectra to 'phase_analysis_spectra.csv'
stack.plot_pca_kmeans(results, roll_shift=-6, outfile="phase_analysis")
```

---

## 5. Interactive Dashboards (Jupyter)

The package includes interactive dashboards built with `Plotly FigureWidgets`. These allow you to select regions on a map to see the corresponding summed spectra or PFY (Partial Fluorescence Yield).

```python
from sgmanalysis import interactive_map_analysis, interactive_stack_analysis

# Interactive dashboard for a single map
interactive_map_analysis(scan, detector_name='sdd1')

# Interactive dashboard for a stack (multi-detector)
# Syncs selection across all maps and PFY plots
interactive_stack_analysis(stack, detector_names=['sdd1', 'sdd2'])
```

## Dependencies

- **Data**: `h5py`, `numpy`
- **Visualization**: `matplotlib`, `plotly`
- **Analysis**: `scikit-learn`
- **UI**: `ipywidgets`, `anywidget`
