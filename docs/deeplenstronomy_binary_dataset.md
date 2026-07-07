# Creating a Binary Strong-Lensing Dataset with deeplenstronomy

This guide explains how to use `deeplenstronomy` to simulate a binary image-classification dataset for gravitational lens detection.

The target labels are:

- `label = 1`: a strong lens is present. A foreground deflector has a mass profile and lenses a background source.
- `label = 0`: no strong lens is present. Foreground and background galaxies may appear, but the foreground object does not act as a lens.

The recommended pattern is to make each class a separate `GEOMETRY.CONFIGURATION_*` entry. This follows the classification workflow described in the deeplenstronomy documentation, where configuration names serve as discrete class labels.

## References

- deeplenstronomy home: <https://deepskies.github.io/deeplenstronomy/>
- Configuration file guide: <https://deepskies.github.io/deeplenstronomy/Notebooks/ConfigFiles.html>
- Classification and regression notebook: <https://deepskies.github.io/deeplenstronomy/Notebooks/Metrics.html>
- `make_dataset` API documentation: <https://deepskies.github.io/deeplenstronomy/docs/deeplenstronomy.html>

## Dataset Design

A deeplenstronomy configuration file is a YAML-style file with these main sections:

- `DATASET`: dataset name, size, output directory, and optional random seed.
- `COSMOLOGY`: cosmological parameters used by lenstronomy.
- `IMAGE`: detector and image-rendering parameters.
- `SURVEY`: bandpasses and observing conditions.
- `SPECIES`: all possible galaxies, mass profiles, point sources, and noise sources.
- `GEOMETRY`: which species appear in each simulated class/configuration.

For binary lens detection, keep the simulated classes balanced unless you intentionally want an imbalanced training set:

- `CONFIGURATION_1`: `LENS_PRESENT`, `FRACTION: 0.5`
- `CONFIGURATION_2`: `NO_LENS`, `FRACTION: 0.5`

The key modeling distinction is the foreground object:

- In `LENS_PRESENT`, the foreground `LENS` galaxy has a mass profile and appears in front of the `SOURCE` galaxy, so deeplenstronomy can lens the source light.
- In `NO_LENS`, use a foreground galaxy whose required mass profile has zero lensing strength. This creates a realistic non-lens contaminant image without strong lensing.

Starting in `deeplenstronomy` version `0.0.1.8`, every `SPECIES.GALAXY_*` entry must include at least one `MASS_PROFILE_*`. For galaxies that should contribute light but no lensing, add a valid placeholder mass profile such as `CONVERGENCE` with `kappa: 0.0`. This satisfies the config validator while leaving the simulated lensing behavior unchanged.

## Example Binary Dataset Config

Save this as `data/binary_lens_config.yaml` if you want to generate the dataset exactly as shown.

```yaml
DATASET:
    NAME: BinaryStrongLensDataset
    PARAMETERS:
        SIZE: 1000
        OUTDIR: data/binary_lens_raw
        SEED: 42

COSMOLOGY:
    PARAMETERS:
        H0: 70
        Om0: 0.3

IMAGE:
    PARAMETERS:
        exposure_time: 90
        numPix: 100
        pixel_scale: 0.263
        psf_type: 'GAUSSIAN'
        read_noise: 7
        ccd_gain: 6.083

SURVEY:
    PARAMETERS:
        BANDS: g,r,i,z,Y
        seeing: 0.9
        magnitude_zero_point: 30.0
        sky_brightness: 23.5
        num_exposures: 10

SPECIES:
    GALAXY_1:
        NAME: LENS
        LIGHT_PROFILE_1:
            NAME: SERSIC_ELLIPSE
            PARAMETERS:
                magnitude:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 18.0
                            maximum: 21.0
                center_x: 0.0
                center_y: 0.0
                R_sersic:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 2.0
                            maximum: 8.0
                n_sersic: 4
                e1:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
                e2:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
        MASS_PROFILE_1:
            NAME: SIE
            PARAMETERS:
                theta_E:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.7
                            maximum: 1.8
                e1:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.2
                            maximum: 0.2
                e2:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.2
                            maximum: 0.2
                center_x: 0.0
                center_y: 0.0
        SHEAR_PROFILE_1:
            NAME: SHEAR
            PARAMETERS:
                gamma1:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.08
                            maximum: 0.08
                gamma2:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.08
                            maximum: 0.08

    GALAXY_2:
        NAME: FOREGROUND_GALAXY
        LIGHT_PROFILE_1:
            NAME: SERSIC_ELLIPSE
            PARAMETERS:
                magnitude:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 18.0
                            maximum: 21.0
                center_x: 0.0
                center_y: 0.0
                R_sersic:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 2.0
                            maximum: 8.0
                n_sersic: 4
                e1:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
                e2:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
        MASS_PROFILE_1:
            NAME: CONVERGENCE
            PARAMETERS:
                kappa: 0.0
                ra_0: 0.0
                dec_0: 0.0

    GALAXY_3:
        NAME: SOURCE
        LIGHT_PROFILE_1:
            NAME: SERSIC_ELLIPSE
            PARAMETERS:
                magnitude:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 21.0
                            maximum: 24.5
                center_x:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
                center_y:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.25
                            maximum: 0.25
                R_sersic:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.5
                            maximum: 3.0
                n_sersic:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 1.0
                            maximum: 4.0
                e1:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.3
                            maximum: 0.3
                e2:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: -0.3
                            maximum: 0.3
        MASS_PROFILE_1:
            NAME: CONVERGENCE
            PARAMETERS:
                kappa: 0.0
                ra_0: 0.0
                dec_0: 0.0

    NOISE_1:
        NAME: POISSON_NOISE
        PARAMETERS:
            mean: 2.0

GEOMETRY:
    CONFIGURATION_1:
        NAME: LENS_PRESENT
        FRACTION: 0.5
        PLANE_1:
            OBJECT_1: LENS
            PARAMETERS:
                REDSHIFT:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.2
                            maximum: 0.6
        PLANE_2:
            OBJECT_1: SOURCE
            PARAMETERS:
                REDSHIFT:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.8
                            maximum: 2.0
        NOISE_SOURCE_1: POISSON_NOISE

    CONFIGURATION_2:
        NAME: NO_LENS
        FRACTION: 0.5
        PLANE_1:
            OBJECT_1: FOREGROUND_GALAXY
            PARAMETERS:
                REDSHIFT:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.2
                            maximum: 0.6
        PLANE_2:
            OBJECT_1: SOURCE
            PARAMETERS:
                REDSHIFT:
                    DISTRIBUTION:
                        NAME: uniform
                        PARAMETERS:
                            minimum: 0.8
                            maximum: 2.0
        NOISE_SOURCE_1: POISSON_NOISE
```

