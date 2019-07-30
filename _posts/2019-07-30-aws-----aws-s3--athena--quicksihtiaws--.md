---
layout: post
title: "AWS에서 데이터처리 맛보기 AWS S3 , Athena , Quicksight"
description: ""
category: dev
tags: ["AWS", "데이터처리", "athena"]
comments: true
thumbnail: http://drive.google.com/uc?export=view&id=1ErRHkReP5Ic0-s_euuv8BdFMO6Famb4z
---
### Prologue

이번 여름방학은 연세대 HCI랩에서 진행하는 드림아카데미 프로그램에 참여 중이다.

드림아카데미는 UX를 고려한 서비스 디자인 방법을 공부하고 실제 기획하는 프로그램이라 

팀원들과 빡🔥세🔥게🔥 구르고 있다 

![](http://drive.google.com/uc?export=view&id=174IXJAhTyb9E0OwuYDyjLGqfHZ5xJ92F)

이 프로그램을 운영하시는 김진우 교수님께서 올해는 Connected AI라는 프로젝트 주제를 정해주셨다. 

그래서 데이터를 이용한 서비스를 결국 기획하긴 했는데,,  AI도 데이터도 모르고 서버 개발이랑 언어 위주로 한 

개발자 나부랭이지만 일단 팀에 유일한 개발자라 일단 데이터 처리 과정을 만들어 볼 수 밖에 없었다.

---

### 데이터 선정

아직 서비스를 만든게 아니기 때문에 먼저 기존에 있는 데이터셋을 가지고 시작!

데이터셋은 [https://www.yelp.com/dataset](https://www.yelp.com/dataset) 음식 리뷰 어플인 Yelp의 Dataset을 이용했다.

    {
       "review_id": "zdSx_SD6obEhz9VrW9uAWA",
       "user_id": "Ha3iJu77CxlrFm-vQRs_8g",
       "business_id": "tnhfDv5Il8EaGSXZGiuQGg",
       "stars": 4,
       "date": "2016-03-09",
       "text": "Great place to hang out after ...",
       "useful": 0,
       "funny": 0,
       "cool": 0
    }

Yelp Dataset은 위와 같이 JSON 형식으로 총 8Gb크기의 데이터셋을 제공한다.

서비스에 대한 리뷰, 가게명, 유저 정보등이 들어가 있는데 이걸 직접 읽어서 처리하는걸 하려면 할 수는 있는데,,, 

매번 새로운 분석을 하려면 코딩을 또 해야하고 귀찮아 질 것 같아서... 최대한 일을 안할 수 있는 방법으로...

&nbsp;

어떻게 하지 고민하다가 전에 친구랑 밥 먹으면서 AWS를 이용해서 데이터 분석을 할 수 있다는 말을 얼핏 들었던것 같아서 찾아보니 **[AWS Athena](https://aws.amazon.com/ko/athena/)** 라는 서비스를 찾았다.

써본 AWS 서비스라곤 ec2 인스턴스, 도메인 적용하기 위한 route53 딱 두 개 뿐이지만... (심지어 S3도 따로 안써봄) 

그래도 서버 쪽 하려면 AWS공부도 한 번 해봐야 할 것 같아서 도전해봤다..

---

### AWS 아이쇼핑

![](http://drive.google.com/uc?export=view&id=1iI4DerfABaiUPCltu8TgIaByv76jzGhM)

> AWS 서비스를 이용한 아키텍처 - 출저 [https://aws.amazon.com/ko/blogs/korea/serverless-architecture-by-korean-developers/](https://aws.amazon.com/ko/blogs/korea/serverless-architecture-by-korean-developers/)

데이터 파이프라인은 유저로 부터 들어오는 데이터를 **수집.** 전처리 과정을 거쳐 필요한 데이터만 **정리**. 

데이터베이스에 **저장**. 저장된 데이터를 다시 **분석 및** **이용**하는 과정을 말한다.
   
   &nbsp;
  
AWS에서는 이렇게 데이터 파이프라인을 쉽게 만들 수 있도록 다양한 서비스들을 제공하는데 대충 찾아보니 요즘 자주쓰는 것들은

- Kinesis - 들어오는 데이터들을 여러 포맷으로 실시간으로 변환해서 스트리밍
- S3 - 데이터 저장소. 여러 AWS 서비스들에서 주소만 주면 땡겨 쓸 수 있다.
- DynamoDB - 풀옵션 NoSql Database. 클러스터링, 백업 등등 다 해준다.
- Athena - SQL을 이용해서 S3에 저장된 데이터를 질의 할 수 있다.
- QuickSight - 여러 AWS 서비스에 저장된 데이터를 이용해서 시각화를 간단하게 할 수 있다.

등등... 많은게 있지만 공간이 부족해서 생략한다.

![](http://drive.google.com/uc?export=view&id=1Aw3mrWdyarelDv0F-3hDL4sIPm3o4oMQ)

나는 이번에 맛보기로 간단하게 하는 거기 때문에 데이터를 `S3`에 저장 `Athena`에서 불러와서 `QuickSight`로 시각화 하는 것만 테스트 해봤다. 

데이터를 데이터 베이스에 밀어넣을 수도 있지만 NoSql을 쓸 정도로 큰 데이터 양은 아닌 것 같고 Athena 설명대로 따로 테이블 같은걸 안만들고 바로 JSON을 질의 넣을 수 있고 시각화까지 해준다면 완전 편할 것 같아 Athena와 QuickSight를 이용했다.

아마 내가 NoSql에 넣을 만큼 데이터를 직접 많이 모을 수 있는 방법은 별로 없을 것 같으니 내 노트북과 스마트폰 네트워크 트래픽을 분석 할 수 있는 서비스를 만든다던가 해봐도 괜찮을 것 같다.(떡밥)

---

### 구성 하기

자세한 서비스 구축방법은 나보다 [이 아조씨](https://www.youtube.com/watch?v=mxLm-mC-n54)가 더 친절이 잘 설명 해주신다. 저는 간단하게 소개만 하는 거라..

![](http://drive.google.com/uc?export=view&id=1iSLLGQ_3H44xlljEHKRNmKVf86bRwNh-)

먼저 `S3`에 데이터 셋을 업로드 해준다. 원래 데이터는 business.json, review.json, user.json 등 몇개의 json파일이 있다. 

이때 각 파일별로 폴더를 구성해서 만들어준다.

Athena에서 S3의 데이터를 불러올 때 트래픽을 최소화 하게 하려면 필요한 데이터만 스캔하게 해야 한다.

그래서 로그 데이터를 Athena를 이용해 분석할 때 날자별로 폴더를 만들어서 불러오게 한다거나 하는 방식을 이용 한다고도 한다.

![](http://drive.google.com/uc?export=view&id=1LyrsYYLzgYC7u1FK6r7_iZ5NjAWZ-ek3)

이후 `Athena`에서 데이터베이스를 만들어준다. 처음 써보는거라 S3 주소를 넣으라길래 어디 있는지 한참 찾았는데 's3://' + 'bucket 이름/' + '폴더명/' 이렇게 써주면 된다.

&nbsp;

**QuickSight를 사용할 꺼라면 반드시 Athena를 QuickSight를 사용하는 리젼으로 맞춰야한다.** 

안그러면 QuickSight에서 데이터를 불러오지 못한다. 권한을 다 줬는데도 Athena에 접근을 못 하길래 한참 삽질했었다.. 한 시간 정도...

![](http://drive.google.com/uc?export=view&id=1sI4Qc6XOEPKqAP7cMnF6B9WJF3JKLyQg)

그렇게 하다보면 컬럼을 지정해야 한다고 한다. 뭐야 결국 스키마 다 지정해줘야하고 RDBMS 쓰는 것 만큼 귀찮잔아 라고 생각했지만 이미 물릴 수 없기 때문에 울며 겨자 먹기로 컬럼을 일일히 다 지정해줬다 🤯

&nbsp;

그리고 후에 Athena 리젼을 잘못 설정 했다는 걸 알게 된 후에 다시 할 때 `Glue` 라는게 있다는 걸 알게 됐다.

이 서비스는 S3에 저장된 데이터를 Athena에서 쓸 수 있게끔 크롤러를 돌려서 알아서 테이블을 만들어준다.

![](http://drive.google.com/uc?export=view&id=16ZQ2dXVNjVfnSdmbdcKdBwqi3isZkEJq)

사람들이 왜 AWS, AWS하는지 알것같은 부분이였다.

또 Glue를 써야 좋은점은 JSON을 컬럼 형식으로 변환시켜 모든 컬럼에 대한 풀 스캔을 피하게 할 수 있다.

이렇게 될 경우 JSON 스키마가 바뀌더라도 Athena 스키마를 따로 변경 시켜줄 필요없이 Glue가 알아서

스키마를 만들어 준다는 이점도 존재한다.

![](http://drive.google.com/uc?export=view&id=1qPKDwIpBYHEKehUWaEfxDyvXJAQBShEt)

![](http://drive.google.com/uc?export=view&id=1dfN5oB2301ULoybU2wd9e0XlFOLDcxkq)

데이터베이스를 만들고 나면 SQL을 이용해서 조회할 수 있다. 몇 초가 걸리는지 얼마나 데이터를 스캔했는지를 다 보여준다(다 돈이야 돈). 확실히 Standard SQL을 쓰기 때문에 쿼리문 작성은 매우 편했다.

이렇게 간단하게 데이터에서 리뷰 수 50 이상인 사업장 중 평균 별점이 높은 집들을 찾아볼 수 있다.

이제 마지막으로 `QuickSight` 를 이용해서 시각화만 하면 된다.

![](http://drive.google.com/uc?export=view&id=19AKQLa424jS4Q6m6P9Bs0nNyBKOXMR_D)

QuickSight는 매우 간단한 UI로 구성돼 있다. 

원하는 차트 모양을 선택하고 x, y축에 해당하는 컬럼을 선택해 그래프를 바로 뽑아준다.

![](http://drive.google.com/uc?export=view&id=1qXr6i7Cjyxz1-wucBK9tya83KR621S1y)

그래프 종류도 꽤 많아서 적절한걸 고르면 편하게 분석 할 수 있을 것 같다는 생각이 들었다.

---

### 후기

이 과정은 AWS 서비스를 처음 써보낸 내가 총 3시간 가량 걸렸다.

물론 내가 AWS를 전에도 써봤었다면 아마 한 시간도 안 걸렸을 것 같이 쓰기 쉬운 구조였다.

왜 많은 곳에서 AWS기반 서비스를 택하는지 알 수 있을것 같다..

&nbsp;

QuickSight는 정말 간단한 시각화에 쓸만할것 같고, 눈이 조금 더 가는건 Athena다. 

S3에 로그 백업 해놓고 추가적으로 더 설정할 것 없이 분석 하거나 실 서비스 DB에 부담 주지 않고

분석하거나, DB에 넣기 전 전처리용으로 등등.. 활용처가 많은 것 같다.

평소 쓰던 기술들만 쓰곤 했었는데 가끔 이렇게 다른걸 써보는것도 꽤 재미있었다.

---

### 광고

혹시 회사에 보충역 산업기능요원이 필요하신가요? 2년간 저와 함께 성장할 회사를 찾습니다! 

자동화, 백엔드 쪽에 관심이 많고 여러 분야에 대해 공부 중입니다.

 &nbsp; 

여러 분야를 경험 해 보면서 언어나 프레임워크 상관 없이 빠르게 배울 수 있습니다.

제 [이력서](https://jen6.github.io/move/resume)를 읽어보시고 관심이 생기신다면 work.jen6@gmail.com 으로 연락주세요 😁

[https://jen6.github.io/move/resume](https://jen6.github.io/move/resume)

---

### 참고자료

**[버즈빌의 누구나 궁금해하는 개발 이야기] 데이터 파이프라인(pipes data) 구축 방법**

[https://www.mobiinside.com/kr/2018/08/01/buzzvill-pipesdata/](https://www.mobiinside.com/kr/2018/08/01/buzzvill-pipesdata/)

**[뱅크샐러드]Analyze Data in MongoDB with AWS**

[https://medium.com/rainist-engineering/analyze-data-in-mongodb-with-aws-43c25ef0592f](https://medium.com/rainist-engineering/analyze-data-in-mongodb-with-aws-43c25ef0592f)

**Amazon Athena 및 Amazon QuickSight를 활용한 2백년간 글로벌 기후 데이터 시각화**

[https://aws.amazon.com/ko/blogs/korea/visualize-over-200-years-of-global-climate-data-using-amazon-athena-and-amazon-quicksight/](https://aws.amazon.com/ko/blogs/korea/visualize-over-200-years-of-global-climate-data-using-amazon-athena-and-amazon-quicksight/)

**DynamoDB에 대해서 알아보자 - 1**

[https://velog.io/@drakejin/DynamoDB에-대해서-알아보자-1](https://velog.io/@drakejin/DynamoDB%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-1)
