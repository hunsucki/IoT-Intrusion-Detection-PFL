# 연합학습 기반 IoT 기기 침입 탐지 모델 고도화

## 1. 프로젝트 개요
본 프로젝트는 사물인터넷(IoT) 환경의 보안 위협에 대응하기 위해, 데이터 프라이버시를 보호하면서도 효과적인 침입 탐지가 가능한 '연합학습(Federated Learning)' 모델을 연구 및 구현한 결과물임.
선행 연구(Rahman et al., 2020)를 분석 및 재현하여 연합학습의 유효성을 검증하고, 나아가 최신 IoT 데이터셋(CICIoT2023) 환경에서 발생하는 성능 저하 문제를 해결하기 위해 '개인화 연합학습(PFL)' 기법을 도입하여 탐지 성능을 고도화하였음.

## 2. 선행 연구 분석 (Baseline Study)
* **논문명**: *Internet of Things Intrusion Detection: Centralized, On-Device, or Federated Learning?* (IEEE Network, 2020)
* **핵심 내용**:
    * 기존 중앙 집중식(Centralized) 방식은 단일 실패 지점(Single Point of Failure) 및 데이터 프라이버시 침해 위험이 존재함.
    * 온디바이스(On-Device) 학습은 프라이버시는 보호되나, 기기 간 지식 공유 부재로 미지 탐지(Unknown Attack) 성능이 낮음.
    * 본 논문은 데이터를 로컬에 유지하면서 모델 파라미터만 공유하는 **연합학습(Federated Learning)**을 제안하며, 이를 통해 프라이버시 보호와 탐지 정확도를 동시에 확보할 수 있음을 주장함.

## 3. 연구 수행 내용

본 프로젝트는 총 3단계(검증-확장-개선)로 진행되었음.

### 3.1. [1단계] 베이스라인 모델 재현
논문의 실험 환경을 동일하게 구현하여 연합학습의 유효성을 검증함.
* **데이터셋**: NSL-KDD (Legacy Dataset)
* **실험 환경**: Use Case #1 (Non-IID). 4개의 노드가 각각 서로 다른 단일 공격 유형(DoS, Probe, R2L, U2R)만을 학습하는 환경 설정.
* **검증 결과**:
    * 개별 학습(Self-Learning) 시 학습하지 않은 공격 유형을 탐지하지 못하는 한계 확인.
    * 연합학습(FL) 적용 시, 타 노드의 지식을 공유받아 미지의 공격 탐지율이 상승하며 중앙 집중식 모델에 근접한 성능을 보임.

### 3.2. [2단계] 최신 데이터셋 적용과 한계
1999년 데이터인 NSL-KDD의 한계를 극복하기 위해, 2023년 공개된 최신 대규모 데이터셋을 적용함.
* **데이터셋**: CICIoT2023 (33종의 최신 공격 시나리오 포함).
* **문제점 발견 ('평균의 함정')**:
    * 데이터 분포가 극단적으로 이질적인 최신 환경(Non-IID)에서는 일반적인 연합학습(FedAvg) 모델이 특정 노드의 성능을 오히려 저하시키는 현상 발생.
    * 예시: Mirai 봇넷 공격을 학습한 노드의 경우, 글로벌 모델 적용 시 정확도가 55%로 급락함.

### 3.3. [3단계] 개인화 연합학습(PFL) 적용
'평균의 함정'을 해결하고 탐지 성능을 극대화하기 위해 모델 고도화를 수행함.
* **제안 기법**: 전체 레이어 미세 조정(Full Layer Fine-tuning) 기반의 PFL.
* **알고리즘**:
    1. 서버에서 집계된 글로벌 모델(Global Model)을 각 로컬 노드로 전송.
    2. 각 노드는 수신한 모델을 자신의 로컬 데이터로 추가 재학습(Fine-tuning, 10 Epochs) 수행.
* **최종 결과**:
    * 기존 FedAvg 대비 모든 노드에서 탐지 정확도가 비약적으로 상승.
    * 중앙 집중식 모델(99.36%)에 근접한 **97~98%대의 정확도**를 달성하여 데이터 이질성 문제를 해결함.

## 4. 폴더 구조 및 파일 설명

본 레포지토리는 연구 단계별 소스코드를 포함하고 있음.

```text
IoT-Intrusion-Detection-PFL/
├── Code/                            # 소스 코드 폴더
│   ├── 1-baseline-NSL-KDD.ipynb
│   ├── 2-ciciot-2023-fl.ipynb
│   ├── 2-1-ciciot-2023-fl.ipynb
│   └── 3-PFL-entire-finetuning.ipynb
├── IoT-Intrusion-Detection-PFL.pdf       # 발표 자료
└── README.md                             # README
```

## 5. 실행 매뉴얼

### 5.1. Kaggle 환경 실행 (권장)
본 코드는 데이터셋 다운로드 및 라이브러리 설치 편의성을 위해 Kaggle 환경 실행을 권장함.

1.  **Notebook 생성**
    * Kaggle 접속 및 로그인 후 'Create New Notebook' 선택.
2.  **데이터셋 추가**
    * Notebook 우측 'Add Input' 메뉴에서 아래 두 데이터셋을 검색하여 추가.
        * **NSL-KDD**: `hassan06/nslkdd`
        * **CICIoT2023**: `himadri07/ciciot2023`
3.  **코드 업로드**
    * `File` > `Import Notebook` 기능을 사용하여 `1-baseline-NSL-KDD.ipynb` 또는 `3-PFL-entire-finetuning.ipynb` 파일 업로드.
4.  **라이브러리 설치**
    * Notebook 첫 번째 셀에 아래 명령어 입력 및 실행.
    ```python
    !pip install flwr
    ```
5.  **실행**
    * 상단 메뉴의 `Run All`을 클릭하여 전체 실험 진행 및 결과 그래프 확인.

### 5.2. 로컬 환경 실행
로컬 환경에서 실행할 경우 데이터셋 경로 설정 수정이 필요함.

1.  **환경 설정**
    ```bash
    pip install flwr tensorflow pandas scikit-learn numpy matplotlib
    ```
2.  **데이터셋 준비**
    * 상기 Kaggle 링크에서 데이터셋 다운로드 후, 소스코드와 동일한 경로의 `data/` 폴더에 압축 해제.
    * 소스코드 내 데이터 로드 경로(`pd.read_csv`)를 로컬 경로에 맞게 수정.
3.  **실행**
    * Jupyter Notebook 실행 후 각 셀을 순차적으로 수행.

## 6. 참고 문헌 및 데이터셋
* **논문**: S. A. Rahman, H. Tout, C. Talhi and A. Mourad, "Internet of Things Intrusion Detection: Centralized, On-Device, or Federated Learning?," *IEEE Network*, vol. 34, no. 6, pp. 310-317, 2020. ([IEEE Xplore Link](https://ieeexplore.ieee.org/document/9183799))
* **NSL-KDD Dataset**: [Kaggle Link](https://www.kaggle.com/datasets/hassan06/nslkdd)
* **CICIoT2023 Dataset**: [Kaggle Link](https://www.kaggle.com/datasets/himadri07/ciciot2023)
