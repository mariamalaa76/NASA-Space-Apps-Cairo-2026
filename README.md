# TerraMatch AI — CRISM Data Analysis

## Project Overview
TerraMatch AI aims to discover Earth locations with environmental and surface characteristics similar to selected regions on Mars. As a first step, we explored a CRISM observation to understand the Martian surface data and extract useful features.

## Workflow

### 1. Search the ODE Catalog
We used the **Mars Orbital Data Explorer (ODE) REST API** to search available Mars observations.

- **Why ODE?** It provides a searchable catalog of Mars mission products and their downloadable files.
- We filtered the results for **MRO** (Mars Reconnaissance Orbiter) and **CRISM**.

### 2. Select a CRISM MTRDR Product
We selected a **MTRDR** product (Map-Projected Targeted Reduced Data Record).

- **What is MTRDR?** A map-projected CRISM data product containing calibrated and corrected spectral information.
- **Why MTRDR?** It is suitable for spatial analysis and mineral-related spectral analysis.

### 3. Inspect the Product Files
We inspected the product file list and identified the files needed for analysis:

- **IF**: the hyperspectral image cube containing corrected I/F values.
- **WV**: the wavelength table, which links each spectral band to its wavelength.
- **SU**: spectral summary parameter maps derived from the spectral data.
- **XML / HDR**: metadata describing the product, image dimensions, data type, layout, and file offsets.

We focused on these files to avoid downloading unrelated browse images and supporting files.

### 4. Load the Hyperspectral Cube
We loaded the IF image as a NumPy memory-mapped array.

- The cube contains **489 spectral bands**, **204 lines**, and **318 samples**.
- Each pixel has a spectrum: one value for each wavelength.
- **Why use `np.memmap`?** It allows us to access a large binary image without loading the entire file into memory at once.

### 5. Match Bands to Wavelengths
We read the WV table and checked that its 489 wavelength records match the 489 bands in the cube.

- **Why?** Band numbers alone do not describe the physical part of the spectrum. Wavelengths are needed to interpret spectral features correctly.
- We preserved the original wavelength order and did not interpolate across spectral gaps.

### 6. Check Data Quality
We inspected the values and identified `65535` as the product's NoData value.

- We masked these values before calculating statistics or plotting.
- We checked the number of valid and invalid measurements.
- We also considered known CRISM spectral regions with calibration or processing concerns, rather than interpreting every spectral feature as a real surface signature.

### 7. Explore Spectral Signatures
We examined the spectrum of a selected pixel and plotted the full wavelength range, then focused on a more reliable spectral region.

- **Why?** Different materials interact with light differently, producing characteristic reflectance and absorption patterns.
- Local minima were treated as possible spectral features, not as confirmed mineral identifications.

### 8. Read Spectral Summary (SU) Maps
We extracted selected parameter maps from the SU file, including:

- **OLINDEX3** — an index used to investigate olivine-related spectral responses.
- **LCPINDEX2** — an index related to low-calcium pyroxene.
- **HCPINDEX2** — an index related to high-calcium pyroxene.

The map locations and byte offsets were read from the SU XML label.

- **Why use SU maps?** They provide precomputed spectral parameters, making it easier to explore mineral-related patterns across the image than analysing every spectrum manually.

### 9. Statistical and Spatial Analysis
For the selected maps, we calculated descriptive statistics, examined distributions, and computed correlations.

We also:
- Compared the spatial patterns of the three indices.
- Selected the highest-response 5% of valid pixels as an exploratory threshold.
- Examined connected pixel regions and overlap between high-response masks.
- Tested other percentile thresholds to see how sensitive the regions were to the chosen cutoff.

**Important:** These percentile masks show relative high index responses within this observation. They do not directly measure mineral abundance or prove that a mineral is present.

## Tools Used

| Tool | Purpose |
|---|---|
| Python | Main analysis language |
| `requests` | Sending requests to the ODE API and downloading product files |
| `xml.etree.ElementTree` | Reading PDS4 XML metadata and locating SU map offsets |
| `pandas` | Reading wavelength tables and organizing results |
| `NumPy` | Loading arrays, masking NoData, and calculating statistics |
| `Matplotlib` | Plotting spectra, maps, and distributions |
| `SciPy` | Connected-component analysis of thresholded regions |

## Current Result and Scope
We successfully explored one CRISM MTRDR observation, loaded its hyperspectral cube and wavelength information, examined data quality, and analysed three spectral summary parameter maps.

This is the **Mars-data exploration stage** of TerraMatch AI. Earth-data collection, comparable feature preparation, and the actual Earth–Mars similarity analysis are still required before identifying candidate Earth analog sites.
"""

output_path = "/mnt/data/README_TerraMatch_Summary.md"
pypandoc.convert_text(readme, "md", format="md", outputfile=output_path, extra_args=["--standalone"])
print(f"Created: {output_path}")
