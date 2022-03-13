<div align="center">
<h1>카카오페이 서버개발 과제(부동산/신용 투자 서비스)</h1>
</div>


## 1.swagger api문서

    - http://localhost/swagger-ui/


## 2.개발 환경

    - JDK : 1.8
    - Framework : Spring boot 2.5.5
    - Database : H2


## 3.핵심 전략

    - `투자 상품 마스터(PRODUCT)와 투자자별 투자 내역(INVEST_LIST) 테이블을 1:N 관계로 설계 생성`
    - `어플리케이션 기동시 data.sql의 insert script로 투자 상품 마스터 테이블에 초기 데이터 4건 삽입(현재 일시 기준 유효 2건, 무효 2건)`

    - `전체 투자 상품 조회는 모집 기간에 현재 일시(조회 요청 일시)가 포함되는 전체 투자 상품을 조회`

    - `투자 요청 시 총 투자모집 금액을 넘어 가지 않는 경우 투자 내역 데이터를 생성한 뒤 투자 상품 마스터 데이터에 투자 내역을 반영
      (총 투자금액 누적, 투자자 누적, 모집 마감 여부) `
    - `투자 요청 시 총 투자모집 금액에 딱 맞춘 경우 Sold out 메시지 리턴하고 투자 성공 처리`
    - `투자 요청 시 모집 완료는 아니나 투자 성공 처리 시 총 투자모집 금액을 넘어 갈 경우 투자 금액 한도 초과 메시지를 리턴하고
       투자 실패 처리하여 총 투자모집 금액까지 남은 잔액 한도 내에서 투자하게끔 유도`
    - `투자 요청 시 기 마감된 투자 상품인 경우 Sold out 메시지 리턴하고 투자 실패 처리`
    - `투자 요청 처리시 멀티 인스턴스 다량 트래픽 환경에서도 데이터 정합성을 유지하기 위해 트랜잭션 격리수준을 REPEATABLE_READ 로 설정`

    - `나의 투자 상품 조회는 유저의 ID값을 받아 해당 유저의 전체 투자 내역 조회`

    - `junit을 활용하여 총 8가지 케이스를 단위 테스트 수행`


## 4.주요 메시지

    investStatus(투자 모집 상태)
	- Recruitment : 모집중
	- Recruitment completed : 모집 완료

    resultMsg(투자 처리 결과 메시지)
	- Sold out : 마감
	- Done, more investment is possible : 완료, 추가 투자 가능
	- Investment limit exceeded : 모집 금액 초과


## 5. 전체 투자 상품 조회
    - Request 
	Method : GET
	URI : localhost/productList

    - Response
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
	      "investStatus": "Recruitment",
	      "startedAt": "2022-02-01 11:11:11.111",
	      "finishedAt": "2022-12-31 22:22:22.222"
	    },
	    {
	      "productId": 2,
	      "title": "부동산 포트폴리오",
	      "totalInvestingAmount": 50,
	      "currInvestingAmount": 0,
	      "investerCount": 0,
	      "investStatus": "Recruitment",
	      "startedAt": "2022-01-01 00:00:00",
	      "finishedAt": "2022-11-01 00:00:00"
	    }
	  ]
	}

## 2. 공지 생성

Method : POST
URL : http://localhost:8080/spreading

[입력]
HEADER
key/value : Content-Type / application/json
key/value : X-ROOM-ID / aa
key/value : X-USER-ID / 77

BODY
{
   "amount": "10000",
   "count": "3"
}

[출력]
{
    "code": "00",
    "message": "Success",
    "body": "GGM"
}

    - `POST`
    - `localhost:8080/notice`
    - `헤더 부분은 놔두고 Body의 form-data 영역만 기재`
    - `Body form-data 영역 KEY : 'json_str', VALUE : 하단 포맷 참조`

      {
        "title" : "공지제목",
	"author" : "작성자",
        "content" : "공지내용",
        "startTime" : "202112010000",
        "endTime" : "202112021000",
        "attachments" : [
          {
            "name" : "실제 파일명1"
          },
          {
            "name" : "실제 파일명2"
          }
        ]
      }

     - `Body 영역 KEY : 'file'로 기재하고 타입을 파일로 선택, VALUE : 실제 파일 선택하고 상단의 '실제 파일명'을 동일하게 맞출것, 복수 기재 가능`
     - `파일 저장 경로 d:\\temp\\spring_uploaded_files`


## 6. 첨부파일 다운로드
    - `GET`
    - `localhost:8080/notice/download?filename={실제 파일명}`
    - `파일 저장 경로 d:\\temp\\spring_uploaded_files`



## 8. 공지 수정
    - `PUT`
    - `localhost:8080/notice/{noticeId}`
    - `헤더 정보 - KEY : content-type, VALUE : application/json`
    - `Body Raw 영역 하단 포맷 참조`
	{
	  "title" : "공지제목 수정",
	  "content" : "공지내용 수정",
	  "author" : "작성자 수정",
	  "startTime" : "202212010000",
	  "endTime" : "202212021000"
	}


## 9. 첨부파일 id별 수정(교체)
    - `PUT`
    - `localhost:8080/notice/{noticeId}/attach/{attachmentId}`
    - `헤더 부분은 놔두고 Body의 form-data 영역만 기재`
    - `Body form-data 영역 KEY : 'json_str', VALUE : 하단 포맷 참조`
	{
	  "id" : 1,
	  "name" : "수정 첨부 파일명"
	}

     - `Body 영역 KEY : 'file'로 기재하고 타입을 파일로 선택, VALUE : 실제 파일 선택하고 상단의 '수정 첨부 파일명'을 동일하게 맞출것`
     - `파일 저장 경로 d:\\temp\\spring_uploaded_files`

## 10. 첨부파일 ID별 삭제
    - `DELETE`
    - `localhost:8080/notice/{noticeId}/attach/{attachmentId}`

## 11. 공지 전체 삭제
    - `DELETE`
    - `localhost:8080/notice/{noticeId}`
