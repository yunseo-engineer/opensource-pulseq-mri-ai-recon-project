# 데이터 안내

본 프로젝트에서는 Pulseq를 이용하여 획득한 **multi-coil MRI raw k-space data**를 사용하였습니다.

원본 MRI 데이터는 `.dat` 형식이며, 파일 용량 및 데이터 공유 관련 사유로 본 저장소에는 포함하지 않았습니다.

### 데이터 구성

- Raw data: `.dat`
- Multi-coil MRI k-space data
- Random / Uniform Cartesian undersampling
- Acceleration factor: R = 1, 2, 4, 6, 8

MRI acquisition에 사용된 Pulseq sequence (`.seq`) 파일은 `seq/` 폴더에서 확인할 수 있습니다.
