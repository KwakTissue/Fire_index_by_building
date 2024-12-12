# 소방안전 빅데이터 공모전
> 프로젝트 진행 기간 : 2024.10.07 ~ 2024.10.21 \
> 건물별 화재 발생시 피해 예측을 위한 화재 확산 속도 지수 개발

### 목차
1. 배경
2. 목적
3. 활용 데이터
4. 데이터 분석 및 모델링
5. 기대효과
6. 개인적인 회고
## 1. 배경
화재 사고는 인명과 재산에 큰 피해를 줄 수 있으며, 신속한 초기 대응이 피해를 줄이는 핵심요소이다. 본 연구는 건물의 자재, 구조 등 다양한 변수에 따라 달라지는 화재 확산 양상을 분석하여 맞춤형 화재 대응 전략을 제안하고자 한다.
## 2. 목적
화재사고는 예상치 못하는 상황에서 발생하며, 그 피해는 빠른 시간 동안 급격히 상승한 다. 현행 화재 대응 시스템에서 초기진압 골든타임은 일반적으로 7분으로 정의되어 있지만, 모든 상황에 적합하지 않으며, 이런 일률적인 골든타임을 기준으로 한 화재 대응은 비효율적 으로 이루어질 가능성이 높다. 그러므로 각 건물의 특성들을 종합적으로 고려한 세부적인 화 재 확산 양상 예측이 필수적이다.

