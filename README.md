# 의료 접근성이 건강 결과에 미치는 영향 분석

BRFSS 2015 당뇨병 건강지표 데이터셋(25만 행)을 이용해 **의료 접근성(보험 유무, 비용 부담으로 인한 병원 미방문, 정기검진 여부)이 만성질환·건강 결과와 어떤 관계가 있는지** 분석한 프로젝트입니다. EDA, 교차분석, 로지스틱 회귀(오즈비 추정), 모델 가정 점검, 예측 모델링까지 전체 데이터 분석 파이프라인을 다룹니다.

## 문제 정의

- **발견하고 싶은 문제**: 의료 접근성이 낮은 집단(비용 부담으로 병원을 못 간 경험, 보험 미보유)이 실제로 만성질환 관리와 건강 결과에서 더 취약한가? 소득·연령 같은 교란요인을 통제해도 접근성 자체의 독립적 영향이 남는가?
- **이유**: 의료 접근성 격차가 건강 결과에 실질적 영향을 미치는지 데이터로 검증하면, 어떤 소득·연령대에 접근성 개선 정책이 우선적으로 필요한지 근거를 제시할 수 있습니다.

## 데이터

- **파일**: `data/diabetes_binary_health_indicators_BRFSS2015.csv`
- **출처**: CDC BRFSS(Behavioral Risk Factor Surveillance System) 2015 설문조사를 기반으로 가공된 공개 파생 데이터셋(Diabetes Health Indicators Dataset)
- **규모**: 253,680행 × 22열, 결측치 없음, 전부 이진/순서형 코드(float64)
- **타깃**: `Diabetes_binary` (당뇨 진단 여부, 양성 비율 13.9%)
- 용량 문제로 저장소에는 포함하지 않았습니다(`.gitignore`에 `data/` 등록). 분석을 재현하려면 CSV를 위 경로에 직접 위치시켜야 합니다.

## 실행 방법

```bash
# 의존성 설치 (uv 사용)
uv sync

# data/diabetes_binary_health_indicators_BRFSS2015.csv 준비 후

# 노트북 실행 (Jupyter)
uv run jupyter lab notebooks/healthcare_access_analysis.ipynb

# 또는 커맨드라인에서 전체 재실행
uv run jupyter nbconvert --to notebook --execute --inplace notebooks/healthcare_access_analysis.ipynb
```

## 분석 파이프라인

`notebooks/healthcare_access_analysis.ipynb`

1. **데이터 전처리** — 결측치/중복행/값 범위 점검, 중복행 처리 방침 명시
2. **EDA** — 접근성 변수(`AnyHealthcare`, `NoDocbcCost`, `CholCheck`) 분포
3. **교차분석 & 카이제곱검정** — 접근성 변수별 건강 결과 비율 차이
4. **소득 층화분석** — 소득 구간별 접근성·결과지표 변화, 스피어만 상관
5. **로지스틱 회귀(오즈비)** — 비보정 vs 보정(소득·연령·성별·교육·BMI 통제) 모델 비교
   - 5.1 모델 가정 점검 — 다중공선성(VIF), 선형성(Box-Tidwell 검정)
   - 보조 결과지표(`PoorGenHlth`, 주관적 건강상태) 모델
6. **예측 모델링 및 성능 평가** — 접근성 변수만 사용한 모델 vs 전체 21개 변수 모델 비교 (Train/Test split, ROC-AUC, 분류 리포트, 혼동행렬)
7. **종합 결론 및 한계**

## 핵심 결과

- `NoDocbcCost`(비용 부담으로 병원 미방문 경험)는 소득·연령 등을 통제한 뒤에도 당뇨(OR 1.37)와 주관적 건강불량(OR 2.46)에 독립적으로 유의한 영향을 미칩니다.
- `CholCheck`, `AnyHealthcare`의 큰 오즈비(각 5.10배, 1.23배)는 인과효과보다 **검출편향**(검진·의료 이용이 있어야 진단이 기록되는 구조)으로 해석해야 합니다.
- 소득 최하위군은 최상위군 대비 비용장벽 7배, 주관적 건강불량 7.7배, 당뇨 유병률 3배 높은 뚜렷한 사회경제적 구배를 보입니다.
- 접근성 변수 3개만으로 만든 예측 모델은 ROC-AUC 0.532(사실상 무작위 수준)에 그치지만, 전체 21개 변수를 쓴 모델은 0.822까지 올라갑니다 — **통계적으로 유의한 연관과 예측력은 별개**임을 보여줍니다.

## 한계

- `Diabetes_binary`는 실제 유병 여부가 아니라 "진단받은 적 있는지"를 묻는 자가보고 변수라 미진단 편향 가능성이 있습니다.
- 단면(cross-sectional) 데이터이므로 오즈비는 연관성이지 인과관계를 증명하지 않습니다.
- 전체 표본의 9.5%가 완전 중복행이지만, 구간화된 범주형 변수 특성상 데이터 오류로 보기 어려워 삭제하지 않고 유지했습니다.

## 기술 스택

Python, pandas, NumPy, scikit-learn, statsmodels, matplotlib, seaborn, Jupyter — [uv](https://docs.astral.sh/uv/)로 의존성 관리
