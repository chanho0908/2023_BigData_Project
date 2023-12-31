# 2023_BigData_Project

## ✅ Theme: 경기도 수원시의 최근 물가 동향을 분석해보자~
### Language & Libraries
1. Python   
2. pandas : 데이터 분석
3. matplotlib & seaborn : 시각화
4. 공공 데이터 포털 : https://www.data.go.kr/data/15010720/fileData.do

## 1️⃣ 데이터 전처리
#### **1. 데이터 프레임 리스트를 가져온 후 필요없는 열을 삭제합니다.**
```
# 데이터 프레임 리스트
dfs = [
    df_20220228, df_20220324, df_20220424, df_20220524, df_20220624, df_20220724, df_20220824, df_20220926, df_20221024, df_20221125,
    df_20221227, df_20220125, df_20230228, df_20230328, df_20230428, df_20230526, df_20230626, df_20230726, df_20230825, df_20230926
]

# 데이터 프레임 합치기
firstDataFrame = pd.concat(dfs, ignore_index=True)

# 필요없는 열 삭제
firstDataFrame.drop(['데이터기준일자', '규격 및 단위'], axis=1, inplace=True)
````
#### **2. Json에서 값이 ""일 경우 판다스는 누락된 값이 아닌 빈 문자열로 처리하기 때문에 물가동향 열에서 결측치를 NaN으로 변환하고 행을 삭제합니다.**
```
# Json에서 값이 ""일 경우 판다스는 누락된 값이 아닌 빈 문자열로 처리
# 물가동향 열에서 빈 문자열을 NaN으로 변환
firstDataFrame['물가동향'].replace('', pd.NA, inplace=True)

# 숫자로 변환이 불가능한 값은 NaN으로 처리
firstDataFrame['물가동향'] = pd.to_numeric(firstDataFrame['물가동향'], errors='coerce')

# NaN 값을 가진 행을 삭제
df_natnan = firstDataFrame.dropna(subset=['물가동향'])
```

#### **3. 물가 동향의 결측치도 제거해 줍니다.**
```
# 물가 동향의 값이 빈 경우 확인
no_values_count = df_natnan['물가동향'].isna().sum()
#print('물가 동향의 값이 비어있는 경우: ', no_values_count)

# 물가 동향의 값이 데이터 미존재인 경우 확인
noData = (df_natnan['물가동향'] == '데이터미존재').sum()
#print('물가 동향의 값이 데이터미존재인 경우: ', noData)

```
#### **4. 현재 데이터는 한 달 동안 한 가지 품목의 물가에 대해서 날짜에 따라 여러번 작성됨 한 달 동안 가장 높은 가격을 기록한 날만 남기고 나머지 데이터는 삭제.**

```
# 데이터프레임을 '기준일', '품목' 및 '물가동향' 열을 기준으로 그룹화
# 각 그룹에 대해 '물가동향' 열에서 최대 값을 가지는 행을 선택
grouped_df = df_natnan.groupby(['기준일', '품목'])['물가동향'].idxmax()

# 선택한 행의 인덱스를 사용하여 원래 데이터프레임에서 해당 행을 추출하여 새로운 데이터프레임을 생성
mainDataFrame = df_natnan.loc[grouped_df]

#삭제된 행 확인
#deleted_rows = mainDataFrame[~mainDataFrame.index.isin(filtered_df.index)]
#deleted_rows

mainDataFrame

```
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/3483ddac-0a06-48be-b47e-fa217afcae85)



### 3️⃣ 데이터 분석

* 한국인이라면 누구나 사랑하는 K-Food 한식의 전월 대비 물가 상승률을 한번 살펴보겠습니다.   
```
# 한식만 추출
korean_food = mainDataFrame[mainDataFrame['구분'].str.contains('한식')].copy()
# 중복을 제거한 한식 품목 추출
k_food_list = korean_food['품목'].drop_duplicates().tolist()
# k_food_list에 저장된 리스트 저장
df_kfood = mainDataFrame[mainDataFrame['품목'].isin(k_food_list)]

