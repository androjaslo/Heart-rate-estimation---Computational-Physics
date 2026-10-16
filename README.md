# Heart Rate Estimation from Inertial Measurements (SCG / BCG)

**Laboratory of Computational Physics — Final Project**

Università degli Studi di Padova (2026)

<p align="center">
  <img src="example.png" width="55%" alt="Schematic of a supine subject instrumented with wearable inertial sensors">
</p>

## Abstract

Seismocardiography (SCG) and ballistocardiography (BCG) are non-invasive techniques that infer cardiac activity from the micro-vibrations the beating heart transmits to the chest wall (SCG) or to the body's centre of mass and its supports (BCG). In this project we estimate the heart rate (HR) of healthy subjects from linear-acceleration and angular-velocity signals recorded with the **MuSe** inertial measurement unit (IMU) by 221e. Two recordings are analysed: the mandatory **centre-of-sternum** recording and a second recording selected among five alternative sensor placements on the basis of a comparative statistical analysis. The processing pipeline consists of sensor calibration, temporal reconstruction, magnitude computation, exploratory and statistical analysis, principal component analysis (PCA), spectral analysis via the Fast Fourier Transform (FFT), Butterworth band-pass filtering and time-domain peak detection. The dominant cardiac frequency for the centre-of-sternum recording is found at **1.25 Hz (≈ 75 bpm)**, and at **1.38 Hz (≈ 83 bpm)** for the secondary recording, both within the physiological resting range of 40–100 bpm.

---



## Authors

