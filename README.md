# 의료 접근성이 건강 결과에 미치는 영향 분석

BRFSS 2015 당뇨병 건강지표 데이터셋(25만 행)으로 **의료 접근성(보험 유무, 비용 부담 병원 미방문, 정기검진 여부)이 건강 결과와 어떤 관계가 있는지** 분석한 프로젝트입니다.

## 문제 정의

- **문제**: 의료 접근성이 낮은 집단이 건강 결과에서도 더 취약한가? 소득·연령을 통제해도 그 영향이 남는가?
- **이유**: 검증되면 접근성 개선 정책이 우선 필요한 대상을 데이터로 뒷받침할 수 있습니다.

## 프로젝트 구조

```
project1/
├── data/
│   ├── raw/          # 원본 데이터 (수정 금지)
│   ├── interim/       # 전처리 중간 산출물 (비어 있음)
│   └── processed/     # 분석용 데이터 (비어 있음)
├── notebooks/         # 1.0-healthcare-access-analysis.ipynb
├── models/            # 학습한 모델 파일 (비어 있음)
├── reports/
│   ├── healthcare_access_analysis_report.html  # 노트북 HTML 리포트
│   ├── dashboard.html  # 핵심 수치 요약 대시보드
│   └── figures/       # 그래프 PNG
├── references/
│   └── data_dictionary.md  # 컬럼 사전
├── requirements.txt    # pip 의존성 목록
├── pyproject.toml / uv.lock  # uv 의존성 관리
└── README.md
```

## 데이터

- **파일**: `data/raw/diabetes_binary_health_indicators_BRFSS2015.csv` ([컬럼 사전](references/data_dictionary.md))
- **출처**: CDC BRFSS(Behavioral Risk Factor Surveillance System) 2015 설문 기반 공개 파생 데이터셋(Diabetes Health Indicators Dataset)
- **규모**: 253,680행 × 22열, 결측치 없음. 타깃은 `Diabetes_binary`(당뇨 진단 여부, 양성 비율 13.9%)
- 용량 문제로 저장소에는 포함하지 않았습니다. 재현하려면 CSV를 위 경로에 직접 위치시켜야 합니다.

## 실행 방법

**요구 사항**: Python 3.14 이상 ([uv](https://docs.astral.sh/uv/) 사용 시 `uv sync`가 자동 설치)

```bash
uv sync
# data/raw/diabetes_binary_health_indicators_BRFSS2015.csv 준비 후
uv run jupyter lab notebooks/1.0-healthcare-access-analysis.ipynb
```

## 분석 파이프라인

`notebooks/1.0-healthcare-access-analysis.ipynb`

1. **데이터 이해** — 데이터를 고른 이유, 행/컬럼 의미, 출처
2. **데이터 전처리** — 결측치/중복행/이상치 점검
3. **EDA** — 타깃 분포, 접근성 변수 분포
4. **교차분석 & 카이제곱검정** — 접근성 변수별 건강 결과 비율 차이
5. **교란변수 선정** — 상관관계 스크리닝 + 매개변수 여부 논리적 재검토
6. **소득 심층 층화분석** — 소득 구간별 접근성·결과지표 변화
7. **로지스틱 회귀(오즈비)** — 비보정 vs 보정 모델 비교, 모델 가정 점검(VIF·선형성), 민감도 분석, 보조 결과지표
8. **예측 모델링 및 성능 평가** — 기준선 vs 접근성만 사용한 모델 vs 전체 변수 모델 비교, 변수 중요도, 트리 기반 모델과의 비교
9. **합성데이터 검증** — 정답을 아는 가상 데이터로 교란변수 통제·검출편향 해석의 타당성 시뮬레이션
10. **종합 결론, 한계, 비즈니스 인사이트**

## 핵심 결과

- `NoDocbcCost`(비용 부담으로 병원 미방문 경험)는 소득·연령 등을 통제한 뒤에도 당뇨(OR 1.37)와 주관적 건강불량(OR 2.46)에 독립적으로 유의한 영향을 미칩니다.
- `CholCheck`(OR 5.10), `AnyHealthcare`(OR 1.23)의 큰 오즈비는 인과효과가 아니라 **검출편향**(검진·의료 이용이 있어야 진단이 기록되는 구조)으로 해석해야 합니다.
- 소득 최하위군은 최상위군 대비 비용장벽 7배, 주관적 건강불량 7.7배, 당뇨 유병률 3배 높은 뚜렷한 사회경제적 구배를 보입니다.
- 접근성 변수 3개만으로 만든 예측 모델은 ROC-AUC 0.532(사실상 무작위 수준)에 그치지만, 전체 21개 변수를 쓴 모델은 0.822까지 올라갑니다 — **통계적으로 유의한 연관과 예측력은 별개**임을 보여줍니다.
- 트리 기반 모델(부스팅)로 바꿔도 성능 차이는 크지 않아(0.822→0.830), 해석 가능성을 위해 로지스틱 회귀를 택한 대가는 작습니다.
- 진짜 오즈비를 아는 합성데이터 실험으로, 교란변수 통제가 추정치를 실제 값에 수렴시키고 검출편향이 실제로 오즈비를 부풀린다는 것을 확인해 분석에 쓴 통계적 논리의 타당성을 뒷받침했습니다.

## 한계

- 설문 가중치·인종/지역 변수가 없어 전국 대표성 보정과 소득 외 격차 분석이 불가능합니다.
- 모든 변수가 자가보고(self-report)이고 2015년 단면(cross-sectional) 데이터라, 오즈비는 연관성일 뿐 인과관계를 증명하지 않습니다.
- 전체 표본의 9.5%가 완전 중복행이지만, 구간화된 범주형 변수 특성상 데이터 오류로 보기 어려워 삭제하지 않고 유지했습니다.

## 기술 스택

Python, pandas, NumPy, scikit-learn, statsmodels, matplotlib, seaborn, Jupyter — [uv](https://docs.astral.sh/uv/)로 의존성 관리

## 저자

이대호 ([GitHub @DaeHo238](https://github.com/DaeHo238))
