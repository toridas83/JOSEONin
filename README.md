# JOSEONin: YOLO 기반 선박 용접 육안 검사 솔루션

[cite_start]**JOSEONin**은 선박 건조 현장의 용접 품질 관리를 자동화하기 위해 개발된 **AI 기반 비전 검사 솔루션**입니다. [cite: 1, 2, 3]

[cite_start]YOLO(You Only Look Once) 객체 탐지 모델을 활용하여 용접 부위를 실시간으로 분석하고, **양품, 불량, 블루홀** 등 3가지 상태로 정밀 분류하여 육안 검사의 한계를 극복하고 안전성을 높이는 것을 목표로 합니다. [cite: 1, 76, 77, 128]

## 📋 목차 (Table of Contents)
1. [프로젝트 개요 (Overview)](#프로젝트-개요-overview)
2. [주요 기능 (Key Features)](#주요-기능-key-features)
3. [기술 스택 (Tech Stack)](#기술-스택-tech-stack)
4. [AI 모델 및 학습 정보 (AI Model & Training)](#ai-모델-및-학습-정보-ai-model--training)
5. [실행 방법 (Usage)](#실행-방법-usage)

---

## 🚩 프로젝트 개요 (Overview)
### 배경 및 문제 인식
* [cite_start]**숙련 인력 부족:** 조선업계의 숙련 용접 인력 고령화 및 청년층 유입 저조로 인한 인력난이 심화되고 있습니다. [cite: 32, 33]
* [cite_start]**안전 및 품질 이슈:** LNG 운반선 등에서 발생하는 미세 용접 결함은 누설 및 폭발 사고로 직결될 수 있으나, 기존 육안 검사로는 미세 균열 탐지에 한계가 존재합니다. [cite: 36, 37, 38]

### 솔루션 목표
* [cite_start]**자동화 검사:** 카메라와 AI를 활용하여 용접 부위를 자동으로 검수하고 데이터를 수집합니다. [cite: 76, 87]
* [cite_start]**효율성 증대:** 검사 시간을 단축하고 재작업 비용을 절감하여 조선 공정의 생산성을 향상시킵니다. [cite: 151, 153]

---

## 🔍 주요 기능 (Key Features)
* [cite_start]**실시간 결함 탐지:** 카메라로 촬영된 영상을 분석하여 용접 부위의 상태를 즉각적으로 판독합니다. [cite: 76, 95]
* [cite_start]**3단계 품질 분류:** 용접 상태를 아래 3가지 유형으로 식별합니다. [cite: 77]
    * [cite_start]**Good (양품):** 정상적인 용접 상태 [cite: 80]
    * [cite_start]**Bad (불량):** 용접 불량 상태 [cite: 78]
    * [cite_start]**Bluehole (블루홀):** 기공 등 특정 결함(블루홀)이 발생한 상태 [cite: 79]
* [cite_start]**데이터 파이프라인 최적화:** 현장 데이터를 수집하고 학습 가능한 형태로 변환하는 자동화 파이프라인을 구축했습니다. [cite: 87]

---

## 🛠 기술 스택 (Tech Stack)
* **Language:** Python
* **Computer Vision:** Ultralytics YOLOv8
* **Development Environment:** Google Colab, Google Drive
* **Libraries:** `ultralytics`, `PIL` (Image Processing), `shutil`, `glob`, `json`

---

## 🧠 AI 모델 및 학습 정보 (AI Model & Training)
본 프로젝트는 **YOLOv8 Nano (`yolov8n.pt`)** 모델을 베이스로 하여 용접 데이터셋에 대한 전이 학습(Transfer Learning)을 수행했습니다.

### 1. 클래스 정의 (Class Definition)
학습 데이터셋은 다음 3가지 클래스로 매핑되어 학습되었습니다.
* `0`: **good_weld** (양품, ID: 1101)
* `1`: **bad_weld** (불량, ID: 1102)
* `2`: **bluehole_weld** (블루홀, ID: 1202)

### 2. 데이터 전처리 (Preprocessing)
* **Format Conversion:** COCO 포맷(JSON)으로 제공된 어노테이션 데이터를 YOLO 학습 포맷(`.txt`)으로 변환하는 스크립트를 적용했습니다.
* **Coordinate Normalization:** `x_min, y_min, width, height` 좌표를 YOLO 포맷인 `x_center, y_center, width, height` (0~1 정규화)로 변환했습니다.

### 3. 학습 설정 (Training Config)
* **Epochs:** 10
* **Image Size:** 640
* **Device:** GPU (Colab Environment)
* **Train/Val Split:** 학습 및 검증 데이터셋을 분리하여 과적합(Overfitting) 방지

---

## 🚀 실행 방법 (Usage)

### 1. 필수 라이브러리 설치
```bash
pip install ultralytics
```

### 2. 데이터셋 준비 및 변환
JSON 라벨 데이터를 YOLO 형식으로 변환합니다. (제공된 스크립트 활용)
```Python
# 경로 설정 후 변환 함수 실행
convert_data(train_good_img_path, train_good_lbl_path, output_img_dir, output_lbl_dir)
```

### 3. 모델 학습 (Training)
```Python
from ultralytics import YOLO

# 모델 로드 (YOLOv8 Nano)
model = YOLO("yolov8n.pt")

# 학습 시작
model.train(
    data='weld_multiclass.yaml', 
    epochs=10, 
    imgsz=640, 
    project='/content/drive/MyDrive/weld_project', 
    name='train_result'
)
```

### 4. 추론 (Inference)
학습된 모델(best.pt)을 사용하여 새로운 이미지의 용접 상태를 판별합니다.
```Bash
yolo task=detect mode=predict model=best.pt source='test_image.jpg' save=True
```
