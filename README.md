# Multi-View 3D Fish Tracking

[![DOI](https://zenodo.org/badge/1308073187.svg)](https://doi.org/10.5281/zenodo.22646946)

This repository contains the code developed for my Master's thesis on multi-view 3D fish tracking.

The project investigates the reconstruction of 3D fish trajectories from synchronized observations obtained from three cameras. The pipeline combines geometric triangulation with learning-based refinement using a Multi-Layer Perceptron (MLP) and a Transformer model.

## Overview

The code follows the main steps below:

1. Load the synthetic multi-camera data and 3D ground truth.
2. Process the 2D observations from the three cameras.
3. Reconstruct 3D fish positions using geometric triangulation.
4. Fill missing geometric estimates using interpolation.
5. Compute geometric, visibility, and temporal features.
6. Split the data by frame order into training, validation, and test sets.
7. Train an MLP to refine the geometric 3D estimates.
8. Train a Transformer using temporal windows of consecutive frames.
9. Apply both models to refine the reconstructed trajectories and fill missing positions.
10. Evaluate the different approaches against the 3D ground truth.

## Data

The experiments use a controlled synthetic dataset containing multiple fish observed by three cameras.

The dataset includes:

- 3D ground-truth fish positions
- 2D observations for each camera
- Camera intrinsic and extrinsic parameters
- Camera distortion parameters
- Visibility information
- Detection confidence information

The use of synthetic data provides known 3D positions that can be used as ground truth for evaluating the reconstruction pipeline.

## Geometric Reconstruction

The first stage reconstructs fish positions using multi-view geometric triangulation.

A 3D position can be triangulated when the same fish is visible in at least two cameras. The pipeline also computes geometric quality information, including:

- triangulation residual
- reprojection error
- number of visible cameras
- camera visibility flags

Missing 3D positions are initially filled using linear interpolation.

## Feature Engineering

The learning-based models use 22 input features containing geometric, temporal, and visibility information.

These features include information about:

- current 3D position
- velocity
- previous position
- previous velocity
- reprojection errors
- number of visible cameras
- camera visibility
- triangulation quality
- detection confidence
- tracklet length

The models learn a residual correction to the geometric estimate rather than predicting the complete 3D position directly.

The final prediction is therefore:

`refined position = geometric/interpolated position + predicted correction`

## MLP Refiner

The first learning-based approach is a Multi-Layer Perceptron implemented in PyTorch.

The network receives the 22 engineered features and predicts a three-dimensional correction corresponding to the X, Y, and Z coordinates.

The architecture contains:

- 22 input features
- hidden layer with 128 units and ReLU activation
- dropout
- hidden layer with 64 units and ReLU activation
- 3 output values representing the 3D correction

The model is trained using the Adam optimizer and Mean Squared Error (MSE) loss.

## Transformer Refiner

The second learning-based approach uses a Transformer encoder to include temporal information in the prediction.

Instead of processing each frame independently, the Transformer receives a window of 32 consecutive frames.

The architecture contains:

- linear projection of the 22 input features
- positional encoding
- Transformer encoder
- two Transformer encoder layers
- four attention heads
- final linear layer predicting the X, Y, and Z correction

The prediction for the central frame of each temporal window is used as the output.

## Temporal Data Split

The dataset is divided according to frame order:

- first 70% of frames: training
- next 10% of frames: validation
- last 20% of frames: testing

The temporal order is preserved instead of randomly splitting individual observations.

For the Transformer, temporal windows are constructed independently inside each subset. This prevents windows from crossing the boundaries between training, validation, and test data.

## Training

Both learning-based models use:

- Adam optimization
- Mean Squared Error loss
- data augmentation during training
- validation-based early stopping
- feature normalization based only on training statistics

The experiments can also be repeated using multiple random seeds to evaluate the stability of the learning-based models.

## Evaluation

The code supports two main evaluation settings:

**3D refinement:** comparison of the learning-based refined positions with the geometric triangulation on frames where triangulation is available.

**Trajectory completion:** evaluation on all frames with available ground truth, including frames where geometric reconstruction is missing because of limited camera visibility.

All evaluation is performed on the temporally separated test set.

## Requirements

The main Python dependencies are:

- NumPy
- Pandas
- PyTorch
- OpenCV
- Matplotlib
- Scikit-learn
- SciPy
- IPython

Install the required packages with:

```bash
python -m pip install -r requirements.txt

