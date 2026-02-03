# Amazon 제품 성공 점수 예측 (AutoGluon 활용)

## 프로젝트 개요
이 프로젝트는 Amazon 제품 데이터를 분석하여 제품의 '성공 점수(Success Score)'를 예측하는 머신러닝 모델을 구축하는 것을 목표로 합니다. 데이터 전처리, 파생 변수 생성(Feature Engineering), 그리고 **AutoGluon** 라이브러리를 활용한 회귀 모델링의 전 과정을 포함하고 있습니다.

## 데이터셋 정보
*   **파일**: `amazon.csv`
*   **주요 데이터**: 제품명, 카테고리, 할인된 가격, 실제 가격, 할인율, 평점, 리뷰 수 등.

## 데이터 전처리 (Data Preprocessing)
Raw Data를 모델 학습 이전에 분석 가능한 형태로 정제하였습니다.
1.  **통화 및 기호 제거**: 가격 컬럼의 `₹`, `,` 기호와 할인율의 `%` 기호를 제거하고 수치형(float)으로 변환했습니다.
2.  **카테고리 파싱**: 복잡한 `category` 문자열을 분석하여 대분류(`main_category`)와 소분류(`sub_category`)로 분리했습니다.
3.  **브랜드명 정제**: 중복되거나 불일치하는 브랜드 이름을 통일(Normalization)하고, 결측치를 처리했습니다.
4.  **결측치 처리**: `rating`, `rating_count` 등의 주요 컬럼 내 결측치를 적절한 값으로 대체하거나 제거하여 데이터 무결성을 확보했습니다.

## 피처 엔지니어링 (Feature Engineering)
모델의 예측 성능을 높이기 위해 도메인 지식을 활용하여 새로운 파생 변수를 생성했습니다.
*   **Success Score (타겟 변수)**: 제품의 평점과 리뷰 수를 결합하여 종합적인 성공 지표를 정의했습니다.
    *   공식: `Success Score = Rating * log10(Rating Count + 1)`
    *   리뷰 수가 많을수록 평점의 신뢰도가 높다는 점을 반영하여 로그 변환을 적용했습니다.
*   **Brand Explicit**: 주요 IT/전자 브랜드 리스트를 기반으로, 해당 제품이 유명 브랜드(`Branded`)인지 일반 제품(`Non-Branded`)인지 구분하는 이진 변수를 생성했습니다.

## 모델링 (Modeling: AutoGluon)
AutoML 라이브러리인 **AutoGluon**의 `TabularPredictor`를 사용하여 회귀 예측 모델을 학습했습니다.

*   **Task Type**: Regression (회귀)
*   **Target Variable**: `success_score`
*   **Evaluation Metric**: RMSE (Root Mean Squared Error)
*   **학습 전략**:
    *   **Data Split**: 학습용(Train) 80%, 테스트용(Test) 20%
    *   **Presets**: `best_quality` (최고의 예측 성능을 달성하기 위해 앙상블 및 스택 모델링 등 고도화된 전략 자동 적용)
*   **사용된 피처**:
    *   `main_category`, `sub_category`, `brand_final` (범주형)
    *   `discounted_price`, `actual_price`, `discount_percentage`, `rating`, `rating_count` (수치형)
    *   `is_brand_explicit` (이진형)

## 분석 결과 및 인사이트
*   **Feature Importance**: 모델 분석 결과, 제품의 성공 점수에 가장 큰 영향을 미치는 요인은 **서브 카테고리(`sub_category`)**와 **브랜드(`brand_final`)**로 나타났습니다.
*   **브랜드 영향력**: 데이터 시각화를 통해, 유명 브랜드 제품이 비브랜드 제품 대비 전반적으로 더 높은 성공 점수를 기록함을 확인했습니다.
*   **모델 성능**: Residual Analysis(잔차 분석) 결과, 예측 오차가 0을 중심으로 정규 분포에 가까운 형태를 보여 모델이 편향 없이 안정적으로 예측하고 있음을 검증했습니다.
