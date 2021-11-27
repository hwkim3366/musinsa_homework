#개발 프레임워크
JDK : jdk1.8.0_221
IDE : sts-4.8.1.RELEASE (Spring Boot)
DB : H2 (memory DB)

H2 DB console 접속정보
URL : http://localhost:8080/h2-console 
(JDBC URL 설정: jdbc:h2:mem:testdb)

테스트용 클라이언트 툴 : Postman


#핵심 문제해결 전략
[테이블 정보]
증서번호(KEY) 테이블
STOCK

증서번호(KEY) 테이블 컬럼
STOCK.STOCK_ID : 증서번호(PK)
STOCK.ISSUED : 발급여부(Y/N)

문의접수번호(KEY) 테이블
RECEIPT

문의접수번호(KEY) 테이블 컬럼
RECEIPT.ID : 문의접수번호 (PK)
RECEIPT.ISSUED : 발급여부(Y/N)


controller의 method는 모두 syncronized 로 동기화 처리

1. KEY 등록 API
-POST Method로 들어욘 요청의 HTTP BODY에서 key, description, generator, min_length를 받아 증서번호 또는 문의접수번호 키를 생성
-식별이 용이하게 보험증서 번호는 무조건 숫자형, 문의접수번호는 무조건 문자형으로 구분 정의
-숫자형은 최소 자리수를 먼저 채우는 식으로 키 생성(max length 무제한 rule 감안) 
-숫자형은 생성시 생성 일시(yyyyMMddHHmmssSSS) + Thread의 숫자부분을 더하여 unique하게 처리
-숫자형 MysqlKeyGenerator 방식은 환경 문제 상 MY SQL DB 설치 실패로 인해 별도 쿼리 활용 없이 prefix로 구분하는 방법으로 구현
-문자형은 pattern과 length 로 유효성 및 중복 체크 후 저장
-최초 키 생성시 발급여부는 'N'으로 DB에 저장
-처리결과 상태값 전송

2. KEY 발급(증서번호, 문의접수번호) API
-증서번호(KEY) 테이블에서 발급여부 'N'인 키값을 추출 후 발급여부 상태값 'Y'값 변경 한뒤 키값 전달
-발급할 KEY가 없는 경우 키가 없다는 메시지 처리

3.다수의 서버 다수의 인스턴스에서 동작 가능하게 처리
-쓰레드 충돌 방지를 위해 컨트롤러의 각 메서드 synchronized 처리

5.Junit 테스트 파일
-TestAbstract.java : 테스트용 abstract 클래스
-ApiTest.java : 3가지 타입 키 생성 맟 업무 구분별 2가지 타입 키 발급 테스트


6.EXCEPTION 처리
-AleadyExistKeyException : 이미 존재하는 키값입니다.
-KeyNotFoundException : 발급 가능한 KEY가 없습니다.
-KeyTypeValidException : 올바른 key 생성 타입을 입력 바랍니다.
-KeyValidException : KEY값 입력 형식이 잘못되었습니다.
-MinLengthException : 올바른 최소자릿수를 입력 바랍니다.


7.Response Code 정의
-00 : Success 성공
-01 : Required input value is missing 필수입력값 누락
-02 : Bad Request 유효하지 않은 요청
-03 : Forbidden 권한 없음
-04 : Not Found 찾을수 없음





#어플리케이션 실행 방법
첨부한 jar 파일을 특정 경로에 두고 해당 경로로 가서 아래 커맨드 실행
java -jar kpay-spread-0.0.1-SNAPSHOT.jar


#API Call Sample
1. KEY 등록 (generic 보험증서 숫자형)
Method : POST
URL : http://localhost:8080/api/key/register

[입력]

BODY
{
   "key": "policy-number",
   "type" : "number",
   "generator" : "generic",
   "min_length" : "10"
}

[출력]
{
    "code": "00",
    "message": "Success",
    "body": null
}


2. KEY 등록 (mysql 보험증서 숫자형)
Method : POST
URL : http://localhost:8080/api/key/register

[입력]

BODY
{
   "key": "policy-number",
   "generator" : "mysql",
   "min_length" : "10"
}

[출력]
{
    "code": "00",
    "message": "Success",
    "body": null
}

3. KEY 등록 (문의접수번호 문자형)
Method : POST
URL : http://localhost:8080/api/key/register

[입력]

BODY
{
   "key": "claim-number",
   "description": "AAAA-9999-0000-1ABC"
}

[출력]
{
    "code": "00",
    "message": "Success",
    "body": null
}


4. KEY 발급 (보험증서 번호)
Method : GET
URL : http://localhost:8080/api/key/policy-number

[출력]
{
    "value": "10000000002021052023205536680805"
}

5. KEY 발급 (문의접수 번호)
Method : GET
URL : http://localhost:8080/api/key/claim-number

[출력]
{
    "value": "AAAA-9999-0000-1ABC"
}



P.S 개인 사정으로 시간에 쫓겨 mysql 설치 환경 구축을 못한것과 테스트 케이스를 충분히 만들지 못한게 많이 아쉽습니다 확인 검토 해주셔서 감사합니다
