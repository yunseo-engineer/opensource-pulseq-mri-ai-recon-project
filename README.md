# Open-source Pulseq Programming with Self-supervised AI Image Reconstruction

> 본 프로젝트는 보스턴(Boston)에서 진행된 **Global Biohealth Engineering Hackathon**의 일환으로,  
> **Massachusetts General Hospital (MGH)의 Athinoula A. Martinos Center for Biomedical Imaging**에서  
> Harvard–MIT 연구 환경과 연계하여 수행되었습니다.

## 프로젝트 소개

MRI에서는 더 많은 k-space 데이터를 획득할수록 높은 품질의 영상을 얻을 수 있지만,
그만큼 촬영 시간이 길어지고 환자의 움직임에 의한 artifact가 발생할 가능성도 증가합니다.

본 프로젝트의 목표는 **MRI acquisition 과정에서 획득하는 k-space 데이터를 줄여 촬영 시간을 단축하면서도,
영상 복원 기술을 통해 충분한 영상 품질을 유지할 수 있는 방법을 탐색하는 것**이었습니다.

이를 위해 MRI 촬영 단계부터 영상 reconstruction 및 성능 평가까지의 전체 과정을 직접 구성하였습니다.

---

## 1. Open-source Pulseq를 이용한 MRI Sequence 설계

**Pulseq**는 특정 MRI 장비 제조사에 종속되지 않고 MRI pulse sequence를 설계할 수 있도록 개발된
**open-source, vendor-independent MRI sequence programming framework**입니다.

RF pulse, gradient, ADC 등의 MRI acquisition event를 코드로 정의하여 `.seq` 형식의 sequence 파일을 생성할 수 있으며,
이를 이용해 다양한 acquisition protocol을 비교적 자유롭게 설계하고 실험할 수 있습니다.

본 프로젝트에서는 Pulseq를 이용하여 동일한 기본 acquisition parameter를 유지하면서
**k-space sampling 방식과 acceleration factor가 서로 다른 9개의 GRE sequence**를 설계하였습니다.

### Acceleration Factor

- R = 1 : Fully sampled reference
- R = 2
- R = 4
- R = 6
- R = 8

### Sampling Pattern

- Uniform Cartesian undersampling
- Random Cartesian undersampling

R = 1을 제외한 각 acceleration factor에 대해
Random / Uniform Cartesian sampling sequence를 각각 설계하였습니다.

생성된 Pulseq sequence (`.seq`) 파일은 `seq/` 폴더에서 확인할 수 있습니다.

---

## 2. MRI Data Acquisition

설계한 Pulseq sequence를 실제 MRI scanner에서 실행하여
**multi-coil raw k-space data**를 획득하였습니다.

데이터는 scanner raw data인 `.dat` 형식으로 저장되었으며,
fully sampled acquisition을 기준 영상(reference)으로 사용하였습니다.

Acceleration factor가 증가할수록 획득하는 phase-encoding line의 수가 감소하므로
MRI scan time 역시 감소하게 됩니다.

예를 들어 본 실험에서는:

| Acceleration | Acquired ky lines | Scan Time |
|---|---:|---:|
| R = 1 | 256 | 7.12 s |
| R = 2 | 128 | 4.56 s |
| R = 4 | 64 | 3.28 s |
| R = 6 | 43 | 2.86 s |
| R = 8 | 32 | 2.64 s |

와 같이 acceleration factor에 따른 scan-time reduction을 확인하였습니다.

원본 `.dat` 파일은 파일 용량 및 데이터 공유 관련 사유로 본 repository에는 포함하지 않았습니다.

---

## 3. MRI Reconstruction

Undersampling을 통해 촬영 시간을 줄이면 k-space 정보가 부족해지기 때문에,
단순 inverse Fourier transform만으로는 aliasing artifact가 발생하게 됩니다.

따라서 본 프로젝트에서는 동일한 undersampled k-space 데이터를 여러 reconstruction 방법으로 복원하고
각 방법의 성능을 비교하였습니다.

### Zero-filled Reconstruction

획득하지 않은 k-space 영역을 0으로 채운 후 inverse Fourier transform을 수행하는 가장 기본적인 reconstruction 방법입니다.

### SENSE

Multi-coil MRI에서 각 receiver coil의 spatial sensitivity 차이를 이용하여
undersampling으로 발생한 aliasing을 제거하는 parallel imaging reconstruction 방법입니다.

### L1 Wavelet Reconstruction

MRI 영상이 wavelet domain에서 sparse하다는 특성을 이용하여
L1 regularization 기반의 compressed sensing reconstruction을 수행하였습니다.

### Self-supervised AI Reconstruction

Fully sampled ground truth 없이도 학습할 수 있도록
획득된 undersampled k-space를 training 영역과 validation 영역으로 나누어 학습하는
**self-supervised reconstruction** 방법을 적용하였습니다.

모델은 acquired k-space 내부의 일부 데이터를 학습 target으로 활용하여
missing k-space 정보를 복원하도록 학습되었습니다.

---

## 4. Reconstruction Performance Evaluation

각 reconstruction 결과는 fully sampled reference image를 기준으로 비교하였습니다.

정량적 평가지표로 다음 세 가지 metric을 사용하였습니다.

- **NMSE (Normalized Mean Squared Error)**
- **PSNR (Peak Signal-to-Noise Ratio)**
- **SSIM (Structural Similarity Index Measure)**

또한 reconstruction image와 residual/error image를 함께 비교하여
각 acceleration factor와 sampling pattern에서 발생하는 artifact를 시각적으로 분석하였습니다.

비교 대상은 다음과 같습니다.

- Acceleration factor: R = 2, 4, 6, 8
- Sampling pattern: Random / Uniform Cartesian
- Reconstruction: Zero-filled / SENSE / L1 Wavelet / Self-supervised AI

---

## 전체 프로젝트 Workflow

```text
Pulseq Sequence Design
        ↓
R = 1 / 2 / 4 / 6 / 8
Random & Uniform Cartesian Sampling
        ↓
MRI Scanner Acquisition
        ↓
Multi-coil Raw k-space (.dat)
        ↓
Preprocessing
        ↓
Reconstruction
 ├─ Zero-filled
 ├─ SENSE
 ├─ L1 Wavelet
 └─ Self-supervised AI
        ↓
Image Quality Evaluation
 ├─ NMSE
 ├─ PSNR
 ├─ SSIM
 └─ Error Map
        ↓
Scan Time vs. Image Quality Analysis
