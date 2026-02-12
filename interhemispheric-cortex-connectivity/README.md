# Interhemispheric Cortex Connectivity Microscopy Dataset

## Overview

This dataset contains high-resolution light-sheet microscopy volumes of mouse brains, acquired using ExA-SPIM (Expansion-Assisted Selective Plane Illumination Microscopy) to map interhemispheric cortical connectivity. The data captures callosal projection neurons and their axonal arbors spanning both brain hemispheres at sub-micron resolution.

The dataset is hosted on the [Registry of Open Data on AWS](https://registry.opendata.aws/pasteur-interhemispheric-cortex-connectivity/) through the AWS Open Data Sponsorship Program.

## Data Access

All data is stored in a public Amazon S3 bucket:

```
s3://[BUCKET-NAME-TBD]
```

No AWS account is required to browse or download data:

```bash
# List all data assets
aws s3 ls --no-sign-request s3://[BUCKET-NAME-TBD]/

# Download a specific data asset
aws s3 sync --no-sign-request s3://[BUCKET-NAME-TBD]/exaspim_[SUBJECT-ID]_[DATE]_[TIME]/ ./local_copy/
```

## Imaging Platform

| Parameter | Value |
|-----------|-------|
| Microscope | ExA-SPIM (Expansion-Assisted Selective Plane Illumination Microscopy) |
| Expansion factor | [e.g., 3× or 4×] |
| Effective optical resolution (lateral) | [e.g., ~300 nm] |
| Effective optical resolution (axial) | [e.g., ~800 nm] |
| Excitation wavelengths | [e.g., 488 nm, 561 nm, 639 nm] |
| Detection objective | [Describe lens / NA] |
| Camera | [Model] |
| Acquisition software | [e.g., Acquire / custom] |
| File format | OME-Zarr |

## Sample Preparation

[Describe your tissue preparation protocol:]
- Species: Mus musculus
- Strain: [e.g., C57BL/6J]
- Labeling strategy: [e.g., viral tracing with AAV-GFP / transgenic reporter / etc.]
- Target cell population: [e.g., callosal projection neurons in cortical layers II/III and V]
- Tissue clearing/expansion protocol: [e.g., expansion microscopy with X× gel, reference protocol]

## Bucket Organization

Following the [AIND data organization model](https://allenneuraldynamics.github.io/data.html), the bucket is organized as a flat list of data assets. Each data asset is a self-contained directory with the following naming convention:

```
<modality>_<subject-id>_<acquisition-date>_<acquisition-time>/
```

### Example structure

```
s3://[BUCKET-NAME-TBD]/
    exaspim_MOUSE001_2025-01-15_10-30-00/
        exaspim/
            image.ome.zarr/          # Multiscale OME-Zarr volume
                .zattrs
                .zgroup
                0/                   # Full resolution
                1/                   # 2× downsampled
                2/                   # 4× downsampled
                ...
        data_description.json        # Dataset-level metadata
        subject.json                 # Animal metadata (strain, age, sex, genotype)
        procedures.json              # Surgery, injection, perfusion details
        acquisition.json             # Microscope settings, tile layout, channels
        processing.json              # Post-acquisition processing steps
    exaspim_MOUSE001_2025-01-15_10-30-00_stitched_2025-01-16_08-00-00/
        stitched/
            fused.ome.zarr/          # Stitched and fused volume
        data_description.json
        subject.json
        procedures.json
        acquisition.json
        processing.json
    exaspim_MOUSE002_2025-02-01_14-00-00/
        ...
```

## Metadata Schema

Sidecar JSON metadata files conform to a schema inspired by [aind-data-schema](https://github.com/AllenNeuralDynamics/aind-data-schema/). Each data asset includes the following metadata files:

### `data_description.json`
- Dataset name, creation date, institution, funding sources
- Data level (raw / derived)
- Modality
- License information

### `subject.json`
- Subject ID, species, strain, sex, date of birth, age at imaging
- Genotype (if transgenic)

### `procedures.json`
- Surgical procedures (e.g., viral injections)
- Injection coordinates (AP, ML, DV), volumes, virus titers
- Perfusion and fixation protocol
- Expansion/clearing protocol details

### `acquisition.json`
- Microscope identifier and configuration
- Objective, magnification, numerical aperture
- Laser lines and power settings
- Tile layout (number of tiles, overlap)
- Voxel size (before and after expansion correction)
- Channel information (fluorophores, emission filters)

### `processing.json`
- Processing steps applied (stitching, deconvolution, registration)
- Software versions
- Input/output asset references

## Data Format: OME-Zarr

All imaging data is stored in [OME-Zarr](https://ngff.openmicroscopy.org/) format (OME-NGFF v0.4), enabling:

- **Multiscale access**: Multiple resolution levels allow efficient browsing without downloading full-resolution data
- **Cloud-native streaming**: Compatible with tools like Neuroglancer, napari, BigDataViewer
- **Chunked storage**: Data is stored in chunks optimized for partial reads over the network

### Viewing data remotely

You can view OME-Zarr data directly from S3 without downloading:

```python
# Using napari with napari-ome-zarr plugin
import napari

viewer = napari.Viewer()
viewer.open("s3://[BUCKET-NAME-TBD]/exaspim_MOUSE001_2025-01-15_10-30-00/exaspim/image.ome.zarr",
            plugin="napari-ome-zarr")
napari.run()
```

```python
# Using Python with zarr and dask
import zarr
import dask.array as da

store = zarr.storage.FSStore("s3://[BUCKET-NAME-TBD]/exaspim_MOUSE001_2025-01-15_10-30-00/exaspim/image.ome.zarr/0",
                              mode='r', anon=True)
z = zarr.open(store, mode='r')
data = da.from_zarr(z)
print(f"Shape: {data.shape}, Dtype: {data.dtype}, Chunks: {data.chunks}")
```

## License

This dataset is released under the [Creative Commons Attribution 4.0 International License (CC-BY-4.0)](https://creativecommons.org/licenses/by/4.0/).

## Citation

If you use this dataset, please cite:

> [Haiss et al. (YEAR). TITLE. JOURNAL. DOI: XXXX]

Additionally, please include the following data availability statement:

> Data are available at registry.opendata.aws/pasteur-interhemispheric-cortex-connectivity.

## Contact

- **Florent Haiss** — florent.haiss@pasteur.fr
- **Institut Pasteur** — https://www.pasteur.fr

## Acknowledgements

This dataset is hosted through the [AWS Open Data Sponsorship Program](https://aws.amazon.com/opendata/open-data-sponsorship-program/).
