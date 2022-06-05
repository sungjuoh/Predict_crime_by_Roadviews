# Predict_crime_by_Roadviews
로드뷰 사진을 통해 서울 시내의 범죄 위험도를 예측할 수 있을까?


### 연구 주제
- 로드뷰 사진을 통해 서울 시내의 범죄 위험도를 알 수 있을까?
- 그것이 가능하다면, 로드뷰로 해당 도로의 범죄 위험도를 예측하고 위험한 지역의 건물이나 가로등 등의 지형지물을 수리함으로써 범죄를 예방할 수 있지 않을까?

### 주제 선정 동기
  범죄에 큰 관심이 없더라도 한번쯤은 들어봤을 법한 "깨진 유리창 이론"을 떠올리며 주제를 떠올리게 되었습니다. 깨진 유리창 하나를 방치해 두면 그 지점을 중심으로 범죄가 확산되기 시작한다고 주장하는 깨진 유리창 이론은 사소한 무질서를 방치하면 큰 문제로 이어질 가능성이 높다는 것을 시사하는 한편, 범죄가 발생하는 장소에 초점을 맞추고 있습니다. 범죄의 한 장면을 상상해보라 하면 많은 사람들이 쉽게 떠올릴 법한 장소와 그 느낌이 있다고 생각했고, 그러한 장소와 그 장소의 느낌을 바꾸어주면 범죄를 예방할 수 있을 것이라는 생각으로 연구를 시작하게 되었습니다.  
  실제로 범죄학의 이론으로도 정립되어 있었는데, 일상활동이론(cohen & felson, 1979)에 따르면 범죄는 동기가 부여된 범죄자가 보호자가 없는 적절한 피해자를 시간과 공간적으로 동시에 만날 때 발생한다고 합니다. 즉, 범죄 발생에 있어서 장소의 역할이 매우 중요합니다. 범죄성(criminality)을 가진 범죄자들은 범죄에 대한 동기가 크게 부여되어 있을 수 있지만, 항상 범죄를 저지르는 것은 아닙니다. 따라서 어떠한 '상황'이, '장소'가 그들로 하여금 범죄를 저지르게 하는지 또는 억제하는지를 살펴보는 것은 중요하고 장소나 건물의 디자인을 변경함으로써 범죄율을 줄일 수 있을 것이라 생각하였습니다.
  
  ### 연구 진행 순서
  1. 데이터 수집
      1) QGIS에서 구 레이어 안의 랜덤포인트를 200개씩 생성한 뒤, 해당 랜덤포인트의 위도, 경도를 카카오맵 사이트에 넣으며 로드뷰 사진        크롤링 (총 5000개)  
      2) 해당 위도, 경도에 로드뷰가 없어 기본화면이 저장된 경우 등을 포함하여 의미 없는 사진을 삭제한 뒤, 필요한 부분만 정사각형으로 자름 (최종 데이터 개수 2663개)  
      3) 서울 열린데이터광장(https://data.seoul.go.kr/) 에서 2014년 ~ 2020년 구 단위 데이터 수집(건강보험대상자 진료실적, 서울시 유흥주점 영업 인허가 정보, 어린이 보호구역 위치도, 방범용 cctv 현황 등 24개 데이터 셋 활용)  
      4) 24개의 데이터 테이블에서 필요한 정보들을 추출하여 저장하는 방식으로 raw_2014 ~ raw_2020 엑셀파일 생성
          - row : 175 (7개년 x 25개 구)
          - columns : 46 (5대 범죄 발생횟수 및 여러 구 데이터 40개)

  2. XGBoost       
      1) Regression : 5대 범죄 합계 발생횟수를 target으로 하여 40개의 변수로 regression 진행 -> feature importance 살펴 봄  
      2) Classification : 5대 범죄 합계 발생횟수 상위 50%는 unsafe, 하위 50%는 safe로 하여 이진분류 모델 학습 -> feature importance 살펴 봄  
      3) feature importance 상위 12개 변수 : 출동건수_기타활동, 유통업체 개수, 노년부양비, 인구, 인구이동(전출), 등록외국인, 기초생활보장 수급자, 65세 이상 인구, 유흥주점 개수, 인구밀도, 시장 개수, 방범용 cctv 개수 -> 이후 앙상블 모델에서 활용  
  3. CNN   
      1) 레이블 생성 : 7개년 중 4개년 이상 5대 범죄 합계 발생횟수가 상위 50%였던 12개의 구를 unsafe, 나머지를 safe로 label 생성  
      2) 이후 앙상블 모델에서 CNN 결과 레이블이 필요하기 때문에 전체 데이터의 50%만 학습에 활용하고 50%는 예측에 활용  
      3) pretrained resnet18을 활용하여 CNN 모델 학습  
  4. Ensemble
      1) 위에서 CNN을 학습시켜 나온 결과 및 XGBoost로 얻은 주요 변수 12개로 범죄 위험도를 예측하는 앙상블 모델(로지스틱 회귀) 생성


  ### 연구 결과
  1. XGBoost(Regression)  
    - Feature Importance
      ![image](https://user-images.githubusercontent.com/85203515/172050348-c04ab061-c0fb-4d2c-b838-113f8512787d.png)
    - Scatter plot of number of Crime  
      ![image](https://user-images.githubusercontent.com/85203515/172050449-c37f67d4-cf77-4f0e-9294-026ac7938b4a.png)
  2. XGBoost(Classification)
     - Confusion Matrix
      ![image](https://user-images.githubusercontent.com/85203515/172050636-c2a708f0-2a47-4fd8-b564-2bd85bf04b95.png)
     - Feature Importance
      ![image](https://user-images.githubusercontent.com/85203515/172050700-35658277-e692-4936-8d67-2807ea823211.png)
  3. CNN  
      - Confusion Matrix  
        ![image](https://user-images.githubusercontent.com/85203515/172050736-d137b27a-4076-4a24-8b86-f8f24e7fea88.png)
  4. Ensemble
      - Confusion Matrix  
        ![image](https://user-images.githubusercontent.com/85203515/172050784-b6e23411-dd77-4de6-b0cd-76573a32af88.png)

  
