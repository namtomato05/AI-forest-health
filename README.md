# 🌿 AI Forest Health Monitor
### NDVI 없이 기상·지형 데이터만으로 한반도 식생 건강도를 예측하는 멀티모달 딥러닝 시스템

---

## 📌 프로젝트 개요

VCI(Vegetation Condition Index)는 원래 **NDVI(다중분광 센서)** 로 계산되는 식생 건강도 지표입니다.  
이 프로젝트는 다음 질문에서 출발했습니다:

> **"고가의 다중분광 센서 없이, 강수량·지표면온도·고도만으로 식생 건강도를 추론할 수 있을까?"**

Sentinel-2 위성 이미지 패치 + 기상/지형 수치 데이터를 결합한 **멀티모달 CNN** 으로 VCI를 직접 회귀 예측합니다.

---

## 🗺️ 분석 대상

| 항목 | 내용 |
|------|------|
| 지역 | 한반도 전체 (124.5~131°E, 33~39.5°N) |
| 기간 | 2017년 ~ 2025년 (3월~11월) |
| 샘플 | 기간당 500 포인트 × 81개 기간 |
| 데이터 소스 | Google Earth Engine (Sentinel-2, CHIRPS, MODIS, SRTM) |

---

## 🏗️ 모델 아키텍처

```
위성 이미지 패치 (7채널, 64×64)
        ↓
    LightCNN
  (Conv2d × 3 + BN + Pool)
        ↓ 128차원
        ├──────────────────────┐
수치 피처 (Rain, LST, Elev)    │
        ↓                     │
      MLP (3→64)               │
        ↓ 64차원               │
        └──────────────────────┘
                ↓
          Fusion Head
        (192→128→64→1)
                ↓
          Sigmoid 출력
        VCI 예측값 (0~1)
```

---

## 📁 파일 구조

```
AI_FOREST_HEALTH/
├── data.ipynb        # GEE 데이터 탐색 & 인터랙티브 시각화
├── pipeline.ipynb    # 학습 데이터 구축 (CSV + 위성 패치 수집)
├── model.ipynb       # 모델 학습 & 평가
├── map.ipynb         # 전국 예측 지도 생성
├── data/
│   ├── csv/          # 수치 피처 + VCI 라벨 CSV
│   └── patches/      # Sentinel-2 64×64 패치 (.npy)
├── models/
│   ├── best_model_regression.pth
│   └── norm_params.npz
└── maps/
    ├── veg_health_map_vci.png
    ├── veg_health_map_class.png
    └── heatmap_VCI_*.html
```

---

## 🔬 VCI 라벨 기준 (Kogan, 1995)

| 등급 | VCI 범위 | 의미 |
|------|----------|------|
| 🟢 매우건강 | ≥ 0.75 | 식생 상태 매우 양호 |
| 🟩 건강 | 0.55 ~ 0.75 | 식생 상태 양호 |
| 🟨 보통 | 0.40 ~ 0.55 | 주의 필요 |
| 🟠 주의 | 0.25 ~ 0.40 | 식생 스트레스 징후 |
| 🔴 위험 | < 0.25 | 심각한 식생 저하 |

---

## ⚙️ 학습 설정

| 항목 | 값 |
|------|----|
| Loss | MSELoss |
| Optimizer | AdamW (lr=3e-4) |
| Scheduler | CosineAnnealingLR |
| Epochs | 최대 60 (Early stopping) |
| Batch size | 64 |
| 평가지표 | MAE / RMSE / R² |

---

## 🚀 실행 순서

```bash
# 1. 데이터 수집 및 패치 다운로드
jupyter notebook pipeline.ipynb

# 2. 모델 학습
jupyter notebook model.ipynb

# 3. 전국 예측 지도 생성
jupyter notebook map.ipynb
```

### 사전 요구사항
```bash
pip install earthengine-api geemap torch numpy pandas scikit-learn matplotlib folium
```
Google Earth Engine 계정 및 프로젝트 설정 필요 (`ee.Initialize(project='YOUR_PROJECT')`)

---

## 💡 핵심 기여

- **NDVI 제거**: VCI 계산의 핵심 재료인 NDVI를 입력에서 제외 → 고가 센서 없이도 예측 가능성 검증
- **멀티모달 융합**: CNN(이미지) + MLP(수치) 결합으로 공간·기상 정보 동시 활용
- **공식 VCI 라벨**: Kogan(1995) 공식 기반 5년 과거 NDVI min/max로 신뢰성 있는 라벨 생성
- **인터랙티브 지도**: folium 기반 기간별 예측 결과 시각화

---

## 🛠️ 기술 스택

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat&logo=pytorch&logoColor=white)
![Google Earth Engine](https://img.shields.io/badge/Google%20Earth%20Engine-4285F4?style=flat&logo=google&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)

---

## 📄 참고문헌

- Kogan, F.N. (1995). *Application of vegetation index and brightness temperature for drought detection*. Advances in Space Research.
- Copernicus Sentinel-2 Mission, ESA
- CHIRPS: Funk et al. (2015), Scientific Data