따라서 본 연구의 목적은 건물의 구조적 특성, 사용 용도, 재료 등을 종합적으로 반영하여 건물별 화재 확산 속도 지수를 개발하고, 이를 활용하여 진압소요시간에 따른 화재 피해량을 예측한다.
## 3. 활용데이터
1. 화재출동현황2023 (https://www.bigdata-119.kr/)
2. 건축물대장 - 표제부 (https://open.eais.go.kr/)
3. 건물 에너지정보 - 전기에너지, 가스에너지 (https://open.eais.go.kr/)
## 4. 데이터 분석 및 모델링
### 1) 데이터 준비
1. 데이터 수집
  - 화재출동 현황 2023 \
    건축물대장, 건물 에너지정보 데이터가 지번주소 기준임을 확인 후, 카카오 API를 사용하여 GIS_X좌표, GIS_Y좌표를 기준으로 지번주소를 생성
  - 건축데이터개방_건축물대장 표제부 \
    공공데이터포털 API를 사용하여, 지번주소를 기준으로 건물용도, 지붕자재, 사용승인일, 용적률, 용적률산정연면적, 연면적, 지상층수, 지하층수, 승용승강기수, 비상용승강기수 데이터를 수집 \
    이때 같은 지번주소에 여러 개의 표제부 정보가 존재함을 확인하여, 데이터 정합성을 위해 화재출동 현황 2023와 표제부 데이터의 연면적의 차가 가장 작은 데이터를 수집
  - 건물 에너지정보 \
    공공데이터포털 API를 사용하여, 지번주소를 기준으로 화재발생 달의 전기사용량, 가스사용량 데이터를 수집
  - 초기 수집 데이터 변수 정보 \
    <img width="529" alt="image" src="https://github.com/user-attachments/assets/99b71096-cfb9-4d70-a4b9-377442bad38d" />
2. 데이터 전처리
* 결측치 처리
  * `사용승인일` : 311개 제거
  * `풍속` : 15개 결측치를 같은 날 풍속 데이터가 존재할 경우에 따라 처리
    * 존재할 경우 : 같은 날 풍속 평균 사용
    * 존재하지 않을 경우 : 전날 다음날 풍속 평균 사용
  * `전기사용량 & 가스사용량` : 전기사용량 1932개, 가스사용량 2307개 결측치를 머신러닝 예측을 통해 결측치 처리
    * 예측과정
      1. 전기사용량, 가스사용량과 다른 변수 상관관계 확인 후, 상관계수가 0.2 이상인 용적률산정연면적, 층수, 승강기개수, 기온, 습도, 풍속 변수 사용
      2. Standard 스케일링과 XGBoost 모델를 사용
      3. 가스사용량 모델 R2 : 0.43, 전기사용량 모델 R2 : 0.45
* 변수 생성
  * `총층수`, `승강기개수`, `노후도`, `진압소요시간` : 생성
  * `화재발생시각`, `주중 및 주말` : 재범주화
  * 건물 용도 및 자재 변수 재범주화
    * `화재확산_건물용도` : 건축물의 용도와 화재 확산에 영향을 미치는 특성에 따라 4가지 변수로 재범주화
      1. 산업시설 : 연소성 물질이 많고 기계장비등을 많이 구비한 시설
      2. 주거밀집시설 : 인구밀집도가 높은 거주 및 돌봄시설
      3. 공공시설 : 비교적 밀도가 낮은 시설
      4. 대형집합시설 : 대규모인원이 모이는 공공 및 오락시설
    * `화재확산_건물자재` : 건축 자재들을 화재 확산 속도에 영향을 주는 수준에 따라 재범주화
      1. 고연소성 자재 : 가연성이 높으며, 구조적으로 불에 약한 자재
      2. 중간연소성 자재 : 기본적으로 불연성 자재이지만, 열이 가해 지면 금이 가거나 일부 손상이 가는 자재
      3. 중간위험 자재 : 합성재료이기 때문에 열에 약하거나 구조적 특성등의 복합적 요소에 따라 위험정도가 가변적인 자재
      4. 복합 고위험 자재 : 구조 내부에 있는 단열제의 소재에 따라 화재확산속도가 매우 빠른 자재
      5. 불연성 자재 : 불연성자재로써 내화성이 높고 구조적으로 안정적인 자재
    * `화재확산_지붕자재` : 지붕 자재명칭이 화재 확산에 미치는 영향이 잘 확인되도록 변수명을 재정의
      1. 차단형 : 화재에 대한 저항력이 매우 높고, 불이 퍼지기 어려운 자재
      2. 중간차단형 : 일정한 내화성이 있으나, 화재에 장시간 노출되면 손상이 가능한 자재
      3. 약차단형 : 초기 화재에는 저항하지만, 비교적 빠르게 열에 의해 손상되거나 파괴될 가능성이 있는 자재
* 화재확산속도지수(FSI) 생성   
  화재확산속도지수는 화재가 얼마나 빠르게 확산되었는지를 추정하는 지수로, 화재피해와 건물의 면적과 화재성장을 전반적으로 반영
  * 계산식   
    <img width="464" alt="image" src="https://github.com/user-attachments/assets/794e4372-74d0-4995-9443-823654050f99" />   
  * 설명   
    1. 재산피해금액은 화재피해 규모를 나타내고, 피해가 클수록 화재 확산 속도가 높았을 것으로 추정
    2. 화재성장곡선은 선행연구에서 시간 제곱임을 확인 후, 재산피해금액을 진압소요시간의 제곱으로 나눔
    3. 단위 면적에 대한 지표로 사용하기 위해 연면적으로 나눔
3. 분석 전 최종데이터셋   
   <img width="540" alt="image" src="https://github.com/user-attachments/assets/7a1b8703-248d-4e3e-b734-551c909e565d" />
### 2) EDA
1. 화재확산속도지수 기초통계량   
   <img width="469" alt="image" src="https://github.com/user-attachments/assets/1f88ef4d-f4d4-4f97-97be-b61155c0a3b1" />
   <img width="485" alt="image" src="https://github.com/user-attachments/assets/e3377a00-e35d-4f5f-aac8-f67741258a4d" />
   * Boxplot과 FSI의 기초통계량 분석 결과, FSI의 표준편차가 사분위수에 비해 매우 높아 최대 값과 최소값 간의 차이가 크며, 이상치가 다수 존재하는 것을 확인
   * 분위수를 기준으로 IQR를 계산하여 x < Q1 - 1.5 * IQR, x > Q3 + 1.5 * IQR인 값을 제거하는방식으로 정제
2. 독립변수 기초통계량확인
   * 수치형 변수
     <img width="459" alt="image" src="https://github.com/user-attachments/assets/63f3e61a-af07-4b75-9c2d-f9925d41851f" />
     <img width="527" alt="image" src="https://github.com/user-attachments/assets/aea80b18-f61d-4288-983c-a7e2d166f33a" />
     * 대체로 이상치가 많은 것을 보아 모델 적용 전 독립변수를 이상치의 영향을 줄이는 RobustScaling 기법을 활용하여 스케일링이 필요한 것을 판단
   * 범주형 변수
     <img width="518" alt="image" src="https://github.com/user-attachments/assets/3db35f7e-269e-4dfc-9ad2-9ea2cacae011" />
     <img width="543" alt="image" src="https://github.com/user-attachments/assets/9a6c74c8-4596-47c0-b03e-ab7388ab3bc1" />
     * 건물의 상태를 확인할 수 있는 주요변수인 건물용도, 건물자재, 지붕자재의 기초통계량 확인 결과 건물용도에서는 주거시설과 공공시설이 큰 비중을 차지하며, 건물자재에서는 불연성자재, 지붕자재에서는 차단형자재가 큰 비중을 차지함을 확인
3. 상관 분석
   * 상관관계 분석 표   
     <img width="311" alt="image" src="https://github.com/user-attachments/assets/c094d6d2-7e25-4089-ab84-b20f00f7dfa2" />
     * 상관분석 결과 FSI에 대한 상관계수가 총층수가 -0.348로 가장 크고 전기사용량 0.247, 승강기개수 -0.181, 노후도 0.155로 FSI에 대해 주요한 변수로 확인
     * spst의 pearson-r를 활용하여 자세한 상관분석을 진행
  * spst를 사용한 pearson-r 상관계수   
    <img width="362" alt="image" src="https://github.com/user-attachments/assets/4a60c61c-8e44-4908-a6c7-e8f80f9de96c" />
  * 그래프   
    <img width="537" alt="image" src="https://github.com/user-attachments/assets/d37a7f5a-84ec-41e2-bdf5-5bf4820a2543" />   
  * 결과
    * FSI에 대해 유의한 변수 : 노후도, 총층수, 승강기개수, 전기사용량, 가스사용량
    * 위 산점도와 회귀직선 그래프를 확인했을 때 데이터와 회귀직선 사이에 선형성이 나타나지 않아 모델링 과정에서 다중선형회귀모델은 적합하지 않을 것으로 판단
4. T/ANOVA-test
  * T/ANOVA-test 결과표
    <img width="296" alt="image" src="https://github.com/user-attachments/assets/39a29cfa-caad-4c79-bf36-389b56aac9ed" />
  * 그래프   
    <img width="493" alt="image" src="https://github.com/user-attachments/assets/be7213a4-aefc-4b47-903f-2cd061b5b72e" />
  * 결과
    * FSI에 대해 유의한 변수 : 화재확산_건물용도, 화재확산_건물자재, 화재확산_지붕자재
    * FSI의 개발 의도에 따라 건물별 특징이 FSI에 대해 유의한 것을 확인
  ### 3) 모델링
 1. 모델 선택 이유와 진행과정
   * 화재확산속도지수를 통해 건물들의 화재피해를 예측하고자 XGBoost 모델을 사용하여 진행
   * XGBoost 모델 선택 이유
     * 정형 데이터에서 신경망은 noisy feature에 쉽게 과적합되어 대부분의 경우 decision tree 계열의 모델보다 성능이 떨어짐
     * 독립변수와 종속변수 간에 선형성이 나타나지 않아 다중선형회귀 또한 적절하지 않음
     * 과적합 가능성을 줄이고 비선형 관계를 포착할 수 있는 트리기반모델인 XGBoost를 채택
    * 진행과정
      *  사용할 데이터의 크기가 1971개로 작다고 판단하여, GridSearchCV를 통해 하이퍼 파라미터 튜닝을 진행
      *  모델링 과정에서의 중요 변수를 도출하기 위해 Sklearn의 SelectKBest를 통해 feature selection을 진행
2. 데이터 스케일링
  * Feature로 사용할 데이터들은 전처리 단계에서 기초통계량을 확인했을 때 이상치들이 포함 되어 있는 것으로 확인되어 이상치에 영향이 적은 Sklearn의 RobustScaler를 사용
  * Target으로 사용할 FSI의 왜도와 첨도를 확인했을 때 왜도는 1.7로 비대칭적인 분포를 나타내고 첨도는 2.3으로 높게 나타나 log scaling을 진행
3. 하이퍼파라미터 튜닝
  * XGBoost의 하이퍼파라미터 튜닝을 위해 Sklearn의 GridSearchCV를 통해 튜닝   
```
param_grid = {‘max_depth’: [2, 3, 4, 5, 6],
              ‘n_estimators’: [100, 200, 300, 400, 500, 600, 700],
              ‘learning_rate’: [0.1, 0.05, 0.01, 0.015]}
```   
  * 최종 하이퍼파라미터로는 `{'learning_rate': 0.01, 'max_depth': 2, 'n_estimators': 300}`이 채택
4. Feature Selection
  * Sklearn의 SelectKBest를 통해 1부터 총 feature 수인 13까지의 k를 사용해 어떤 feature로 구성할지 선택
  * Training loss와 validation loss를 확인했을 때 k=8일 때 training loss가 가장 작았고, validation loss가 가장 작았던 k=4인 경우보다 R2 값이 더 높아 k=8로 선정
    * Training & Validation Loss 그래프   
      <img width="319" alt="image" src="https://github.com/user-attachments/assets/28e38997-2c7d-482f-9ed0-14108e1f414e" />
  * 상관관계 및 유의확률 분석을 통해 도출한 화재확산속도지수에 대해 유의미한 변수와 Feature Selection에서 선택된 8개의 변수는 동일
  *  `전기사용량`, `가스사용량`, `화재확산_지붕자재`, `화재확산_건물자재`, `화재확산_건물용도`, `승강기개수`, `총층수`, `노후도` 변수로 모델링을 진행
5. 모델링 결과
  * 5-Fold 검증을 통해 학습된 XGBoost모델의 평가지표
    * `R2` : 0.48967
    * `MAE`: 1.51675
  * Feature Importances   
    <img width="361" alt="image" src="https://github.com/user-attachments/assets/44a1f2cf-852f-4f03-a27c-707d27d460e4" />
6. 최종 결과
  * 상관관계 및 유의확률 분석을 통해 도출한 변수 8개와 feature selection을 통해 선택된 변수 8개가 동일
  * 모델링 결과 `총층수(51.1%)`, `승강기개수(19.9%)`, `화재확산_건물자재(8.6%)`, `화재확산_건물용도(7.2%)`, `전기사용량(4.6%)`, `가스사용량(4.0%)`, `노후도(2.6%)`, `화재확산_지붕자재(1.9%)` 순으로 화재확산속도지수에 높은 비중으로 영향을 미치는 것으로 분석




   










    
