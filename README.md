# 2023_fall_DA_teamproject


## 갑상선 질환 분류 예측

환자의 정보를 바탕으로 갑상선 질병의 여부를 분류하는 모델을 생성하고자 하였습니다.

<br>

### 활용 데이터
https://www.kaggle.com/datasets/yasserhessein/thyroid-disease-data-set \
아래와 같이 다양한 feature가 존재하고, 예측하고자 하는 Target 변수는 크게 갑상선 기능 항진증과 갑상선 기능 저하증에 대한 진단 결과. 그 외에도 결합 단백질이나 대체 요법과 같이 추가적인 상태들을 나타내는 값들도 포함
![image](https://github.com/user-attachments/assets/782da076-8d14-4fa4-b6c6-43f607264ea2)


<br>

그리고 실제 데이터를 보면 X|Y로 두가지 상태가 포함된 데이터들이 몇 개 보이는데, 
이는 X와 일치하지만 Y일 가능성이 더 크다는 것을 의미.
![alt text](image-7.png)


<br>

### EDA 및 전처리

범주형 변수들은 대부분 true false로 이진 분류된
데이터였고, false 값이 90%정도를 차지하는 것을 확인 가능
![alt text](image-8.png)

<br>

수치형 변수들은 대부분 고르게 분포 
![alt text](image-9.png)

<br>

box plot으로 확인한 결과 이상치가 꽤 많이 존재하는 것을 확인 가능. 하지만 해당 변수에 대해서 전문지식이 없고, 값 자체는 이상치로 나타나지만 실제로 환자는 해당 수치를 가질 수 있기에 제거하는 것은 지양해야 함. 만약 이 모델이 현업에서 사용된다면, 도메인 지식이 충분한 전문가들의 의견을 통해 적절한 범위를 설정한 뒤 처리가 필요할 것으로 판단. 그 외 나이 컬럼의 이상치나 중복 컬럼 및 결측치가 너무 많은 컬럼은 삭제
![alt text](image-10.png)

<br>

target 컬럼을 단순히 양성인지 음성인지 / 갑상선 기능 항진증인지 저하증인지 음성인지에 따른 분포 확인. 모델링은 이진 분류로 진행
![alt text](image-11.png)
![alt text](image-12.png)

<br>

#### 기타 전처리
- 불필요한 변수 삭제
- 성별 결측치에 대해서는 pregnant 컬럼 값이 True인 경우 Female로 mapping 후 나머지는 drop
- 각 측정 항목 수치들의 결측치는 모두 10% 미만으로, 총 5개의 측정 항목 중 3개 이상 항목에서 동일한 결측치 분포를 보이는 샘플들은 삭제하고 나머지 부분은 KNN 을 통해 결측치를 채우는 방식을 활용
- 이상치의 분포 때문에 데이터가 대부분 양의 왜도를 갖게 되는데, 이는 log화를 진행해 최대한 대칭성을 띄도록 함
- 클래스 불균형 때문에 SMOTE로 오버샘플링
- 전체 데이터로 Logistic Regression 모델에 대한 통계 정보를 확인했을 때, 'goitre' 변수의 std err와 p-value가 타 변수에 비해 지나치게 커서 삭제
- 각 변수 별 VIF factor를 확인했을 때 'T3', 'TT4', 'T4U', 'FTI', 성별 feature의 VIF가 높아 다중 공선성 위험이 있어 feature selection 및 extraction 진행 필요 


<br>

### 활용 알고리즘
- SVM, KNN, Logistic Regression, Naive Bayes 기법 활용


### 실험 feature 조합
- 위 전처리까지 진행한 original features
- 이후 전진 선택법으로 선택한 features
- 이후 후진 소거법으로 선택한 features
- stepwise 기법으로 선택한 features
- pca 진행 시 누적 분산 비율의 elbow point인 6차원으로 추출된 features
- p-value값이 통계적으로 유의하지 않았던 변수들을 따로 추출해, 이 변수들에 대해서 2차원으로 차원 축소를 진행해 유의한 데이터와 합친 새로운 features


<br>

### 실험 결과
평가 지표는 f1-score와 2종 오류의 치명성을 고려한 recall 사용
![alt text](image-13.png)
![alt text](image-14.png)
![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-17.png)
![alt text](image-18.png)

#### 결과 요약
- NB: 학습 및 추론 시간은 적으나 두 평가지표 모두 낮음
- kNN: 추론 시간은 오래 걸리나 f1 score에서 비교적 좋은 성능
- Logistic Regression: 학습 및 추론 시간은 적고 recall에서 비교적 좋은 성능
- SVM: 학습 및 추론 시간이 매우 오래 걸리지만 대체로 f1 score와 recall 모두 높았음

<br>

### 결론

의료 데이터에서 중요한 recall 측면에서 SVM 모델을 활용하는 것이 가장 좋을 것으로 보이고, 학습 및 추론 시간이 중요한 online의 경우 recall, 속도 모두 준수한 편이었던 logistic Regression 모델을 쓰는것이 합당.

<br>

유의하지 않았던 변수의 PCA 차원축소 + 유의했던 변수 조합의 feature selection 실험 결과가 가장 뛰어난 recall 값을 보였음