# DataFrame을 '품목' 컬럼으로 그룹핑 하고, '물가동향' 컬럼에 12개월(12개월, 즉 1년을 나타냄)
# 로 df_kfood 변화율( ) 함수를 적용하여 전년 대비 인플레이션을 계산
df_kfood['전년동월대비'] = df_kfood.groupby('품목')['물가동향'].pct_change(periods=12)
df_kfood.dropna(inplace=True)
```
* 가장 높은 상승률을 보여준 품목은 **등심구이**로 **2023년 4월**에 무려 **18.6%** 나 상승했네요!  

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/fe71d851-4631-467f-9928-841db646a895)

<hr>

* 그렇다면 원자재인 소고기 물가에서도 큰 상승이 일어났을까요? 🤔
  ```
    # 2023년 쇠고기 품목만 추출
    df_beef = mainDataFrame[mainDataFrame['품목'] == '쇠고기']
    df_beef = df_beef[df_beef['기준일'].str.contains('2023')]
    df_beef['전년동월대비'] = df_beef['물가동향']
  ```  
  2023년 4월에는 소고기 물가의 큰 변동은 없었으며 **2023년 8월** 에 큰 폭으로 하락이 있었습니다.

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/dd3149a6-61d3-4b57-990f-ce6ef4a33763)

<hr>

* **소고기**를 사용하는 음식점의 물가를 살펴볼까요 ?  
  준비된 데이터에서 소고기를 사용하는 음식은 갈비탕, 불고기, 설렁탕이 있습니다.
  ```
    df_beef = mainDataFrame[mainDataFrame['품목'] == '쇠고기']
    df_use_beef_items = ['갈비탕', '불고기', '설렁탕']
    
    # 품목이 df_use_beef_items에 있는 데이터만 필터링
    df_selected_beef_items = mainDataFrame[mainDataFrame['품목'].isin(df_use_beef_items)].copy()
    
    # 전년동월대비 물가 상승률 계산
    df_selected_beef_items['전년동월대비'] = df_selected_beef_items.groupby('품목')['물가동향'].pct_change(periods=12)
    df_selected_beef_items.dropna(inplace=True)
  ```
  세가지 품목 모두 공통된 특징이 있는데 **소고기의 가격의 큰 하락이 있던 8월에 동시에 물가 하락**이 있었습니다.

  😊 이를 바탕으로 원자재 값의 하락에 따른 관련 업종 음식의 물가 하락이 있었다고 예측 할 수 있겠죠 !
![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/ea826164-f8d8-407a-863b-576628307f93)

<hr>

🤔 다음으로는 **수산물**의 물가를 살펴보겠습니다. 수산물은 고등어, 갈치, 오징어, 동태가 있습니다.     
```
    df_seafood = mainDataFrame[mainDataFrame['구분'].str.contains('수산물')].copy()
    seafood_list = df_seafood['품목'].drop_duplicates().tolist()
    seafood = mainDataFrame[mainDataFrame['품목'].isin(seafood_list)]
    
    # '품목'을 기준으로 그룹화하고 '물가동향' 열에서 전년동월대비 물가 상승 지수 계산
    df_seafood['전년동월대비'] = df_seafood.groupby('품목')['물가동향'].pct_change(periods=12)
    df_seafood.dropna(inplace=True)
    
    # 결과 확인
    # '고등어' 품목 선택
    df_mackerel = df_seafood[df_seafood['품목'] == '고등어']
    # '갈치' 품목 선택
    df_cutlassfish = df_seafood[df_seafood['품목'] == '갈치']
    # '동태' 품목 선택
    df_dongtae = df_seafood[df_seafood['품목'] == '동태']
    # '오징어' 품목 선택
```
   이 중 물가 상승률이 가장 큰 항목은 **2023-5월** 의 갈치가 **0.44%** 로 가장 큰 전년대비 상승폭을 보여줬습니다.  
   물가 상승률이 가장 적은 항목은 **2023년-5월** 동태로 **0.17%** 하락 하였습니다. 

![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/b3a51436-e912-482e-a874-94e5c5689562)


<hr>

🥠🥠 다음으로 국민 음식 **치킨**을 살펴볼까요 ? 🥠🥠
```
# 치킨 데이터만 분리
df_chicken = mainDataFrame[mainDataFrame['품목'] == '치킨'].copy()

# 전년도 동월 가격과 비교
df_chicken['전년동월대비'] = df_chicken['물가동향'].pct_change(periods=12)
df_chicken.dropna(inplace=True)

