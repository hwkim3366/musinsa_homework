<div align="center">
<h1>카카오페이 서버개발 과제(부동산/신용 투자 서비스)</h1>
</div>


## 1. Swagger api문서

    http://localhost/swagger-ui/


## 2. 개발 환경

    JDK : 1.8
    Framework : Spring boot 2.5.5
    Database : H2


## 3. 핵심 전략
    DB 관련
    - 투자 상품 마스터(PRODUCT)와 투자자별 투자 내역(INVEST_LIST) 테이블(엔티티)를 1:N 관계로 설계 생성
    - 어플리케이션 기동시 data.sql의 insert script로 투자 상품 마스터 테이블에 초기 데이터 4건 삽입(현재 일시 기준 유효 2건, 무효 2건)
    
    전체 투자 상품 조회
    - 전체 투자 상품 조회는 모집 기간에 현재 일시(조회 요청 일시)가 포함되는 전체 투자 상품을 조회
    
    투자하기
    - 투자 요청 시 총 투자모집 금액을 넘어 가지 않거나 딱 맞춘 경우 투자 내역 데이터를 생성한 뒤 투자 상품 마스터 데이터에 투자 내역을 반영
      (총 투자금액 누적, 투자자 누적, 모집 마감 여부)
    - 투자 요청 시 총 투자모집 금액에 딱 맞춘 경우 Sold out 메시지 리턴
    - 투자 요청 시점에 모집 완료는 아니나 성공 처리 시에 총 투자모집 금액을 넘어 갈 경우 투자 금액 한도 초과 메시지를 리턴하고
       투자 실패 처리하여 총 투자모집 금액까지 남은 잔액 한도 내에서 투자하게끔 유도
    - 투자 요청 시 기 마감된 투자 상품인 경우 Sold out 메시지 리턴하고 투자 실패 처리
    - 투자 요청 처리시 멀티 인스턴스 다량 트래픽 환경에서도 데이터 정합성을 유지하기 위해 트랜잭션 격리수준을 REPEATABLE_READ 로 설정
    
    나의 투자 상품 조회
    - `나의 투자 상품 조회는 유저의 ID값을 받아 해당 유저의 전체 투자 내역 내림차순 조회


## 4. 주요 메시지

    message(전체 처리 결과 메시지)
	- Processing completed : 처리 완료

    investStatus(투자 모집 상태)
	- Recruitment : 모집중
	- Recruitment completed : 모집 완료

    resultMsg(투자 처리 결과 메시지)
	- Sold out : 마감
	- Done, more investment is possible : 완료, 추가 투자 가능
	- Investment limit exceeded : 모집 금액 초과
	
    
## 5. 전체 투자 상품 조회 Sample

    `Request`
	Method : GET
	URI : localhost/productList

    `Response case 1` 현재 일시 기준 유효한 2건 모집중 상태로 조회
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": [
	    {
	      "productId": 1,
	      "title": "개인 신용 포트폴리오",
	      "totalInvestingAmount": 100,
	      "currInvestingAmount": 0,
	      "investerCount": 0,
	      "investStatus": "Recruitment", --모집중
	      "startedAt": "2022-02-01 11:11:11.111",
	      "finishedAt": "2022-12-31 22:22:22.222"
	    },
	    {
	      "productId": 2,
	      "title": "부동산 포트폴리오",
	      "totalInvestingAmount": 50,
	      "currInvestingAmount": 0,
	      "investerCount": 0,
	      "investStatus": "Recruitment", --모집중
	      "startedAt": "2022-01-01 00:00:00",
	      "finishedAt": "2022-11-01 00:00:00"
	    }
	  ]
	}

    `Response case 2` 현재 일시 기준 유효한 2건중 한건은 모집 완료 상태로 조회
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": [
	    {
	      "productId": 1,
	      "title": "개인 신용 포트폴리오",
	      "totalInvestingAmount": 100,
	      "currInvestingAmount": 100,
	      "investerCount": 2,
	      "investStatus": "Recruitment completed", --모집 완료
	      "startedAt": "2022-02-01 11:11:11.111",
	      "finishedAt": "2022-12-31 22:22:22.222"
	    },
	    {
	      "productId": 2,
	      "title": "부동산 포트폴리오",
	      "totalInvestingAmount": 50,
	      "currInvestingAmount": 30,
	      "investerCount": 1,
	      "investStatus": "Recruitment", --모집중
	      "startedAt": "2022-01-01 00:00:00",
	      "finishedAt": "2022-11-01 00:00:00"
	    }
	  ]
	}


## 6. 투자하기 Sample

    `Request`
	Method : POST
	URI : localhost/invest
	Header x-user-id : 11
	Body :
	{
	  "amount": 10,
	  "productId": 1
	}

    `Response case 1` 투자 성공 한뒤 모집 금액 한도가 남은 경우
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": {
	    "resultStatus": "SUCCESS",
	    "resultMsg": "Done, more investment is possible", --완료, 추가 투자 가능
	    "investAmount": 10
	  }
	}

    `Response case 2` 투자 성공 한뒤 모집 마감 되는 경우
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": {
	    "resultStatus": "SUCCESS",
	    "resultMsg": "Sold out", --마감
	    "investAmount": 90
	  }
	}

    `Response case 3` 투자 실패 처리(모집 완료는 아니나 총 모집 금액을 초과해서 투자하려 한 경우)
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": {
	    "resultStatus": "FAIL",
	    "resultMsg": "Investment limit exceeded", --모집 금액 초과
	    "investAmount": 0
	  }
	}

    `Response case 4` 투자 실패 처리(모집 완료 후 투자하려 한 경우)
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": {
	    "resultStatus": "FAIL",
	    "resultMsg": "Sold out", --마감
	    "investAmount": 0
	  }
	}


## 7. 나의 투자 상품 조회 Sample

    `Request`
	Method : GET
	URI : localhost/investList
	Header x-user-id : 11

    `Response case 1` 특정 유저 아이디로 투자한 내역 전체 내림차순 조회
	{
	  "code": "00",
	  "message": "Processing completed",
	  "body": [
	    {
	      "investId": 3,
	      "userId": "11",
	      "productId": 2,
	      "amount": 30,
	      "investingAt": "2022-03-14 00:14:18.152",
	      "title": "부동산 포트폴리오",
	      "totalInvestingAmount": 50
	    },
	    {
	      "investId": 2,
	      "userId": "11",
	      "productId": 1,
	      "amount": 90,
	      "investingAt": "2022-03-14 00:10:06.719",
	      "title": "개인 신용 포트폴리오",
	      "totalInvestingAmount": 100
	    },
	    {
	      "investId": 1,
	      "userId": "11",
	      "productId": 1,
	      "amount": 10,
	      "investingAt": "2022-03-13 23:59:10.882",
	      "title": "개인 신용 포트폴리오",
	      "totalInvestingAmount": 100
	    }
	  ]
	}
