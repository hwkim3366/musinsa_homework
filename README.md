<div align="center">
<h1>카카오페이 서버개발 과제(부동산/신용 투자 서비스)</h1>
</div>

## 4.핵심 전략

    - `대용량 트래픽에 의한 DB(H2) 부하 감소 전략으로 내장 redis 사용(공지사항 내용 캐싱 및 조회수의 특정 배수일때만 DB 갱신 (10회))`


## 5. 공지 생성

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

## 7. 공지 조회
    - `GET`
    - `localhost:8080/notice/{noticeId}`
    - `조회할때 마다 view count 증가`

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