# 시각화할 데이터만 분리
df_chicken = df_chicken[['기준일', '전년동월대비']]
```
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

### 4️⃣ 네이버 API 크롤링
 **크롤링**을 통해 **수원시 물가**에 대한 기사를 좀 더 분석해 보겠습니다.  
  > ### 1. 프로그램 구성 설계하기
    def main()

    1. 검색어 지정
    
    2. 네이버 뉴스 검색
    
    3. 응답 데이터 정리 후 리스트에 저장
    
    4. 리스트를 JSON 파일로 저장

  ```
  #1. 검색어 지정
  def getRequestUrl(url):
    req = urllib.request.Request(url) # 매개변수로 받은 url에 대한 요청을 보낼 객체를 생성
    req.add_header("X-Naver-Client-Id", client_id) # API를 사용하기 위한 Client ID와 Client Secret 코드를 요청 객체 헤드에 추가
    req.add_header("X-Naver-Client-Secret", client_secret)

    try:
        response = urllib.request.urlopen(req) # 요청 객체를 보내고 그에 대한 응답을 받아 response 객체에 저장
        if response.getcode() == 200: # 정상 처리
            print ("[%s] Url Request Success" % datetime.datetime.now()) # 정상 처리 메시지 출력
            return response.read().decode('utf-8') # utf-8 형식으로 디코딩 후 반환
    except Exception as e:
        print(e)
        print("[%s] Error for URL : %s" % (datetime.datetime.now(), url))
        return None
  ```
```
  # 2. 네이버 뉴스 검색
    #[CODE 2]
    def getNaverSearch(node, srcText, start, display):
      base = "https://openapi.naver.com/v1/search" 
      node = "/%s.json" % node
      parameters = "?query=%s&start=%s&display=%s" % (urllib.parse.quote(srcText), start, display)
  
      url = base + node + parameters # 네이버 검색 API 정보에 따라 요청 URL을 구성
      responseDecode = getRequestUrl(url)   #[CODE 1]
  
      if (responseDecode == None):
          return None
      else:
          return json.loads(responseDecode) # 서버에서 받은 JSON 형태의 응답 객체를 파이썬 객체로 로드하여 반환
```

```
    # 3. 응답 데이터 정리 후 리스트에 저장
    #[CODE 3]
    def getPostData(post, jsonResult, cnt):
        # 검색 결과가 들어 있는 post 객체에서 필요한 데이터 항목을 추출하여 변수에 저장
        title = post['title']
        description = post['description']
        org_link = post['originallink']
        link = post['link']
    
        pDate = datetime.datetime.strptime(post['pubDate'],  '%a, %d %b %Y %H:%M:%S +0900')
        pDate = pDate.strftime('%Y-%m-%d %H:%M:%S') # 날짜를 '연-월-일 시:분:초' 형식으로 나타낸다.

        # 데이터를 딕셔너리로 구성하여 리스트 객체인 jsonResult에 추가
        jsonResult.append({'cnt':cnt, 'title':title, 'description': description,
        'org_link':org_link,   'link': org_link,   'pDate':pDate})
            return
```

```
  # 4. 데이터를 JSON으로 저장
   #[CODE 0]
    def main():
        node = 'news'   # 크롤링 할 대상
        srcText = input('검색어를 입력하세요: ')
        cnt = 0
        jsonResult = []
    
        jsonResponse = getNaverSearch(node, srcText, 1, 100)  #[CODE 2]
        total = jsonResponse['total']
    
        while ((jsonResponse != None) and (jsonResponse['display'] != 0)):
            for post in jsonResponse['items']:
                cnt += 1
                getPostData(post, jsonResult, cnt)  #[CODE 3]
    
            start = jsonResponse['start'] + jsonResponse['display']
            jsonResponse = getNaverSearch(node, srcText, start, 100)  #[CODE 2]
    
        print('전체 검색 : %d 건' %total)
    
        with open('%s_naver_%s.json' % (srcText, node), 'w', encoding='utf8') as outfile:
            jsonFile = json.dumps(jsonResult,  indent=4, sort_keys=True,  ensure_ascii=False)
    
            outfile.write(jsonFile)
    
        print("가져온 데이터 : %d 건" %(cnt))
        print ('%s_naver_%s.json SAVED' % (srcText, node))
    
    if __name__ == '__main__':
        main()
```
  ### 크롤링 데이터 확인
  ![image](https://github.com/chanho0908/2023_BigData_Project/assets/84930748/639f10fb-3a6c-44ae-bb97-1691abcc9310)


> ### 2. 워드 클라우딩
```
mask_path = 'cloud.png'
cloud_mask = np.array(Image.open(mask_path)) # 구름 모양 워드 클라우드 생성

wc = WordCloud(font_path, background_color='white', width=800, height=600, mask=cloud_mask, colormap='inferno')
cloud=wc.generate_from_frequencies(word_count)
```
  크롤링된 워드를 활용하여 구름 모양 워크 클라우드를 만들었습니다.  
  
  <img src="https://github.com/chanho0908/2023_BigData_Project/assets/84930748/1c11ecf9-ddde-45c6-a8ae-a098fb6027fe" width="400"/><img src="https://github.com/chanho0908/2023_BigData_Project/assets/84930748/f39412c5-9b18-4576-ae26-2f45ee9d5867" width="400"/>

<hr>

### 5️⃣ 데이터 예측
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