## Generate Images and Labels

Use `make_dataset` to generate images, then assign labels from the configuration each image came from.

```python
import numpy as np
import pandas as pd
import deeplenstronomy.deeplenstronomy as dl

dataset = dl.make_dataset(
    "data/binary_lens_config.yaml",
    save_to_disk=True,
    store_in_memory=True,
    image_file_format="npy",
    verbose=True,
)

lens_images = dataset.CONFIGURATION_1_images
no_lens_images = dataset.CONFIGURATION_2_images

X = np.concatenate([lens_images, no_lens_images], axis=0)
y = np.concatenate([
    np.ones(len(lens_images), dtype=np.int64),
    np.zeros(len(no_lens_images), dtype=np.int64),
])

lens_metadata = dataset.CONFIGURATION_1_metadata.copy()
lens_metadata["label"] = 1
lens_metadata["class_name"] = "LENS_PRESENT"

no_lens_metadata = dataset.CONFIGURATION_2_metadata.copy()
no_lens_metadata["label"] = 0
no_lens_metadata["class_name"] = "NO_LENS"

metadata = pd.concat([lens_metadata, no_lens_metadata], ignore_index=True)

print(X.shape)
print(y.shape)
print(metadata["label"].value_counts().sort_index())
```

Before training, shuffle the examples so the model does not see all lens images followed by all non-lens images.

```python
rng = np.random.default_rng(42)
indices = rng.permutation(len(y))

X = X[indices]
y = y[indices]
metadata = metadata.iloc[indices].reset_index(drop=True)
```

## Optional Lens Diagnostics

For development or debugging, call `make_dataset` with `solve_lens_equation=True`. This adds metadata columns such as `num_source_images-g`, `num_source_images-r`, and similar per-band fields. These columns can help confirm whether the lens-present class is producing multiple source images.

```python
diagnostic_dataset = dl.make_dataset(
    "data/binary_lens_config.yaml",
    store_in_memory=True,
    save_to_disk=False,
    solve_lens_equation=True,
    verbose=True,
)

diagnostic_dataset.CONFIGURATION_1_metadata.filter(like="num_source_images").head()
```

Do not use these diagnostic columns as model inputs unless that is an intentional metadata-based experiment. For image classification, the model should learn from pixels, not from simulation bookkeeping.

## Quality Checks

Run these checks before treating the dataset as training-ready.

```python
assert len(dataset.CONFIGURATION_1_images) == len(dataset.CONFIGURATION_2_images)
assert set(np.unique(y)).issubset({0, 1})
assert metadata["label"].value_counts().to_dict() == {1: len(lens_images), 0: len(no_lens_images)}
```

Visualize a few samples from each class.

```python
import matplotlib.pyplot as plt

def show_sample(image, title):
    # Images have shape (bands, height, width). Show the first band.
    plt.imshow(image[0], origin="lower", cmap="gray")
    plt.title(title)
    plt.axis("off")

plt.figure(figsize=(8, 4))
plt.subplot(1, 2, 1)
show_sample(lens_images[0], "label=1: lens present")
plt.subplot(1, 2, 2)
show_sample(no_lens_images[0], "label=0: no lens")
plt.tight_layout()
plt.show()
```

Check for leakage before model training:

- Do not include `class_name`, `CONFIGURATION_*`, filenames, row order, or output paths as input features.
- Shuffle after labels are created.
- Split train/validation/test after shuffling.
- Keep the same preprocessing for both classes.
- If you save arrays and labels separately, verify that image and label ordering remains aligned.

## Tiny Smoke Test

For a quick local test, temporarily change `SIZE` to `4` and keep the two fractions at `0.5`. Before shuffling, the expected labels are:

```python
[1, 1, 0, 0]
```

This confirms that each class generated two images and the label mapping is deterministic.