| Member | Main contribution |
|---|---|
| [Gabriela Landinez](https://github.com/gabriela-landinez) | FFT implementation |
| [Andrés Rojas](https://github.com/androjaslo) | Filter implementation; BPM and heart-rate-variability histograms |
| [Polina Gridiaeva](https://github.com/polinagridiaeva) | Statistical analysis and PCA |
| [Melissa Nespeque](https://github.com/melissanespeque) | Exploratory analysis of all datasets |


All members contributed to the bibliographic review, the interpretation of results and the writing of the final notebook.

---

## Repository Structure

```
Computational-Physics-Project/
├── Heart rate estimation  - project.ipynb   # Complete analysis notebook (code, figures, discussion)
├── README.md                                # This document
├── example.png                              # Illustration of the experimental setup (used in the notebook)
├── .gitattributes                           # Git line-ending normalisation
└── Data/
    ├── center_sternum.txt                   # Mandatory recording — sensor on the sternum (subject 1)
    ├── 1_Stave_supine_static.txt            # Sensor on a bed stave, under the mattress (subject 2)
    ├── 2_Mattress_supine.txt                # Sensor on the mattress, near a corner (subject 2)
    ├── 3_Subject_sitting_chair.txt          # Sensor on a desk, subject seated (subject 2)
    ├── 4_Chest_sweater.txt                  # Sensor on the chest, over a sweater (subject 2)
    ├── 5_Under_chair.txt                    # Sensor under the seat of a chair (subject 2)
    ├── Calibration data 1.txt               # Calibration of MuSe_2128 (SCG session, 13-12-2019)
    └── Calibration data 2.txt               # Calibration of MuSe_2168 (BCG session, 16/17-04-2019)
```

The entire analysis is contained in a single Jupyter notebook, organised as follows:

| Section | Content |
|---|---|
| Introduction | Physical background of SCG/BCG and of IMU-based cardiac monitoring |
| Center Sternum (mandatory dataset) | Calibration, data treatment, exploratory and statistical analysis, PCA, FFT, filtering, HR estimation |
| Second dataset selection | Calibration and pre-processing of the five alternative recordings; comparative analysis and choice of the secondary dataset |
| Chest Sweater (secondary dataset) | Statistical analysis, PCA, correlation matrix, FFT, filtering, HR estimation |
| Conclusions · Bibliography · Contributions | Summary of findings, references and division of work |

---

## Data

### Acquisition

All recordings were acquired with the MuSe (Multi-Sensor) platform, a miniaturised, low-power wireless IMU produced by [221e](https://www.221e.com). Two healthy volunteers took part after giving informed consent: the centre-of-sternum recording belongs to subject 1, while all other recordings belong to subject 2.

| File | Sensor placement | Subject | Device | Nominal rate | Samples |
|---|---|---|---|---|---|
| `center_sternum.txt` | Centre of the sternum; subject supine, then on left and right side | 1 | MuSe_2128 | 200 Hz | 16 506 |
| `1_Stave_supine_static.txt` | Bed stave under the mattress, at chest level; subject supine | 2 | MuSe_2168 | 100 Hz | 9 170 |
| `2_Mattress_supine.txt` | On the mattress near a corner, not under the pillow | 2 | MuSe_2168 | 100 Hz | 11 746 |
| `3_Subject_sitting_chair.txt` | On a desk; seated subject resting forearms and hands on it | 2 | MuSe_2168 | 100 Hz | 6 986 |
| `4_Chest_sweater.txt` | On the chest, directly over a sweater | 2 | MuSe_2168 | 100 Hz | 14 938 |
| `5_Under_chair.txt` | Under the seat of a chair; subject seated | 2 | MuSe_2168 | 100 Hz | 9 912 |

### File format

Each recording is a tab-separated text file with a header row and 16 columns:

| Column(s) | Description |
|---|---|
| `Log Mode` | Logging mode of the device |
| `Log Freq` | Nominal acquisition frequency (Hz) |
| `Timestamp` | Unix time (s); multiple samples share the same second |
| `AccX`, `AccY`, `AccZ` | Linear acceleration along each axis (mg) |
| `GyroX`, `GyroY`, `GyroZ` | Angular velocity along each axis (dps) |
| `MagnX`, `MagnY`, `MagnZ` | Magnetic field along each axis |
| `qw`, `qi`, `qj`, `qk` | Orientation quaternion |

### Calibration files

`Calibration data 1.txt` and `Calibration data 2.txt` report the device configuration (full-scale ranges and sampling rate) together with:

- a **gyroscope bias** vector **b**;
- an **accelerometer** 3 × 3 correction matrix **C** and offset vector **o**;
- a magnetometer correction matrix and offset (not used in this work).

---

## Methodology

The same pipeline is applied to both analysed recordings.

1. **Calibration.** Raw readings are corrected as
   **a**<sub>cal</sub> = (**a**<sub>raw</sub> − **o**) **C**<sup>T</sup> for the accelerometer and **g**<sub>cal</sub> = **g**<sub>raw</sub> − **b** for the gyroscope, using the parameters of the corresponding device.
2. **Variable selection.** Only accelerometer and gyroscope channels are retained. Magnetometer data are discarded because of their sensitivity to environmental magnetic perturbations, and quaternions are not required for HR estimation.
3. **Time reconstruction.** Timestamps are verified to be Unix times. Since the clock has a 1 s resolution, a uniform time axis is rebuilt from the mean number of samples per second (≈ 217 Hz effective for the sternum recording, ≈ 99 Hz for the others).
4. **Magnitude signals.** Following Haescher et al. (2015), the axis-independent magnitudes
   |a| = √(a<sub>x</sub>² + a<sub>y</sub>² + a<sub>z</sub>²) and |g| = √(g<sub>x</sub>² + g<sub>y</sub>² + g<sub>z</sub>²)
   are computed and mean-centred, which removes the gravitational component and the residual gyroscope bias.
5. **Trimming.** Transients caused by attaching and removing the sensor are excluded after visual inspection of zoomed views of the beginning and end of each recording.
6. **Statistical description.** Mean, standard deviation, median, minimum, maximum, skewness and kurtosis are computed for each axis.
7. **Principal component analysis.** The eigenvalues of the covariance matrix of the three accelerometer and the three gyroscope axes quantify how the variance is distributed among the axes.
8. **Selection of the secondary dataset.** The five alternative recordings are compared through the time evolution of |a| and |g|, their standard deviations and coefficients of variation. The recording with the lowest acceleration variability that still preserves a meaningful gyroscope signal is retained.
9. **Spectral analysis.** The FFT of the acceleration signal, of the gyroscope signal and of the product of their amplitude spectra is computed. The dominant peak within 0.5–3 Hz (30–180 bpm) is taken as the cardiac frequency, and HR = 60 · f<sub>peak</sub>.
10. **Band-pass filtering.** A second-order Butterworth band-pass filter (0.6–3 Hz), applied with zero-phase forward–backward filtering (`scipy.signal.filtfilt`), suppresses respiratory and high-frequency components; the FFT is recomputed on the filtered signal.
11. **Beat-to-beat analysis.** Peaks of the filtered signal are detected with `scipy.signal.find_peaks` (minimum separation 0.5 s, prominence ≥ 0.3 σ). Inter-peak intervals give the instantaneous HR (60/Δt), whose distribution is shown together with a histogram of the intervals as a qualitative indicator of heart rate variability (HRV).

---

## Results

### Centre-of-sternum recording (subject 1)

**PCA — explained variance**

| Component | Accelerometer | Gyroscope |
|---|---|---|
| PC1 | 74.7 % | 80.3 % |
| PC2 | 14.7 % | 15.3 % |
| PC3 | 10.6 % | 4.5 % |

All three axes of both sensors contribute non-negligible variance and were retained.

**Heart-rate estimates**

| Method | Peak frequency | Heart rate |
|---|---|---|
| FFT of \|a\| | 1.25 Hz | 75.0 bpm |
| FFT of \|g\| | 1.25 Hz | 75.0 bpm |
| FFT of the product \|A(f)\| · \|G(f)\| | 1.25 Hz | 75.0 bpm |
| FFT of band-pass-filtered \|a\| | 1.25 Hz | 75.0 bpm |
| Beat-to-beat (filtered \|a\|) | — | 84.2 ± 19.0 bpm (range 42.7–120.7) |

The three spectral estimators agree, and band-pass filtering yields a sharper peak at the same frequency. Most instantaneous values fall between 70 and 90 bpm.

### Secondary dataset selection

| Dataset | σ(\|a\|) | σ(\|g\|) | Assessment |
|---|---|---|---|
| 1 – Stave, supine | 36.9 | 6.74 | Strong noise; would require discarding much of the record |
| 2 – Mattress, supine | 10.2 | 0.56 | High variability |
| 3 – Sitting, sensor on desk | 10.6 | 0.14 | Over-smoothed; high coefficient of variation |
| 4 – Chest, over sweater | 9.5 | 0.70 | **Lowest acceleration variability, informative gyroscope signal — selected** |
| 5 – Under chair | 9.8 | 0.18 | Spurious peaks; strongly posture-dependent |

### Secondary recording (subject 2)

**PCA — explained variance**

| Component | Accelerometer | Gyroscope |
|---|---|---|
| PC1 | 74.2 % | 86.1 % |
| PC2 | 19.6 % | 8.0 % |
| PC3 | 6.2 % | 5.8 % |

Since the acceleration axes were weakly correlated, the spectral analysis used the X-axis acceleration, which showed correlation with the gyroscope signal.

**Heart-rate estimates**

| Method | Peak frequency | Heart rate |
|---|---|---|
| FFT of a<sub>x</sub> | 0.52 Hz | 31.0 bpm (outside physiological range; rejected) |
| FFT of \|g\| | 1.38 Hz | 82.8 bpm |
| FFT of the product \|A<sub>x</sub>(f)\| · \|G(f)\| | 1.38 Hz | 82.8 bpm |
| FFT of band-pass-filtered \|g\| | 1.38 Hz | 82.8 bpm |

### Summary

| Subject | Recording | Estimated HR |
|---|---|---|
| 1 | Centre of sternum | **75.0 bpm** |
| 2 | Secondary recording | **82.8 bpm** |

---

## Discussion

Both estimates lie within the expected resting range, and the agreement between independent estimators (accelerometer, gyroscope and their spectral product) supports the identification of the dominant peak as cardiac in origin. For the secondary recording the gyroscope proved more reliable than the accelerometer: angular velocity captures the small rotational motion of the chest wall associated with cardiac contraction while being less affected by large linear displacements such as breathing or fabric movement, which dominated the low-frequency (≈ 0.5 Hz) acceleration peak. Band-pass filtering consistently produced cleaner time-domain signals and more pronounced spectral peaks.

The beat-to-beat estimate shows a larger spread than the spectral estimate. This reflects both physiological HRV and the sensitivity of peak detection to residual noise and harmonics, so the spectral value should be regarded as the primary result and the instantaneous distribution as a qualitative description. Further limitations are the small number of subjects, the short duration of the analysed windows (≈ 1 min) and the absence of a reference ECG against which to validate the estimates.

---

## Reproducing the Analysis

**Requirements:** Python ≥ 3.9 with

```
numpy  pandas  scipy  matplotlib  seaborn  jupyter
```

```bash
git clone https://github.com/androjaslo/Computational-Physics-Project.git
cd Computational-Physics-Project
pip install numpy pandas scipy matplotlib seaborn jupyter
jupyter notebook
```

The notebook loads the data files by name (e.g. `pd.read_csv('center_sternum.txt', sep='\t')`), so they must be reachable from the notebook's working directory. Either launch Jupyter from inside `Data/` after moving the notebook there, or prefix the file names in the loading cells with `Data/`.

---

## References

1. Haescher, M., Matthies, D. J., Trimpop, J., & Urban, B. (2015). A study on measuring heart- and respiration-rate via wrist-worn accelerometer-based seismocardiography (SCG) in comparison to commonly applied technologies. *Proceedings of the 2nd International Workshop on Sensor-based Activity Recognition and Interaction*, 1–6.
2. Balali, P., Rabineau, J., Hossein, A., Tordeur, C., Debeir, O., & van de Borne, P. (2022). Investigating cardiorespiratory interaction using ballistocardiography and seismocardiography — a narrative review. *Sensors*, 22(23), 9565.
3. Inan, O. T., Migeotte, P. F., Park, K. S., Etemadi, M., Tavakolian, K., Casanella, R., … & Di Rienzo, M. (2014). Ballistocardiography and seismocardiography: A review of recent advances. *IEEE Journal of Biomedical and Health Informatics*, 19(4), 1414–1427.
4. Belaish, E. (2022). *Know your sensors: An introduction to inertial sensors.* 221e White Paper. [PDF](https://www.221e.com/wp-content/uploads/2022/11/221e-Whitepaper-An-Introduction-to-Inertial-Sensors.pdf)
5. Zigliotto, W., & Belaish, E. (2022). *3 IMU sensor fusion troublemakers and how you can outsmart them.* 221e White Paper. [PDF](https://www.221e.com/wp-content/uploads/2022/11/221e-Whitepaper-3-IMU-Sensor-Fusion-Troublemakers.pdf)
6. PySDR — Filters. <https://pysdr.org/es/content-es/filters.html>
7. Butterworth filters with `scipy.signal`. <https://davrot.github.io/pytutorial/scipy/scipy.signal_butterworth/>
