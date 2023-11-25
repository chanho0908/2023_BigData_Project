# 2023_BigData_Project

## ✅ Theme: 경기도 수원시의 최근 물가 동향을 분석해보자~
### 1️⃣ Language & Libraries
1. Python   
2. pandas : 데이터 분석
3. matplotlib & seaborn : 시각화

### 2️⃣ 데이터 출처
공공 데이터 포털 : https://www.data.go.kr/data/15010720/fileData.do

### 3️⃣ 데이터 분석

* 한국인이라면 누구나 사랑하는 K-Food 한식의 전월 대비 물가 상승률을 한번 살펴보겠습니다.  
가장 높은 상승률을 보여준 품목은 **등심구이**로 **2023년 4월**에 무려 **18.6%** 나 상승했네요!  

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/fe71d851-4631-467f-9928-841db646a895)

<hr>

* 그렇다면 원자재인 소고기 물가에서도 큰 상승이 일어났을까요? 🤔

  2023년 4월에는 소고기 물가의 큰 변동은 없었으며 **2023년 8월** 에 큰 폭으로 하락이 있었습니다.

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/dd3149a6-61d3-4b57-990f-ce6ef4a33763)

<hr>

* **소고기**를 사용하는 음식점의 물가를 살펴볼까요 ?  
  준비된 데이터에서 소고기를 사용하는 음식은 갈비탕, 불고기, 설렁탕이 있습니다.  
  세가지 품목 모두 공통된 특징이 있는데 **소고기의 가격의 큰 하락이 있던 8월에 동시에 물가 하락**이 있었습니다.

  😊 이를 바탕으로 원자재 값의 하락에 따른 관련 업종 음식의 물가 하락이 있었다고 예측 할 수 있겠죠 !
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/ea826164-f8d8-407a-863b-576628307f93)

<hr>

🤔 다음으로는 **수산물**의 물가를 살펴보겠습니다. 수산물은 고등어, 갈치, 오징어, 동태가 있습니다.  
   이 중 물가 상승률이 가장 큰 항목은 **2023-5월** 의 갈치가 **0.44%** 로 가장 큰 전년대비 상승폭을 보여줬습니다.  
   물가 상승률이 가장 적은 항목은 **2023년-5월** 동태로 **0.17%** 하락 하였습니다. 

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/7a612f42-d9a8-4ce9-9c80-580c67a8956d)

<hr>

🥠🥠 다음으로 국민 음식 **치킨**을 살펴볼까요 ? 🥠🥠
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/9f3846ae-fa29-4fb8-b039-c0e9b6dbfe59)

<hr>

> 2023년 01월에 **10%** 로 매우 높은 상승률을 기록했는데요 기사에서 확인할 수 있듯이  
러시아와 우크라이나간의 전쟁으로 인한 전 세계적인 물가 상승이 원인으로 보입니다.<br>  
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/d7142a4a-fbc8-4cba-8a2b-bc4abbe9746d)

<hr>

> **1월 최고 상승률**을 보여주고 이후로는 **지속적인 하락세**를 보여주었는데요 기사에 따르면  
이는 4월부터 대한민국 정부에서 치킨 물가 안정을 위한 정책을 펼쳤기 때문으로 예상됩니다.  
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/af6c1d0a-c243-4b44-a42b-ea0ef23a8800)

<hr>

### 4️⃣ 데이터 예측
 ##### 그럼 이제 다양한 회귀 모델을 사용해 치킨 값을 예측해 보겠습니다.
 ####  ✔ LinearRegression
   ```  
  # 특성 선택
  X = df_chicken['기준일'].dt.month.values.reshape(-1, 1)
  y = df_chicken['물가동향'].values
  
  # 훈련 및 테스트 데이터 분리
  X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
  
  # 회귀 모델 학습
  model = LinearRegression()
  model.fit(X_train, y_train)
  
  # 테스트 데이터에 대한 예측
  y_pred = model.predict(X_test)
  
  # 평가
  mse = mean_squared_error(y_test, y_pred)
  rmse = np.sqrt(mse)
  print(f'Root Mean Squared Error (RMSE): {rmse}')
  
  ```
 #### 결과
    RMSE : 300.550
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/90a08b8a-f9d4-4f9d-a167-549cd62c0d24)


####  ✔ DecisionTreeRegressor
  ```
  # 특성 선택
  X = df_chicken['기준일'].dt.month.values.reshape(-1, 1)
  y = df_chicken['물가동향'].values
  
  # 훈련 및 테스트 데이터 분리
  X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
  
  # 결정 트리 모델 학습
  model = DecisionTreeRegressor(max_depth=3) 
  model.fit(X_train, y_train)
  
  # 테스트 데이터에 대한 예측
  y_pred = model.predict(X_test)
  
  # 평가
  mse = mean_squared_error(y_test, y_pred)
  # RMSE 계산 및 출력
  rmse = np.sqrt(mse)
  print(f'Root Mean Squared Error (RMSE): {rmse}')
  ```
  #### 결과
    RMSE : 394.1763
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/5bdcb536-1660-44b1-8136-539420631edf)


####  ✔ 트리 기반의 회귀 모델 [ SVR, AdaBoostRegressor, XGBRegressor, LGBMRegressor ] 및 K겹 교차 검증
  ```
    def get_model_cv_prediction(model, X_data, y_target):
        neg_mse_scores = cross_val_score(model, X_data, y_target, 
                                         scoring="neg_mean_squared_error", cv=7)
        rmse_scores = np.sqrt(-1 * neg_mse_scores)
        avg_rmse = np.mean(rmse_scores)
        print('##### ', model.__class__.__name__, ' #####')
        print(' 7 교차 검증의 평균 RMSE : {0:.3f} '.format(avg_rmse))
    
    y_target = reg_df_chicken['물가동향']
    X_data = reg_df_chicken[['Month']]
    
    ada_reg = AdaBoostRegressor()
    xgb_reg = XGBRegressor()
    
    # 트리 기반의 회귀 모델을 반복하면서 평가 수행
    models = [ada_reg, xgb_reg]
    for model in models:
        get_model_cv_prediction(model, X_data, y_target)
  ```
    
   #### 결과
    #####  AdaBoostRegressor  #####
    7 교차 검증의 평균 RMSE : 826.094 
    #####  XGBRegressor  #####
    7 교차 검증의 평균 RMSE : 855.070 

  ![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/4ff885b6-18db-4906-8fc0-3393fc53931b)

