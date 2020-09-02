

# 자전거렌트 서비스 

본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다.
이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# 구현 Repository
- https://github.com/JuneBeomKim/b-rental
- https://github.com/JuneBeomKim/b-bike
- https://github.com/JuneBeomKim/b-user
- https://github.com/JuneBeomKim/b-voucher
- https://github.com/JuneBeomKim/b-myPage
- https://github.com/JuneBeomKim/b-gateway

# Table of contents

- [자전거렌트 서비스](#---)
  - [서비스 시나리오](#서비스-시나리오)
  - [체크포인트](#체크포인트)
  - [분석/설계](#분석설계)
    - [이벤트스토밍](#이벤트스토밍)
    - [도메인분리](#도메인분리)
    - [비동기호출과 이벤트드리븐](#비동기호출과이벤트드리븐)
    - [장애격리](#장애격리)
    - [열려있는 아키텍쳐](#열려있는아키텍쳐)
    - [헥사고날](#헥사고날)    
  - [구현:](#구현-)   
    - [구현체개발](#구현체개발)
    - [유비쿼터스랭귀지](#유비쿼터스랭귀지)
    - [게이트웨이와인증](#게이트웨이와인증)
    - [비동기호출과 이벤트드리븐](#비동기호출과이벤트드리븐)
  - [운영](#운영)
    - [무정지 재배포](#무정지-재배포)
    - [오토스케일 아웃](#오토스케일-아웃)
  - [신규 개발 조직의 추가](#신규-개발-조직의-추가)

# 서비스 시나리오

기능적 요구사항
1. 고객이 회원가입을 한다.
1. 고객이 개인정보를 변경한다.
1. 고객이 회원탈퇴를 한다.
1. 고객이 바우처를 구매한다. 
1. 고객이 자전거를 선택해서 렌트를 한다. 
1. 렌트 완료시 바우처는 줄어든다.
1. 렌트 완료시 자전거는 예약된다. 
1. 고객이 렌트를 취소할 수 있다. 
1. 관리자가 자전거의 상태를 변경할 수 있다.

비기능적 요구사항
1. 트랜잭션
    1. 바우처가 없는 고객은 렌트가 되지 않아야 한다. Sync 호출 
    1. 렌트된 자전거는 렌트가 되지 않아야 한다. Sync 호출 
1. 장애격리
    1. 즉시 렌트가 되지 않더라도, 렌트예약은 24시간 받을 수 있어야 한다.  Async (event-driven), Eventual Consistency
    1. 실제 자전거 대여시스템에 문제가 발생 시, 복구된 이후에 순차처리를 한다.  Circuit breaker, fallback
1. 성능
    1. 고객이 마이페이지를 통해 렌트한 자전거, 바우처정보, 개인정보를 조회할 수 있다.  CQRS


# 체크포인트

- 분석 설계
1. 이벤트스토밍 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍쳐와의 연계 설계에 적절히 반영하고 있는가?
2. 서브 도메인의 분리 : 팀별KPI와 관심사, 상이한 배포주기 요구에 따른 sub-domain이나 Bounded Context를 적절히 분리하였는가?
3. 업무 중요성과 도메인간 서열을 구분할 수 있는가? 
4. Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할수 있는가?
5. 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없ㅎ도록 설계(장애격리)시 추가 점수
6. 신규 서비스를 추가 하였을 때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열러있는 아키택쳐)할수 있는가?
7. View를 이용한 CQRS 에 의한 데이터 프로젝션을 설계할 수 있는가? 

- 구현
1. 분석단계에서의 스티커별 색상과 헥사조날 아키텍처에 따라 구현체가 매핑되게 개발하였는가?
2. 분석단계에서의 유비쿼터스랭퀴지를 사용하여 소스코드가 서술되었는가?
3. 게이트웨이를 통하여 마이크로서비스들의 집입점을 통일할 수 있는가?
4. 게이트웨이와 인증서버(OAuth), JWT토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
5. 서킷브레이커를 통하여 장애를 격리시킬 수 있는가?

- 운영
1. 무정지 재배포를 위한 Liveness, Readness PRobe의 설정 및 증명
2. 구현 오류나 API 계약위반의 차단을 위한 Contract Test가 수행되는가?
3. 설정의 외부 주입(configMap or Secret)을 통한 운영 유연성을 제공하는가?
4. 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높일 수 있는가?
5. 오토스케일러(HPA)를 설정하여 실제워키로드를 테스트 환경으로 생성하여 (seige or jMeter) Replicas의 증감을 보여줄 수 있는가?
6. 운영 장애상황을 모니터링 할 수 있는 대시보드를 구성할 수 있는가?

# 분석/설계


## AS-IS 조직 (Horizontally-Aligned)
  ![image](https://user-images.githubusercontent.com/487999/79684144-2a893200-826a-11ea-9a01-79927d3a0107.png)

## TO-BE 조직 (Vertically-Aligned)
![image](https://user-images.githubusercontent.com/19456350/87373254-4b8cc500-c5c4-11ea-8b81-deb2c576406b.png)



### 이벤트스토밍
* MSAEz 로 모델링한 이벤트스토밍 결과:  http://www.msaez.io/#/storming/rX6bwdUwpMRNxLWEawoeDIkETow2/every/45e4b40116aedb2ec6a451d83a14e752/-MG1NTXc1M6uWZGKTY8s


### 이벤트 스토밍 결과  

### 이벤트 도출 
![1번이요](https://user-images.githubusercontent.com/25577890/91825953-eaa27480-ec77-11ea-8e57-20960bcafe56.png)

### 부적격 이벤트 탈락
![2번](https://user-images.githubusercontent.com/25577890/91825955-ebd3a180-ec77-11ea-8071-70b258d7255a.png)

 - 대여시> 도서가 예약됨, 대여취소시>취소가능상태인지 확인됨메뉴검색됨 : 팀의 요구사항과 맞지 않는 업무이벤트이므로 제외함 

### 액터, 커맨드 부착하여 읽기 좋게
![3번](https://user-images.githubusercontent.com/25577890/91825956-ebd3a180-ec77-11ea-95b9-e24a07c2352f.png)


### 어그리게잇으로 묶기
![4번](https://user-images.githubusercontent.com/25577890/91825959-ec6c3800-ec77-11ea-81dd-083e06d00e88.png)

- app의 User, BookStore 의 대여, 결제 그와 연결된 command 와 event 들에 의하여 트랜잭션이 유지되어야 하는 단위로 그들 끼리 묶어줌

### 바운디드 컨텍스트로 묶기
![5번](https://user-images.githubusercontent.com/25577890/91825961-ec6c3800-ec77-11ea-83d0-bc66a3fd92f8.png)

### 도메인분리
- 핵심 업무인 자전거대여가 가장 많은 로직이 있으며, 배포주기도 자주 발생하는 요구사항에 따라 아래와 같이 도메인 서열을 구분지음
    - Core Domain:  RentalSystem(자전거대여) 
    - Supporting Domain: user(고객관리),  bike(자전거관리)
    - General Domain:  voucher(바우처관리)

### 폴리시 부착 

![6번](https://user-images.githubusercontent.com/25577890/91825963-ed04ce80-ec77-11ea-8786-b54e519e0ba8.png)


### 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)
![7번 컨텍매핑](https://user-images.githubusercontent.com/25577890/91825966-ed9d6500-ec77-11ea-8668-f31a4aad205f.png)


### 완성된 모형
![완성1](https://user-images.githubusercontent.com/25577890/91920135-1f0f4280-ed03-11ea-8a73-a0f939928ead.PNG)

### 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증
![완성1](https://user-images.githubusercontent.com/25577890/91920135-1f0f4280-ed03-11ea-8a73-a0f939928ead.PNG)

    - 고객이 고객정보를 등록한다 (ok)
    - 매니저가 자전거 정보를 등록한다(ok)
    - 고객이 바우처를 구매한다 (ok)
    - 자전거 대여 시 바우처가 사용된다 (ok)
    - 자전거 대여 시 자전거 상태가 사용불가로 변경된다 (ok)
    - 대여가 취소되면 바우처가 증가한다 (ok)
    - 대여가 취소되면 자전거 상태가 사용가능으로 변경된다 (ok)

### 비동기호출과이벤트드리븐
![완성2](https://user-images.githubusercontent.com/25577890/91922430-fab66480-ed08-11ea-8be4-54c63b25c2e5.PNG)

  1. 동기 호출 트랜잭션 처리
     - 바우처가 정상 처리될때만, 자전거가 사용가능할때에만 대여가 가능해야 한다는 요구사항
     -  Request-Response 방식 처리
  2. 비동기 호출 이벤트 드리븐
     - 대여의 취소 자체는, 추후에 재고 정리하는 방향으로 진행해도 된다는 요구사항
     - Pub-Sub
  3. 자전거가 대여취소 실패되었는데도 렌트를 못하게 되는, 유휴자원 최소화를 위한 Saga패턴 적용
 
### 장애격리
Core Domain인 자전거 렌탈 서비스는 고객, 자전거, 바우처 서비스와 독립적으로 24시간 대여주문을 받을 수 있도록 설계. suporting 서비스에 문제가 생기더라도 core 서비스는 영향을 받지 않도록 사이트카 설정.

### 열려있는아키텍쳐 
현재 서비스에 신규서비스를 추가하더라도 기존 서비스에는 영향이 없음

추가예정인 서비스 
1. 포인트 서비스 (고객에게 사용이력에 따라 포인트 제공)
2. 자전거 렌트 연장 서비스 (바우처를 사용하여 연장신청)
3. 자전거 리뷰 서비스 (자전거 별로 상태 등에 대한 만족도를 작성하고 공유하는 서비스)
4. 자동차 서비스 (자동차도 대여가 가능하도록 별도 관리하여 서비스)




### 헥사고날

![헥사고날](https://user-images.githubusercontent.com/25577890/91920130-1e76ac00-ed03-11ea-9716-3314eff78eb1.png)

    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함


# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 구현하였다. 

### 구현체개발
# github
![git](https://user-images.githubusercontent.com/25577890/91920437-005d7b80-ed04-11ea-9ef8-ee215209520a.PNG)
# kubectl get all
![최종kubectl](https://user-images.githubusercontent.com/25577890/91920370-cbe9bf80-ed03-11ea-8c9f-39c7a98bb227.PNG)


### 유비쿼터스랭귀지

실생활에도 많이 쓰는 영어단을 사용함으로써 모든 개발자들이 직관적으로 공통적으로 의미를 이해 할 수 있도록 하였다.
(bike, voucher, user 등)

### 게이트웨이와인증
![gw](https://user-images.githubusercontent.com/25577890/91920539-54686000-ed04-11ea-83cc-53677289d30c.PNG)
LoadBalancer type으로 서비스

### 동기호출과 이벤트드리븐
분석단계에서의 조건 중 하나로 렌트->bike관리 간의 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 

 - bike관리,바우처 서비스를 잠시 내려놓고, 렌탈을 받은 직후(@PostPersist) bike상태, 바우처 잔여갯수를 변경하도록 처리 -> 동기방식이므로 수신실패
 ![rental 동기오류](https://user-images.githubusercontent.com/25577890/91920825-056efa80-ed05-11ea-9a2f-e9d751fd36b4.PNG)
 
 -  bike관리,바우처 서비스를 올려놓고, 렌탈을 받은 직후(@PostPersist) bike상태, 바우처 잔여갯수를 변경 성공확인
![동기호출 성공](https://user-images.githubusercontent.com/25577890/91920831-06a02780-ed05-11ea-89ec-6b732507339c.PNG)
![동기호출 성공_kafka](https://user-images.githubusercontent.com/25577890/91920834-06a02780-ed05-11ea-9ddc-ba5188b1175b.PNG)
 


### 비동기호출과이벤트드리븐
렌트취소가 이루어진 후에 bike관리, 바우처시스템으로 이를 알려주는 행위는 동기식이 아니라 비 동기식으로 처리하여 렌탈시스템의 처리를 위하여 렌탈주문이 블로킹 되지 않아도록 처리한다.

 - 이를 위하여 렌탈에 렌탈상태를 남긴 후에 곧바로 렌탈취소 되었다는 도메인 이벤트를 카프카로 송출한다(Publish)
 ![비동기호출(bike svc 내린상태)](https://user-images.githubusercontent.com/25577890/91920836-0738be00-ed05-11ea-87e4-b0489a142f96.PNG)
 ![비동기호출_kafka(bike svc 내린상태)](https://user-images.githubusercontent.com/25577890/91920839-07d15480-ed05-11ea-9900-8bf31396667a.PNG)

 - bike, 바우처 서비스가 잠시 내려간 상태라도 렌탈취소를 받는데 문제가 없다.
![kubectl get all (bike서비스 후에 올린상태)](https://user-images.githubusercontent.com/25577890/91920823-04d66400-ed05-11ea-996c-b738656cda74.PNG)

 - bike서비스를 올린후, 카프카에서 bike서비스에 대한 비동기호출 메세지를 수신한 것을 볼 수 있다.
![비동기호출_kafka(bike svc 내렸다 올린상태)](https://user-images.githubusercontent.com/25577890/91920838-0738be00-ed05-11ea-8a64-3188e5e1b98b.PNG)


# 운영

### 서킷 브레이커 테스트
Bike 프로젝트의 사이드카 적용

-  서킷 브레이커 적용 - hysterix 적용
![히스테릭스](https://user-images.githubusercontent.com/25577890/91829110-fbed8000-ec7b-11ea-891c-0d625d5ac85b.png)

-  적용 후 Siege 시, 임계치 초과시 500에러 리턴 - 
![circuit](https://user-images.githubusercontent.com/25577890/91923330-0e62ca80-ed0b-11ea-9111-23c3e029a12b.PNG)


### 무정지 재배포
모든 프로젝트의 readiness probe 및 liveness probe 설정 완료
![readiness](https://user-images.githubusercontent.com/25577890/91814248-cd1cdd00-ec6e-11ea-894e-4f7790e5dbdb.png)




### 오토스케일 아웃
결제서비스에 대해 CPU 사용량이 15프로 넘어가면 replica를 늘려주도록 설정한다 
모니터링을통해 오토스케일 진행현황 확인 가능
![autoscale](https://user-images.githubusercontent.com/25577890/91814246-cbebb000-ec6e-11ea-8074-3a0157634495.PNG)

