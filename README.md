## 비즈뿌리오 메시지 HTTP API 연동 규격 v2.7



## 목차

1. #### [개요](#1-개요)

2. #### [선결 조건](#2-선결-조건)

3. #### [연동 규격](#3-연동-규격)

4. #### [API 도메인](#4-API-도메인)

5. #### [API 기능](#5-API-기능)

6. #### [API 요청](#6-API-요청)

7. #### [전송 결과 전달 (URL PUSH 방식)](#7-전송-결과-전달-(URL-PUSH 방식))

8. #### [API 응답 상태 및 결과 코드](#8-API-응답-상태-및-결과-코드)

9. #### [전송 결과 코드](#9-전송-결과-코드)

10. #### [연동 예제 코드 (메시지 전송)](#10-연동-예제-코드-(메시지-전송))

11. #### [개정 이력](#11-개정-이력)



### 1. 개요

- 본 문서는 비즈뿌리오 서비스의 기능을 HTTP 요청으로 활용할 수 있는 API 에 대한 설명입니다.



### 2. 선결 조건

- API를 사용하여 메시지 전송을 하기 위해서 아래의 조건이 선결되어야 합니다.

1. 서비스 계정 생성 및 사용 승인

   비즈뿌리오 (https://www.bizppurio.com) 에서 API 용도로 계정 생성

   - 운영과 검수 용도로 계정을 구분하여 생성

2. 전송 결과 수신 정보 등록 *5.4 전송 결과 전달 (URL PUSH 방식) 참조

   리포트 수신이 필요한 경우, 수신 받을 URL (IP/PORT)에 대한 정보 등록 요청

   - 443, 80을 제외한 포트를 사용해야 하는 경우, 별도로 방화벽 접근에 대한 허용 요청

3. 카카오톡 비즈메시지 사용하는 경우

   ① 카카오톡 채널 개설 및 비즈니스 채널 신청 (https://center-pf.kakao.com)

   ② 발신 프로필 키 생성 (https://www.bizppurio.com)

   ③ 알림톡 템플릿 등록/승인 및 친구톡 이미지 등록 (https://www.bizppurio.com)

4. RCS 사용하는 경우

   ① RCS 브랜드 개설 및 대행사 설정 (https://www.rcsbizcenter.com)

   ② RCS 브랜드 등록 (https://www.bizppurio.com)

   ③ RCS 발신번호 및 템플릿 등록/승인 (https://www.bizppurio.com)



### 3. 연동 규격

1. HTTPS

   API를 위한 모든 HTTP 요청의 경우 SSL 환경의 HTTPS(443) 를 이용합니다.

2. UTF-8

     UTF-8 인코딩을 기본으로 제공합니다.



### 4. API 도메인

| 환경 | 도메인                |
| ---- | --------------------- |
| 운영 | api.bizppurio.com     |
| 검수 | dev-api.bizppurio.com |



### 5. API 기능

| 경로        | 메소드 | 설명                                                   |
| ----------- | ------ | ------------------------------------------------------ |
| /v1/token   | POST   | 인증 토큰을 발급을 요청하는 기능입니다.                |
| /v3/message | POST   | 메시지 전송을 요청하는 기능입니다.                     |
| /v1/file    | POST   | MMS 발송에 사용될 이미지 파일을 업로드하는 기능입니다. |
| /v2/report  | POST   | 전송 결과 재 요청하는 기능입니다.                      |



### 6. API 요청

1. #### 인증 토큰 발급

   - API 서비스를 이용하기 위해서 인증 토큰을 발급하기 위한 기능입니다.
   - 인증 토큰의 유효 시간은 24시간이며, 이후에는 사용이 불가하고 재발급이 필요합니다.
   - Authorization 헤더에 비즈뿌리오 계정과 암호를 Base64 인코딩한 문자열을 입력합니다.

##### [Request]

###### 	A. Header

```http
POST /v1/token HTTP/1.1
Authorization: Basic {Base64(계정:암호)}
Content-type: application/json; charset=utf-8
```

##### [Response]

###### 	A. Header

```http
HTTP/1.1 200 OK
Content-type: application/json; charset=utf-8
```

###### 	B. Body

| 키          | 타입 | 설명           |
| ----------- | ---- | -------------- |
| accesstoken | text | 인증 토큰      |
| type        | text | Bearer         |
| expired     | text | 토큰 만료 시간 |

###### 	[예시]

```json
{
	"accesstoken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJ2ZwxvcGVydC5jb20iLCJleHAiOiIxNDg1MjcwMDAwMDAwIiwiaHR0cHM6Ly92ZwxvcGVydC5jb20vand0X2NsYWltcy9pc19hZG1pbiI6dHJ1ZswidXNlcklkIjoiMTEwMjgzNzM3MjcxMDIiLCJ1c2VybmFtZSI6InZlbG9wZXJ0In0.WE5fMufM0NDSVGJ8cAolXGkyB5RmYwCto1pQwDIqo2w",
	"type": "Bearer",
	"expired": "20201110185520"
}
```



2. #### 메시지 전송

##### [Request]

###### 	A. Header

```http
POST /v3/message HTTP/1.1
Content-type: application/json; charset=utf-8
Authorization: 인증 토큰 발급을 통해 받은 {type} + " " + {accesstoken}
```

###### 	[예시]

```http
POST /v3/message HTTP/1.1
Content-type: application/json; charset=utf-8
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJ2ZwxvcGVydC5jb20iLCJleHAiOiIxNDg1MjcwMDAwMDAwIiwiaHR0cHM6Ly92ZwxvcGVydC5jb20vand0X2NsYWltcy9pc19hZG1pbiI6dHJ1ZSwidXNlcklkIjoiMTEwMjgzNzM3MjcxMDIiLCJ1c2VybmFtZSI6InZlbG9wZXJ0In0.WE5fMufM0NDSVGJ8cAolXGkyB5RmYwCto1pQwDIqo2w
```

###### 	B. Body

* **공통** 

  | 키        | 타입 | 길이 | 필수 | 설명                                               |
  | --------- | ---- | ---- | ---- | -------------------------------------------------- |
  | account   | text | 20   | Y    | 비즈뿌리오 계정                                    |
  | type      | text | 3    | Y    | 메시지 데이터  ***SMS, LMS, MMS, AT, AI, FT, RCS** |
  | from      | text | 16   | Y    | 발신 번호                                          |
  | to        | text | 16   | Y    | 수신 번호                                          |
  | country   | text | 3    | N    | 국가 코드  ***국제 메시지 발송 참조**              |
  | content   | json | -    | Y    | 메시지 데이터  ***CONTENT 참조**                   |
  | refkey    | text | 32   | Y    | 고객사에서 부여한 키                               |
  | userinfo  | text | 50   | N    | 정산용 부서 코드                                   |
  | resend    | json | 3    | N    | 대체 전송 메시지 유형  ***RESEND 참조**            |
  | recontent | json | -    | N    | 대체 전송 메시지 데이터  ***CONTENT 참조**         |



* **CONTENT**

  * SMS

    | 키      | 타입 | 길이 | 필수 | 설명                              |
    | ------- | ---- | ---- | ---- | --------------------------------- |
    | message | text | -    | Y    | 내용 (EUC-KR 기준, 최대 90바이트) |

    ###### [예시]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "sms",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"sms": {
    			"message": "SMS 전송"
    		}
    	}
    }
    ```

    

  * LMS

    | 키      | 타입 | 길이 | 필수 | 설명                                |
    | ------- | ---- | ---- | ---- | ----------------------------------- |
    | message | text | -    | Y    | 내용 (EUC-KR 기준, 최대 2000바이트) |
    | subject | text | 64   | N    | 제목 (EUC-KR 기준, 최대 64바이트)   |

    ###### [예시]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "lms",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"lms": {
    			"subject": "제목",
    			"message": "LMS 전송"
    		}
    	}
    }
    ```

    

  * MMS

    | 키      | 타입 | 길이 | 필수 | 설명                                |
    | ------- | ---- | ---- | ---- | ----------------------------------- |
    | message | text | -    | N    | 내용 (EUC-KR 기준, 최대 2000바이트) |
    | file    | json | json | Y    | 첨부파일 (최대 3개)  ***FILE 참조** |
    | subject | text | 64   | N    | 제목 (EUC-KR 기준, 최대 64바이트)   |

    - FILE

      | 키   | 타입 | 길이 | 필수 | 설명            |
      | ---- | ---- | ---- | ---- | --------------- |
      | type | text | 3    | Y    | 파일 유형 (IMG) |
      | key  | text | 40   | Y    | 파일 키         |

    ###### [예시]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "mms",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"mms": {
    			"subject": "제목",
    			"message": "MMS 전송",
    			"file": [
    				{
    					"type": "IMG",
    					"key": "1585011852_DD7482861185100000001.jpg"
    				},
    				{
    					"type": "IMG",
    					"key": "1585011852_DD7482861185100000002.jpg"
    				},
    				{
    					"type": "IMG",
    					"key": "1585011852_DD7482861185100000003.jpg"
    				}
    			]
    		}
    	}
    }
    ```

    ​	

  * AT/AI/FT

    | 유형  | 키           | 타입 | 길이 | 필수 | 설명                                                         |
    | ----- | ------------ | ---- | ---- | ---- | ------------------------------------------------------------ |
    | at/ai | message      | text | -    | Y    | 내용 (한글/영문 1000자)                                      |
    |       | senderkey    | text | 40   | Y    | 발신 프로필 키                                               |
    |       | templatecode | text | 32   | Y    | 템플릿 코드                                                  |
    |       | button       | json | -    | -    | ***템플릿에 포함된 경우, 필수 입력**<br/>버튼 (최대 5개)  *** BUTTON 참조** |
    |       | quickreply   | json | -    | -    | ***템플릿에 포함된 경우, 필수 입력**<br/>바로 연결 버튼 (최대 10개)  ***QUICKREPLY 참조** |
    |       | title        | text | 50   | -    | ***템플릿에 포함된 경우, 필수 입력**<br/>템플릿 내용 중 강조 표기할 핵심 정보 |
    | ft    | message      | text | -    | Y    | 내용 (한글/영문 1000자)                                      |
    |       | senderkey    | text | 40   | Y    | 발신 프로필 키                                               |
    |       | button       | json | -    | N    | 버튼 (최대 5개) ***BUTTON 참조**                             |
    |       | image        | json | -    | N    | 이미지 (최대 1개) ***IMAGE 참조**                            |
    |       | userkey      | text | 30   | N    | 사용자 식별 키                                               |
    |       | adflag       | text | 1    | N    | 광고성 메시지 노출 여부 (Y/N, 기본 Y)                        |
    |       | wide         | text | 1    | N    | 와이드 이미지 사용 여부 (Y/N, 기본 N)                        |

    - BUTTON (AT/AI/FT 지원)

      | 키             | 타입 | 길이 | 필수 | 설명                                              |
      | -------------- | ---- | ---- | ---- | ------------------------------------------------- |
      | name           | text | 28   | Y    | 버튼 제목 ***AC 타입인 경우, '채널 추가'로 고정** |
      | type           | text | 2    | Y    | 버튼 타입 ***아래 버튼 타입 별 속성 표 참조**     |
      | url_pc         | text | -    | N    | PC 환경에서 이동할 URL                            |
      | url_mobile     | text | -    | N    | MOBILE 환경에서 이동할 URL                        |
      | scheme_ios     | text | -    | N    | iOS 환경, Application Custom Scheme               |
      | scheme_android | text | -    | N    | ANDROID 환경, Application Custom Scheme           |

      | 지원타입 | 버튼타입 | 속성           | 타입 | 필수 | 설명                                                         |
      | -------- | -------- | -------------- | ---- | ---- | ------------------------------------------------------------ |
      | at/ai/ft | WL       | url_pc         | text | N    | PC 환경에서 이동할 URL                                       |
      |          |          | url_mobile     | text | Y    | MOBILE 환경에서 이동할 URL                                   |
      |          | AL       | scheme_ios     | text | -    | ***scheme_ios, scheme_android, url_mobile 중 2 가지 필수 입력**<br/>mobile ios 환경에서 버튼 클릭 시 실행할 application custom scheme |
      |          |          | scheme_android | text | -    | mobile android 환경에서 버튼 클릭 시 실행할 application custom scheme |
      |          |          | url_pc         | text | N    | PC 환경에서 이동할 URL                                       |
      |          |          | url_mobile     | text | -    | MOBILE 환경에서 이동할 URL                                   |
      |          | BK       | -              | -    | -    | 해당 버튼 텍스트 전송                                        |
      |          | MD       | -              | -    | -    | 해당 버튼 텍스트 + 메시지 본문 전송                          |
      |          | BC       | -              | -    | -    | 상담톡을 이용하는 카카오톡 채널만 이용 가능                  |
      |          |          | chat_extra     | text | N    | 봇 전환 시 전달할 메타 정보                                  |
      |          | BT       | -              | -    | -    | 카카오 \| 오픈 빌더의 챗봇을 사용하는 카카오톡 채널만 이용 가능 |
      |          |          | chat_extra     | text | N    | 봇 전환 시 전달할 메타 정보                                  |
      |          |          | chat_event     | text | N    | 봇 전환 시 연결할 봇 이벤트명                                |
      | at/ai    | DS       | -              | -    | -    | 버튼 클릭 시 배송 조회 페이지로 이동                         |
      |          | AC       | -              | -    | -    | 버튼 클릭 시 카카오톡 채널 추가                              |
      |          | P1       | -              | -    | -    | 이미지 보안 전송 플러그인                                    |
      |          | P2       | -              | -    | -    | 개인정보이용 플러그인                                        |
      |          | P3       | -              | -    | -    | 원클릭 결제 플러그인<br/>(발송시 oneclick_id 또는 product_id를 필수로 전달해야 함) |

      

    - IMAGE (FT 지원)

      | 키       | 타입 | 길이 | 필수 | 설명                      |
      | -------- | ---- | ---- | ---- | ------------------------- |
      | img_url  | text | -    | Y    | 노출할 이미지             |
      | img_link | text | -    | N    | 이미지 클릭 시 이동할 URL |

    ###### 	

    * QUICKREPLY (AT/AI 지원)

      | 키             | 타입 | 길이 | 필수 | 설명                                               |
      | -------------- | ---- | ---- | ---- | -------------------------------------------------- |
      | name           | text | 28   | Y    | 바로 연결 제목                                     |
      | type           | text | 2    | Y    | 버튼 연결 타입 ***아래 버튼 타입 별 속성 표 참조** |
      | url_pc         | text | -    | N    | PC 환경에서 이동할 URL                             |
      | url_mobile     | text | -    | N    | MOBILE 환경에서 이동할 URL                         |
      | scheme_ios     | text | -    | N    | iOS 환경, Application Custom Scheme                |
      | scheme_android | text | -    | N    | ANDROID 환경, Application Custom Scheme            |
      | chat_extra     | text | -    | N    | 상담톡/봇 전환 시 전달할 메타 정보                 |
      | chat_event     | text | -    | N    | 봇 전환 시 연결한 봇 이벤트명                      |

      | 바로연결타입 | 속성           | 타입 | 필수 | 설명                                                         |
      | ------------ | -------------- | ---- | ---- | ------------------------------------------------------------ |
      | WL           | url_pc         | text | N    | PC 환경에서 이동할 URL                                       |
      |              | url_mobile     | text | Y    | MOBILE 환경에서 이동할 URL                                   |
      | AL           | scheme_ios     | text | -    | ***scheme_ios, scheme_android, url_mobile 중 2 가지 필수 입력**<br/>mobile ios 환경에서 버튼 클릭 시 실행할 application custom scheme |
      |              | scheme_android | text | -    | mobile android 환경에서 버튼 클릭 시 실행할 application custom scheme |
      |              | url_pc         | text | N    | PC 환경에서 이동할 URL                                       |
      |              | url_mobile     | text | -    | MOBILE 환경에서 이동할 URL                                   |
      | DS           | -              | -    | -    | 버튼 클릭 시 배송 조회 페이지로 이동                         |
      | BK           | -              | -    | -    | 해당 버튼 텍스트 전송                                        |
      | MD           | -              | -    | -    | 해당 버튼 텍스트 + 메시지 본문 전송                          |
      | BC           | -              | -    | -    | 상담톡을 이용하는 카카오톡 채널만 이용 가능                  |
      |              | chat_extra     | text | N    | 봇 전환 시 전달할 메타 정보                                  |
      | BT           | -              | -    | -    | 카카오 \| 오픈 빌더의 챗봇을 사용하는 카카오톡 채널만 이용 가능 |
      |              | chat_extra     | text | N    | 봇 전환 시 전달할 메타 정보                                  |
      |              | chat_event     | text | N    | 봇 전환 시 연결할 봇 이벤트명                                |
    
    ###### [알림톡(일반) 예시]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "at",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"at": {
    			"senderkey": "12345",
    			"templatecode": "template",
    			"message": "알림톡 + 버튼(WL)",
    			"button": [
    				{
    					"name": "웹 링크 버튼",
    					"type": "WL",
    					"url_pc": "http://www.bizppurio.com",
    					"url_mobile": "http://www.bizppurio.com"
    				}
    			]
    		}
    	}
    }
    ```
    
    ###### [알림톡 이미지 예시]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "ai",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"ai": {
    			"senderkey": "12345",
    			"templatecode": "template",
    			"message": "알림톡 이미지 + 버튼(WL)",
    			"button": [
    				{
    					"name": "웹 링크 버튼",
    					"type": "WL",
    					"url_pc": "http://www.bizppurio.com",
    					"url_mobile": "http://www.bizppurio.com"
    				}
    			]
    		}
    	}
    }
    ```
    
    ###### [친구톡 예시]
    
    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "ft",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"ft": {
    			"senderkey": "1234",
    			"message": "친구톡+버튼+이미지",
    			"adflag": "Y",
    			"button": [
    				{
    					"name": "1234567890123456789",
    					"type": "WL",
    					"url_pc": "http://www.bizppurio.com",
    					"url_mobile": "http://www.bizppurio.com"
    				},
    				{
    					"name": "1234567890123456789",
    					"type": "WL",
    					"url_pc": "http://www.bizppurio.com",
    					"url_mobile": "http://www.bizppurio.com",
    					"scheme_ios": "sceme://xxx.xxx",
    					"schemea_ndroid": "sceme://xxx.xxx"
    				}
    			],
    			"image": {
    				"img_url": "url",
    				"imglink": "http//message.com"
    			}
    		}
    	}
    }
    ```

  

  * RCS

    | 키            | 타입 | 길이 | 필수 | 설명                                                         |
    | ------------- | ---- | ---- | ---- | ------------------------------------------------------------ |
    | message       | json | -    | N    | 메시지 베이스에서 치환할 정보 ***MESSAGE 참조**              |
    | messagebaseid | text | 40   | Y    | RCS 공통 포맷 (RCS SMS, LMS, MMS) 또는 템플릿 ID ***공통 포맷 참조** |
    | chatbotid     | text | 40   | Y    | RCS 브랜드 포탈을 통해 생성한 챗봇 ID                        |
    | agencyid      | text | 20   | N    | 대행사 ID (기본 : daoutech)                                  |
    | header        | text | 1    | Y    | 메시지 상단에 식별 문구 입력 (0:Web 발신, 1:광고)            |
    | footer        | text | 64   | N    | 메시지 하단에 수신 거부 문구를 입력                          |
    | copyallowed   | text | 1    | N    | 단말기에서 사용자에게 ‘복사/공유’ 메뉴 보기 여부             |
    | button        | json | -    | N    | 메시지에 삽입할 버튼 정보 ***BUTTON 참조**                   |

    - MESSAGE

      | 키          | 타입 | 길이 | 필수 | 설명     |
    | ----------- | ---- | ---- | ---- | -------- |
      | title       | text | -    | N    | 제목     |
      | media       | text | -    | N    | 첨부파일 |
      | description | text | -    | N    | 내용     |

      

    - 공통포맷 (MESSAGEBASE ID)

      | MESSAGEBASE_ID | 상품 | 메시지 타입          | 카드장수 | 카드별 최대 버튼수 | 메시지 전체 최대 본문 길이 (글자수) |
    | -------------- | ---- | -------------------- | -------- | ------------------ | ----------------------------------- |
      | SS000000       | SMS  | Standalone           | 1        | 1                  | 100                                 |
      | SL000000       | LMS  | Standalone           | 1        | 3                  | 1300                                |
      | SMwThT00       | MMS  | Standalone Media Top | 1        | 2                  | 1300                                |
      | SMwThM00       | MMS  | Standalone Media Top | 1        | 2                  | 1300                                |
      | CMwMhM0300     | MMS  | Carousel Medium      | 3        | 2                  | 1300 (총합)                         |
      | CMwMhM0400     | MMS  | Carousel Medium      | 4        | 2                  | 1300 (총합)                         |
      | CMwMhM0500     | MMS  | Carousel Medium      | 5        | 2                  | 1300 (총합)                         |
      | CMwMhM0600     | MMS  | Carousel Medium      | 6        | 2                  | 1300 (총합)                         |
      | CMwShS0300     | MMS  | Carousel Small       | 3        | 2                  | 1300 (총합)                         |
      | CMwShS0400     | MMS  | Carousel Small       | 4        | 2                  | 1300 (총합)                         |
      | CMwShS0500     | MMS  | Carousel Small       | 5        | 2                  | 1300 (총합)                         |
      | CMwShS0600     | MMS  | Carousel Small       | 6        | 2                  | 1300 (총합)                         |

      ##### 글자수 및 라인수 정의

      ​	글자 수 : 1줄 당 정상적으로 표현가능한 글자 수, 한글 '가' 기준 측정

      ​	줄(라인) 수 : expand 없이 메시지 버블 최대크기에서 표현 가능한 description 줄 수

      - LMS (Standalone No media)

        ###### [글자 수]

        | 타이틀 | 디스크립션 | 버튼명 |
      | ------ | ---------- | ------ |
        | 16     | 18         | 17     |

        ###### [줄 수 (접혀있는 경우)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 | 버튼 3개 |
      | --------------------------- | -------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 28       | 26       | 24       | 22       |
        | **타이틀 1줄 + 디스크립션** | 27       | 25       | 23       | 20       |
        | **타이틀 2줄 + 디스크립션** | 26       | 23       | 21       | 19       |

      - MMS (Standalone Media Top - 세로형)

        ###### [글자 수]

        | 타이틀 | 디스크립션 | 버튼명 |
      | ------ | ---------- | ------ |
        | 16     | 18         | 17     |

        ###### [줄 수 (Media Tall인 경우, 접혀있는 경우)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 |
      | --------------------------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 9        | 8        | 6        |
        | **타이틀 1줄 + 디스크립션** | 8        | 6        | 4        |
        | **타이틀 2줄 + 디스크립션** | 7        | 5        | 3        |

        ###### [줄 수 (Media Medium인 경우, 접혀있는 경우)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 |
      | --------------------------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 15       | 13       | 11       |
        | **타이틀 1줄 + 디스크립션** | 14       | 12       | 10       |
        | **타이틀 2줄 + 디스크립션** | 13       | 11       | 9        |

      - MMS (Carousel Medium - 슬라이드형)

        ###### [글자 수]

        | 타이틀 | 디스크립션 | 버튼명 |
      | ------ | ---------- | ------ |
        | 13     | 14         | 13     |

        ###### [줄 수 (Media 없는 경우, RCS A2P 단말 기준)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 |
      | --------------------------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 28       | 26       | 23       |
        | **타이틀 1줄 + 디스크립션** | 27       | 25       | 23       |
        | **타이틀 2줄 + 디스크립션** | 26       | 23       | 21       |
        | **타이틀 3줄 + 디스크립션** | 24       | 22       | 20       |

        ###### [줄 수 (Media Medium인 경우, RCS A2P 단말 기준)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 |
      | --------------------------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 17       | 15       | 13       |
        | **타이틀 1줄 + 디스크립션** | 16       | 14       | 12       |
        | **타이틀 2줄 + 디스크립션** | 15       | 13       | 11       |
        | **타이틀 3줄 + 디스크립션** | 14       | 12       | 10       |

      - MMS (Carousel Small - 슬라이드형)

        ###### [글자 수]

        | 타이틀 | 디스크립션 | 버튼명 |
        | ------ | ---------- | ------ |
        | 5      | 6          | 5      |

        ###### [줄 수 (Media Short인 경우, RCS A2P 단말 기준)]

        |                             | 버튼 0개 | 버튼 1개 | 버튼 2개 |
        | --------------------------- | -------- | -------- | -------- |
        | **디스크립션 only**         | 20       | 18       | 16       |
        | **타이틀 1줄 + 디스크립션** | 19       | 17       | 15       |
        | **타이틀 2줄 + 디스크립션** | 18       | 16       | 14       |
        | **타이틀 3줄 + 디스크립션** | 17       | 15       | 13       |
        | **타이틀 4줄 + 디스크립션** | 16       | 14       | 12       |
        | **타이틀 5줄 + 디스크립션** | 15       | 13       | 11       |

      

    - BUTTON

      | 키          | 타입                   | 설명                  |
      | ----------- | ---------------------- | --------------------- |
      | suggestions | array of ‘suggestions’ | ***SUGGESTIONS 참조** |

      - SUGGESTIONS

        | 키     | 타입 | 설명              |
        | ------ | ---- | ----------------- |
        | Action | json | *** ACTION 참조** |

          - ACTION

            | 키                  | -                   | -           | -         | 타입 | 설명                          |
            | ------------------- | ------------------- | ----------- | --------- | ---- | ----------------------------- |
            | **displayText**     |                     |             |           | text |                               |
            | **urlAction**       |                     |             |           | json | **URL 연결하기**              |
            |                     | openUrl             |             |           | json |                               |
            |                     |                     | url         |           | text |                               |
            | **dialerAction**    |                     |             |           | json | **전화 연결하기**             |
            |                     | dialPhoneNumber     |             |           | json |                               |
            |                     |                     | phoneNumber |           | text |                               |
            | **mapAction**       |                     |             |           | json | **지도**                      |
            |                     | showLocation        |             |           | json | **지도 - 지도 보여주기**      |
            |                     |                     | location    |           | json |                               |
            |                     |                     |             | latitude  | text | 위도                          |
            |                     |                     |             | longitude | text | 경도                          |
            |                     |                     |             | label     | text | 표식                          |
            |                     |                     | fallbackUrl |           | text | 대체 URL                      |
            |                     | requestLocationPush |             |           | json | **지도 - 현재 위치 공유하기** |
            | **clipboardAction** |                     |             |           | json | **복사 하기**                 |
            |                     | copyToClipboard     |             |           | json |                               |
            |                     |                     | text        |           | text | 버튼 명 / 복사할 값           |
            | **composeAction**   |                     |             |           | json | **메시지 전송**               |
            |                     | composeTextMessage  |             |           | json |                               |
            |                     |                     | phoneNumber |           | text | 수신 번호                     |
            |                     |                     | text        |           | text | 메시지                        |
            | **calendarAction**  |                     |             |           | json | **캘린더 등록**               |
            |                     | createCalendarEvent |             |           | json |                               |
            |                     |                     | startTime   |           | text | 시작 시간                     |
            |                     |                     | endTime     |           | text | 종료 시간                     |
            |                     |                     | title       |           | text | 제목                          |
            |                     |                     | description |           | text | 설명                          |

    

    ###### [RCS 예시 - SMwThT00 (MMS Standalone Tall) 사용, 첨부파일 1개, 버튼 1개]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "rcs",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"rcs": {
    			"messagebaseid": "SMwThT00",
    			"chatbotid": "15880000",
    			"header": "1",
    			"footer": "무료 수신 거부 080-1234-5678",
    			"copyallowed": "Y",
    			"message": {
    				"description": "안녕하세요! MMS-StandaloneTall(SMwThT00)",
    				"media": "maapfile://BR.05NGK0A6dA.20200326140000.001"
    			},
    			"button": [
    				{
    					"suggestions": [
    						{
    							"action": {
    								"urlAction": {
    									"openUrl": {
    										"url": "https://www.google.com"
    									}
    								},
    								"displayText": "구글로 이동"
    							}
    						}
    					]
    				}
    			]
    		}
    	}
    }
    ```

    ###### [RCS 예시 - CMwMhM0300 (MMS Carousel Medium 3 장) 사용, 첨부파일 3개, 버튼 카드 별 2개]

    ```json
    {
    	"account": "test",
    	"refkey": "test1234",
    	"type": "rcs",
    	"from": "07000000000",
    	"to": "01000000000",
    	"content": {
    		"rcs": {
    			"messagebaseid": "CMwMhM0300",
    			"chatbotid": "15880000",
    			"header": "0",
    			"copyallowed": "Y",
    			"message": {
    				"title1": "첫번째",
    				"description1": "1번",
    				"media1": "maapfile://BR.05NGK0A6dA.20200326140000.001",
    				"title2": "두번째",
    				"description2": "2번",
    				"media2": "maapfile://BR.05NGK0A6dA.20200326140000.002",
    				"title3": "세번째",
    				"description3": "3번",
    				"media3": "maapfile://BR.05NGK0A6dA.20200326140000.003"
    			},
    			"button": [
    				{
    					"suggestions": [
    						{
    							"action": {
    								"urlAction": {
    									"openUrl": {
    										"url": "https://www.google.com"
    									}
    								},
    								"displayText": "URL 연결하기"
    							}
    						},
    						{
    							"action": {
    								"dialerAction": {
    									"dialPhoneNumber": {
    										"phoneNumber": "+1650253000"
    									}
    								},
    								"displayText": "전화 걸기"
    							}
    						}
    					]
    				},
    				{
    					"suggestions": [
    						{
    							"action": {
    								"composeAction": {
    									"composeTextMessage": {
    										"phoneNumber": "+1650253000",
    										"text": "Draft to go into the send message text field."
    									}
    								},
    								"displayText": "메시지 전송"
    							}
    						},
    						{
    							"action": {
    								"clipboardAction": {
    									"copyToClipboard": {
    										"text": "COUPONE-1234-1234"
    									}
    								},
    								"displayText": "복사하기"
    							}
    						}
    					]
    				},
    				{
    					"suggestions": [
    						{
    							"action": {
    								"calendarAction": {
    									"createCalendarEvent": {
    										"startTime": "2017-03-16T17:40:00.214+09:00",
    										"endTime": "2017-03-18T17:40:00.216+09:00",
    										"title": "Meeting",
    										"description": "GSG review meeting"
    									}
    								},
    								"displayText": "캘린더 등록"
    							}
    						},
    						{
    							"action": {
    								"mapAction": {
    									"showLocation": {
    										"location": {
    											"latitude": 37.4220041,
    											"longitude": -122.0862515,
    											"label": "Googleplex"
    										},
    										"fallbackUrl": "https://www.google.com/maps/@37.4219162,-22.078063,15z"
    									}
    								},
    								"displayText": "지도 보여주기"
    							}
    						}
    					]
    				}
    			]
    		}
    	}
    }
    ```

    

* **RESEND**

  메시지 전송이 실패한 경우, 대체 전송 설정

  | 키     | 타입 | 길이 | 필수 | 설명                                                      |
  | ------ | ---- | ---- | ---- | --------------------------------------------------------- |
  | first  | text | -    | N    | 1차 대체 전송 메시지 유형 ***SMS, LMS, MMS, AT, FT, RCS** |
  | second | text | -    | N    | 2차 대체 전송 메시지 유형 ***SMS, LMS, MMS**              |

  ###### [1차 대체 전송 예시 - AT 전송 실패하는 경우, SMS (1차) 대체 전송]

  ```json
  {
  	"account": "test",
  	"refkey": "test1234",
  	"type": "at",
  	"from": "07000000000",
  	"to": "01000000000",
  	"content": {
  		"at": {
  			"senderkey": "12345",
  			"templatecode": "template",
  			"message": "알림톡 + 버튼(WL)",
  			"button": [
  				{
  					"name": "웹 링크 버튼",
  					"type": "WL",
  					"url_mobile": "https: //www.daou.com",
  					"url_pc": "https://www.daou.com"
  				}
  			]
  		}
  	},
  	"resend": {
  		"first": "sms"
  	},
  	"recontent": {
  		"sms": {
  			"message": "SMS 대체 발송"
  		}
  	}
  }
  ```

  ###### [2차 대체 전송 예시 - RCS 전송 실패하는 경우, AT (1차) 대체 전송, AT (1차) 대체 전송 실패하는 경우, SMS (2차) 대체 전송]

  ```json
  {
  	"account": "test",
  	"refkey": "test1234",
  	"type": "rcs",
  	"from": "07000000000",
  	"to": "01000000000",
  	"content": {
  		"rcs": {
  			"messagebaseid": "SL000000",
  			"chatbotid": "15880000",
  			"header": "0",
  			"copyallowed": "Y",
  			"message": {
  				"title": "RCS LMS",
  				"description": "RCS 전송"
  			},
  			"button": [
  				{
  					"suggestions": [
  						{
  							"action": {
  								"mapAction": {
  									"requestLocationPush": {}
  								},
  								"displayText": "현재 위치 공유하기"
  							}
  						}
  					]
  				}
  			]
  		}
  	},
  	"resend": {
  		"first": "at",
  		"second": "sms"
  	},
  	"recontent": {
  		"at": {
  			"senderkey": "1234",
  			"templatecode": "template",
  			"message": "알림톡 전송",
  			"button": [
  				{
  					"name": "웹 링크 버튼",
  					"type": "WL",
  					"url_mobile": "https://www.daou.com"
  				}
  			]
  		},
  		"sms": {
  			"message": "SMS 대체 발송"
  		}
  	}
  }
  ```

  

* **국제 메시지 발송**

  해외수신자에게 국제 메시지를 발송하는 서비스

  | 메시지 유형 | 메시지 길이 제한                 |
  | ----------- | -------------------------------- |
  | sms         | 본문 최대 한글 50자, 영문 150자  |
  | lms         | 본문 최대 한글 150자, 영문 450자 |
  | at/ft       | 한글/영문 1000자                 |

  

  예시) 미국(1) 010-1234-1234로 전송하는 경우

  ###### [수신 번호에 국가 코드 포함 예시]

  | 지원 메시지 유형 |
  | ---------------- |
  | sms              |
  | lms              |

  ```
  [주요 파라미터 예시]
  "to" : "00211012341234"
  ```

  

  ###### [국가 코드 별도 입력 예시]

  | 지원 메시지 유형 | 대체 전송 메시지 유형 |
  | ---------------- | --------------------- |
  | sms              | -                     |
  | lms              | -                     |
  | at               | -                     |
  |                  | sms                   |
  |                  | lms                   |
  | ft               | -                     |
  |                  | sms                   |
  |                  | lms                   |

  ```
  [주요 파라미터 예시]
  "country" : "1",
  "to" : "01012341234"
  ```

  


##### [Response]

###### 	A. Header

```http
HTTP/1.1 200 OK
Content-type: application/json; charset=utf-8
```

###### 	B. Body

| 키          | 타입 | 길이 | 필수 | 설명                                               |
| ----------- | ---- | ---- | ---- | -------------------------------------------------- |
| code        | text | 5    | Y    | 결과 코드 ***8. API 응답 상태 및 결과 코드 참조**  |
| description | text | 32   | Y    | 결과 메시지                                        |
| messagekey  | text | 32   | Y    | 메시지 키 ***고객 문의 및 리포트 재 요청 기준 키** |
| refkey      | text | 32   | Y    | 고객사에서 부여한 키                               |

###### 	[예시]

```json
{
	"code": 1000,
	"description": "Success",
	"refkey": "123456789012345678901234890123",
	"messagekey": "190922175225820#ft002951servj8FU67"
}
```



3. #### MMS 파일 업로드

   - 파일을 업로드 하여 MMS 전송 시 사용할 파일 키를 발급합니다.
   - 파일은 확장자(jpg/jpeg), 크기(300kbyte 이하) 제한이 있습니다.
   - 요청 시 파일 업로드 수는 1개로 제한합니다.

##### [Request]

###### 	A. Header

```http
POST /v1/file HTTP/1.1
Content-Type: multipart/form-data; boundary=5d14-GC42dS9N5BXQAKuhpRfd4VDV54RDDsTJO4
```

###### 	B. Body

| 키      | 타입   | 길이 | 필수 | 설명            |
| ------- | ------ | ---- | ---- | --------------- |
| account | text   | 20   | N    | 비즈뿌리오 계정 |
| file    | binary | -    | Y    | 업로드할 파일   |

###### [예시]

```
--5d14-GC42dS9N5BXQAKuhpRfd4VDV54RDDsTJO4
Content-Disposition: form-data; name="account"
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit
{비즈뿌리오 계정}
--5d14-GC42dS9N5BXQAKuhpRfd4VDV54RDDsTJO4
Content-Disposition: form-data; name="file"; filename="1.jpg"
Content-Type: image/jpeg
Content-Transfer-Encoding: binary
<actual file content, not shown here>
--5d14-GC42dS9N5BXQAKuhpRfd4VDV54RDDsTJO4--
```



##### [Response]

###### 	A. Header

```http
HTTP/1.1 200 OK
Content-type: application/json
```

###### 	B. Body

- 성공

  | 키      | 타입 | 길이 | 필수 | 설명    |
  | ------- | ---- | ---- | ---- | ------- |
  | filekey | text | 40   | Y    | 파일 키 |

- 실패

  | 키          | 타입 | 길이 | 필수 | 설명                                              |
  | ----------- | ---- | ---- | ---- | ------------------------------------------------- |
  | code        | text | 5    | Y    | 결과 코드 ***8. API 응답 상태 및 결과 코드 참조** |
  | description | text | 32   | Y    | 결과 메시지                                       |

###### [예시]

```json
{
	"filekey": " 0920msg_123912934949595969"
}
```



4. #### 전송 결과 재 요청

##### [Request]

###### 	A. Header

```http
POST /v2/report HTTP/1.1
Content-type: application/json
Authorization: 인증 토큰 발급을 통해 받은 {type} + " " + {accesstoken}
```

###### 	B. Body

| 키         | 타입 | 길이 | 필수 | 설명            |
| ---------- | ---- | ---- | ---- | --------------- |
| account    | text | 20   | N    | 비즈뿌리오 계정 |
| messagekey | text | 32   | Y    | 메시지 키       |

###### [예시]

```json
{
	"account": "test",
	"messagekey": "190922175225820#ft002951servj8FU67"
}
```



##### [Response]

###### 	A. Header

```http
HTTP/1.1 200 OK
Content-type: application/json
```

###### 	B. Body

- 성공

  | 키      | 타입 | 길이 | 필수 | 설명    |
  | ------- | ---- | ---- | ---- | ------- |
  | filekey | text | 40   | Y    | 파일 키 |

- 실패

  | 키          | 타입 | 길이 | 필수 | 설명                               |
  | ----------- | ---- | ---- | ---- | ---------------------------------- |
  | code        | text | 5    | Y    | 결과 코드 (1000 : 성공, 이외 실패) |
  | description | text | 32   | Y    | 결과 메시지                        |

###### [예시]

```json
{
	"code": 1000,
	"description": "Success"
}
```



### 7. 전송 결과 전달 (URL PUSH 방식)

​	전송 결과는 고객사에서 사전에 등록 요청한 URL로 PUSH 방식으로 전달합니다.

##### [Request]

###### 	A. Header

```http
POST {고객사에서 등록 요청한 URL} HTTP/1.1
Content-type: application/json
```

###### 	B. Body

| 키          | 필수 | 설명                                                    |
| ----------- | ---- | ------------------------------------------------------- |
| DEVICE      | Y    | 메시지 유형                                             |
| CMSGID      | Y    | 메시지 키                                               |
| MSGID       | Y    | 비즈뿌리오 메시지 키                                    |
| PHONE       | Y    | 수신 번호                                               |
| MEDIA       | Y    | 실제 발송된 메시지 상세 유형 ***MEDIA 유형**            |
| TO_NAME     | N    | 수신자 명                                               |
| UNIXTME     | Y    | 발송 시간                                               |
| RESULT      | Y    | 이통사/카카오/RCS 결과 코드 ***9. 전송 결과 코드 참조** |
| USERDATA    | N    | 정산용 부서 코드                                        |
| WAPINFO     | N    | 이통사/카카오 정보 ***SKT/KTF/LGT/KAO**                 |
| TELRES      | N    | 대체 전송 결과                                          |
| TELTIME     | N    | 대체 전송 시간                                          |
| RETRY_FLAG  | N    | 대체 전송 정보                                          |
| RESEND_FLAG | N    | 대체 전송 메시지 유형                                   |

- MEDIA 유형

  | 분류 | 설명                      |
  | ---- | ------------------------- |
  | SMS  | 국내 SMS 발송             |
  | ISM  | 국제 SMS 발송             |
  | VDO  | 비디오 발송               |
  | MMS  | 이미지 발송               |
  | LMS  | 국내 LMS 발송             |
  | ILM  | 국제 LMS 발송             |
  | FSI  | 국제 FAX 발송             |
  | FSD  | 국내 FAX 발송             |
  | VMC  | 유선 PHONE 발송           |
  | VMW  | 무선 PHONE 발송           |
  | WAP  | WAP 발송                  |
  | KAT  | 알림톡 발송               |
  | KFT  | 친구톡 텍스트 발송        |
  | KFP  | 친구톡 이미지 발송        |
  | KFW  | 친구톡 와이드 이미지 발송 |
  | RSS  | RCS SMS 발송              |
  | RLS  | RCS LMS 발송              |
  | RMS  | RCS MMS 발송              |
  | RTS  | RCS TEMPLATE 발송         |

###### [예시]

```json
{
	"DEVICE": "SMS",
	"CMSGID": "201027134355944sms027420servqer0",
	"MSGID": "1027se_SL4676027383600490148",
	"PHONE": "01000000000",
	"MEDIA": "SMS",
	"UNIXTIME": "1603773837",
	"RESULT": "4100",
	"USERDATA": "daoutech",
	"WAPINFO": "SKT"
}
```



### 8. API 응답 상태 및 결과 코드

| 상태 코드 | 결과 코드 | 설명                                                         |
| --------- | --------- | ------------------------------------------------------------ |
| 200       | 1000      | 성공                                                         |
| 400       | 2000      | 메시지가 유효하지 않음                                       |
| 400       | 3000      | 비즈 뿌리오 계정에 접속 허용 아이피가 등록되어있지 않음      |
| 400       | 3001      | 인증 토큰 발급 호출 시, Basic Authentication 정보가 유효하지 않음 |
| 400       | 3002      | 토큰이 유효하지 않음.                                        |
| 400       | 3003      | 아이피가 유효하지 않음                                       |
| 400       | 3004      | 계정이 유효하지 않음                                         |
| 400       | 3005      | 인증 정보가 유효하지 않음 (bearer)                           |
| 400       | 3006      | 비즈뿌리오 계정이 존재하지 않음                              |
| 400       | 3007      | 비즈뿌리오 계정의 암호가 유효하지 않음                       |
| 400       | 3008      | 비즈뿌리오에 허용된 접속 수를 초과함                         |
| 400       | 3009      | 비즈뿌리오 계정이 중지 상태임                                |
| 400       | 3010      | 비즈뿌리오 계정에 등록된 접속 허용 IP와 일치하지 않음        |
| 400       | 3011      | 비즈뿌리오 내에서 알 수 없는 오류가 발생됨                   |
| 400       | 3012      | 비즈뿌리오에 존재하지 않은 메시지 (예: 보관 주기 35일이 지난 메시지) |
| 400       | 3013      | 완료 처리 되지 않은 메시지 (예: 통신사로부터 결과 미 수신)   |
| 400       | 5000      | 전송 결과 재 요청 실패                                       |
| 400       | 5001      | 요청한 URI 리소스가 존재하지 않음                            |
| 500       | 9000      | 알 수 없는 오류 발생.                                        |



### 9. 전송 결과 코드

- SMS

  | 코드 | 설명                                     |
  | ---- | ---------------------------------------- |
  | 4100 | 전달                                     |
  | 4400 | 음영 지역                                |
  | 4401 | 단말기 전원 꺼짐                         |
  | 4402 | 단말기 메시지 저장 초과                  |
  | 4403 | 메시지 삭제 됨                           |
  | 4404 | 가입자 위치 정보 없음                    |
  | 4405 | 단말기 BUSY                              |
  | 4410 | 잘못된 번호                              |
  | 4420 | 기타 에러                                |
  | 4430 | 스팸                                     |
  | 4431 | 발송 제한 수신거부(스팸)                 |
  | 4411 | NPDB에러                                 |
  | 4412 | 착신 거절                                |
  | 4413 | SMSC형식오류                             |
  | 4414 | 비 가입자, 결번, 서비스 정지             |
  | 4421 | 타임아웃                                 |
  | 4422 | 단말기일시정지                           |
  | 4423 | 단말기착신거부                           |
  | 4424 | URL SMS 미 지원 휴대폰                   |
  | 4425 | 단말기 호 처리 중                        |
  | 4426 | 재시도한도초과                           |
  | 4427 | 기타 단말기 문제                         |
  | 4428 | 시스템 에러                              |
  | 4432 | 회신 번호 차단(개인)                     |
  | 4433 | 회신 번호 차단(기업)                     |
  | 4434 | 회신 번호 사전 등록제에 의한 미등록 차단 |
  | 4435 | KISA 신고 스팸 회신 번호 차단            |
  | 4436 | 회신 번호 사전 등록제 번호 규칙 위반     |



- LMS/MMS

  | 코드 | 설명                                     |
  | ---- | ---------------------------------------- |
  | 6600 | 전달                                     |
  | 6601 | 타임 아웃                                |
  | 6602 | 핸드폰 호 처리 중                        |
  | 6603 | 음영 지역                                |
  | 6604 | 전원이 꺼져 있음                         |
  | 6605 | 메시지 저장 개수 초과                    |
  | 6606 | 잘못된 번호                              |
  | 6607 | 서비스 일시 정지                         |
  | 6608 | 기타 단말기 문제                         |
  | 6609 | 착신 거절                                |
  | 6610 | 기타 에러                                |
  | 6611 | 통신사의 SMC 형식 오류                   |
  | 6612 | 게이트웨이의 형식 오류                   |
  | 6613 | 서비스 불가 단말기                       |
  | 6614 | 핸드폰 호 불가 상태                      |
  | 6615 | SMC 운영자에 의해 삭제                   |
  | 6616 | 통신사의 메시지 큐 초과                  |
  | 6617 | 통신사의 스팸 처리                       |
  | 6618 | 공정위의 스팸 처리                       |
  | 6619 | 게이트웨이의 스팸 처리                   |
  | 6620 | 발송 건수 초과                           |
  | 6621 | 메시지의 길이 초과                       |
  | 6622 | 잘못된 번호 형식                         |
  | 6623 | 잘못된 데이터 형식                       |
  | 6624 | MMS 정보를 찾을 수 없음                  |
  | 6625 | NPDB에러                                 |
  | 6626 | 080 수신거부(SPAM)                       |
  | 6627 | 발송 제한 수신거부(SPAM)                 |
  | 6628 | 회신 번호 차단(개인)                     |
  | 6629 | 회신 번호 차단(기업)                     |
  | 6630 | 서비스 불가 번호                         |
  | 6631 | 회신 번호 사전 등록제에 의한 미등록 차단 |
  | 6632 | KISA 신고 스팸 회신 번호 차단            |
  | 6633 | 회신 번호 사전 등록제 번호 규칙 위반     |
  | 6670 | 첨부파일 사이즈 초과(60K)                |



- AT/FT

  | 코드 | 설명                                    |
  | ---- | --------------------------------------- |
  | 7000 | 전달                                    |
  | 7101 | 카카오 형식 오류                        |
  | 7103 | Sender key (발신프로필키) 유효하지 않음 |
  | 7105 | Sender key (발신프로필키) 존재하지 않음 |
  | 7106 | 삭제된 Sender key (발신프로필키)        |
  | 7107 | 차단 상태 Sender key (발신프로필키)     |
  | 7108 | 차단 상태 옐로우 아이디                 |
  | 7109 | 닫힌 상태 옐로우 아이디                 |
  | 7110 | 삭제된 옐로우 아이디                    |
  | 7203 | 친구톡 전송 시 친구 대상 아님           |
  | 7204 | 템플릿 불일치                           |
  | 7300 | 기타 에러                               |
  | 7305 | 성공 불확실(30일 이내 수신 가능)        |
  | 7306 | 카카오 시스템 오류                      |
  | 7308 | 전화번호 오류                           |
  | 7311 | 메시지가 존재하지 않음                  |
  | 7314 | 메시지 길이 초과                        |
  | 7315 | 템플릿 없음                             |
  | 7318 | 메시지를 전송할 수 없음                 |
  | 7322 | 메시지 발송 불가 시간                   |
  | 7323 | 메시지 그룹 정보를 찾을 수 없음         |
  | 7324 | 재전송 메시지 존재하지 않음             |
  | 7421 | 타임아웃                                |

  

- RCS

  | 코드 | 설명                                                         |
  | ---- | ------------------------------------------------------------ |
  | 8200 | 시스템 에러                                                  |
  | 8201 | 이미 발송한 메시지                                           |
  | 8202 | Message Convert 실패                                         |
  | 8203 | Message Validation 실패                                      |
  | 8204 | Message Send 실패                                            |
  | 8205 | MaaP FE Request 에러                                         |
  | 8206 | MaaP FE Response Status 에러                                 |
  | 8207 | MaaP FE API Response Convert 실패                            |
  | 8208 | RCS 메시지를 수신할 통신사가 없습니다.                       |
  | 8209 | Message Convert 실패                                         |
  | 8800 | Authorization 헤더 파라미터 누락                             |
  | 8801 | Authorization 헤더 값 누락                                   |
  | 8802 | 토큰이 일치하지 않습니다.                                    |
  | 8803 | 토큰이 만료되었습니다.                                       |
  | 8804 | 인증 토큰 에러                                               |
  | 8805 | 요청된 계정 정보를 찾을 수 없습니다(BP ID)                   |
  | 8806 | 요청된 중계사 전송 계정을 찾을 수 없습니다(RCS ID)           |
  | 8807 | 잘못된 패스워드                                              |
  | 8808 | 접근 허용된 IP가 아닙니다                                    |
  | 8809 | 메시지 전송을 할 수 없는 상태입니다. (서버의 요청 거부)      |
  | 8810 | RCS 메시지 TPS가 초과되었습니다.                             |
  | 8811 | RCS 메시지 Quota가 초과되었습니다.                           |
  | 8812 | 시스템 에러                                                  |
  | 8813 | IO 에러 발생                                                 |
  | 8814 | 중복 Key 오류                                                |
  | 8815 | 요청 파라미터 형식 오류                                      |
  | 8816 | 요청 Body JSON 파싱 에러                                     |
  | 8817 | 데이터를 찾을 수 없음                                        |
  | 8818 | 전화번호 형식이 일치하지 않습니다                            |
  | 8819 | 요청을 처리할 수 없는 상태입니다.                            |
  | 8820 | 이미 사용 중인 챗봇 ID입니다.                                |
  | 8821 | 챗봇을 생성할 수 없습니다.                                   |
  | 8822 | 챗봇 정보를 변경할 수 없습니다.                              |
  | 8823 | 챗봇이 있는 브랜드는 삭제 할 수 없습니다.                    |
  | 8824 | 챗봇 Type은 a2p, chatbot 로 설정해야 함                      |
  | 8825 | 요청 URL Parameter의 챗봇 Id와 Body Parameter 불일치         |
  | 8826 | 잘못된 26ebhook중계사 요청 파라미터 입니다.                  |
  | 8827 | Webhook 중계 시스템 연결 오류                                |
  | 8828 | 중계사 Webhook 전송 요청을 실패 했습니다.                    |
  | 8829 | 중계사 Webhook 처리 응답 수신 오류가 발생 했습니다.          |
  | 8830 | 요청을 처리할 수 없는 파일 유형입니다.                       |
  | 8831 | 파일 속성 오류                                               |
  | 8832 | fileID가 없거나 ID형식에 맞지 않음                           |
  | 8833 | File 저장 오류                                               |
  | 8834 | Multipart 데이터 전송 오류                                   |
  | 8835 | 자사 고객이 아닙니다.                                        |
  | 8836 | 자사 고객이지만, RCS메시지를 수신할 수 있는 가입자가 아닙니다. |
  | 8837 | 단말 기기로 RCS 메시지를 전송할 수 없습니다.                 |
  | 8838 | 내부 서버 오류가 발생하였습니다.                             |
  | 8839 | 기업 정보 내용이 누락된 필수 항목이 있습니다.                |
  | 8840 | 대행사 정보 내용이 누락된 필수 항목이 있습니다.              |
  | 8841 | AgencyID가 존재하지 않습니다.                                |
  | 8842 | BrandID에 대행 권한이 없는 AgencyID                          |
  | 8843 | 계약 정보 내용이 부정확하거나 누락된 필수 항목이 있습니다.   |
  | 8844 | 브랜드 정보 내용이 누락된 필수 항목이 있습니다.              |
  | 8845 | 브랜드 명이 누락되어 있습니다.                               |
  | 8846 | 브랜드 프로필 이미지가 누락되어 있습니다.                    |
  | 8847 | 브랜드 CS번호가 누락되어 있습니다.                           |
  | 8848 | 브랜드 메뉴 최대 개수를 초과하였거나 부정확합니다.           |
  | 8849 | 브랜드 카테고리 설정이 잘못되어 있습니다.                    |
  | 8850 | 브랜드 홈페이지 설정이 잘못되어 있습니다.                    |
  | 8851 | 브랜드 이메일 설정이 잘못되어 있습니다.                      |
  | 8852 | 브랜드 주소가 잘못되어 있습니다.                             |
  | 8853 | 브랜드ID가 존재하지 않음                                     |
  | 8854 | 챗봇 정보 내용이 부정확하거나 누락된 필수 항목이 있습니다.   |
  | 8855 | Booted(발신번호)가 전화번호 형식에 맞지 않음                 |
  | 8856 | BrandID에 존재하지 않는 BotID                                |
  | 8857 | 메시지베이스 내용이 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8858 | MessagebaseID가 존재하지 않음                                |
  | 8859 | BrandID에 존재하지 않는 MessagebaseID입니다.                 |
  | 8860 | messagebase의 format string 누락된 필수 항목이 있습니다.     |
  | 8861 | messagebase의 policy Info가 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8862 | messagebase의 param 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8863 | messagebase의 attribute 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8864 | messagebase의 type 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8865 | messagebaseID의 product type과 일치하지 않음                 |
  | 8866 | MessagebaseForm 내용이 부정확하거나 누락된 필수 항목이 있습니다. |
  | 8867 | messagebaseformID가 존재하지 않습니다.                       |
  | 8868 | messaegBase의 상품 코드 에러                                 |
  | 8869 | (광고)를 사용할 수 없음                                      |
  | 8870 | Action button이 허용되지 않는 messagebaseID에서 Action button을 사용하였음 |
  | 8871 | 허용되지 않은 header 값 사용                                 |
  | 8872 | header 값과 일치 하지 않은 footer 사용 (ex. Header가 0인데, footer가 있음) |
  | 8873 | footer값이 누락되어 있습니다 (ex. Header가 1 인데, footer 가 없음) |
  | 8874 | footer validation 오류 (ex. 숫자, 하이픈만 가능. 20자리)     |
  | 8875 | 등록한 패턴과 일치 하지 않음                                 |
  | 8876 | title 최대글자수를 초과했습니다.                             |
  | 8877 | description 최대글자수를 초과했습니다.                       |
  | 8878 | 최대 버튼 수를 초과했습니다.                                 |
  | 8879 | messagebaseID의 number of card 와 입력이 일치하지 않음       |
  | 8880 | 최대 미디어 용량을 초과했습니다.                             |
  | 8881 | 중계사 정보가 부정확하거나 누락된 필수 항목이 있습니다.      |
  | 8882 | 메시지 형식이 부정확하거나 누락된 필수 항목이 있습니다.      |
  | 8883 | 메시지 기술 방법이 잘못되었습니다.                           |
  | 8884 | 메시지 내용이 누락되었거나 부정확합니다.                     |
  | 8885 | 요청을 처리할 수 없는 메시지 유형입니다.                     |
  | 8886 | 같은 메시지 ID로 두 번 이상 메시지 발송이 요청됨             |
  | 8887 | 챗봇 권한 오류                                               |
  | 8888 | 발신 가능한 챗봇 상태가 아님                                 |
  | 8889 | 대행사 권한 오류                                             |
  | 8890 | 메시지 유효기간 입력 값 오류                                 |
  | 8891 | 메시지베이스 파라미터의 길이가 한계 값 이상                  |
  | 8892 | 버튼 필드를 받을 수 없는 메시지베이스 입니다.                |
  | 8893 | 최대 버튼 글자수 초과                                        |
  | 8894 | 버튼 형식 오류                                               |

  

### 10. 연동 예제 코드 (메시지 전송)

- JAVA

  ```JAVA
  package com.daou.sample;
  
  import java.io.BufferedReader;
  import java.io.IOException;
  import java.io.InputStreamReader;
  import java.io.OutputStream;
  import java.net.URL;
  import javax.net.ssl.HttpsURLConnection;
  import javax.net.ssl.SSLContext;
  import javax.net.ssl.TrustManager;
  import java.security.KeyManagementException;
  import java.security.NoSuchAlgorithmException;
  import java.security.cert.X509Certificate;
  import javax.net.ssl.X509TrustManager;
  
  public class Sample {
  	public static void main(String[] args) {
  		String input = null;
  		StringBuffer result = new StringBuffer();
  		URL url = null;
  		try {
  			/** SSL 인증서 무시 : 비즈뿌리오 API 운영을 접속하는 경우 해당 코드 필요 없음 **/
  			TrustManager[] trustAllCerts = new TrustManager[] { new X509TrustManager() {
  				public X509Certificate[] getAcceptedIssuers() { return null; }
  				public void checkClientTrusted(X509Certificate[] chain, String authType) { }
  				public void checkServerTrusted(X509Certificate[] chain, String authType) { } } };
  			
  			SSLContext sc = SSLContext.getInstance("SSL");
  			sc.init(null, trustAllCerts, new java.security.SecureRandom());
  			HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
  			
  			/** 운영 : https://api.bizppurio.com, 개발 : https://dev-api.bizppurio.com:10443 **/
  			url = new URL("https://dev-api.bizppurio.com:10443/v2/message");
  			//url = new URL("https://api.bizppurio.com/v2/message");
  			
  			/** Connection 설정 **/
  			HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
  			connection.setRequestMethod("POST");
  			connection.addRequestProperty("Content-Type", "application/json");
  			connection.addRequestProperty("Accept-Charset", "UTF-8");
  			connection.addRequestProperty("Authorization", "{인증 토큰}");
  			connection.setDoInput(true);
  			connection.setDoOutput(true);
  			connection.setUseCaches(false);
  			connection.setConnectTimeout(15000);
  			/** Request **/
  			OutputStream os = connection.getOutputStream();
  			String sms = "{\"account\":\"test\",\"refkey\":\"1234\","
  					+ "\"type\":\"sms\",\"from\":\"07000000000\",\"to\":\"01000000000\","
  					+ "\"content\":{\"sms\":{\"message\":\"SMS 전송!\"}}}";
  			os.write(sms.getBytes("UTF-8"));
  			os.flush();
  			
  			/** Response **/
  			BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream(), "UTF-8"));
  			while ((input = in.readLine()) != null) {
  				result.append(input);
  			}
  			
  			connection.disconnect();
  			System.out.println("Response : " + result.toString());
  		} catch (IOException e) {
  			// TODO Auto-generated catch block
  		} catch (KeyManagementException e) {
  			// TODO Auto-generated catch block
  			e.printStackTrace();
  		} catch (NoSuchAlgorithmException e) {
  			// TODO Auto-generated catch block
  			e.printStackTrace();
  		}
  	}
  }
  ```

  

- Python

  ```python
  # - * -coding: utf - 8 - * -
  import requests
  import json
  
  # 운영
  # url = 'https: //api.bizppurio.com/v1/message'
  # 검수
  url = 'https: //dev-api.bizppurio.com:10443/v2/message'
  
  data = {
  	'account': 'test',
  	'refkey': '1234',
  	'type': 'at',
  	'from': '07000000000',
  	'to': '01000000000',
  	'content': {
  		'at': {
  			'senderkey': '123',
  			'templatecode': '1234',
  			'message': '알림톡',
  			'button': [
  				{
  					'name': '확인하러 가기',
  					'type': 'WL',
  					'url_mobile': 'http: //www.daou.com',
  					'url_pc': 'http: //www.daou.com'
  				}
  			]
  		}
  	}
  }
  
  session = requests.Session()
  # 운영인 경우, verify 속성을 True 로 변경
  # session.verify = True
  session.verify = False
  
  headers = {'Content-type': 'application/json', 'Accept': 'text/plain', "Authorization": "{인증 토큰}"}
  response = session.post(url, data = json.dumps(data), headers=headers)
  print("Status code: ", response.status_code)
  print("Printing Entire Post Request")
  print(response.json())
  ```

  

- C#

  ```C#
  using System;
  using System.IO;
  using System.Net;
  using System.Text;
  
  namespace BIZPPURIO_HTTP_API
  {
      class Program
  	{
          static void Main(string[] args)
  		{
              // 운영
              //String url = "https://api.bizppurio.com/v2/message";
              // 검수
              String url = "https: //dev-api.bizppurio.com:10443/v2/message";
  			String postData = " {\"account\": \"test\",\"refkey\": \"test1234\","
  							+"\"from\": \"07000000000\",\"to\": \"01000000000\","
  							+"\"type\": \"sms\",\"content\": {\"sms\": {\"message\": \"SMS 전송\"}}}";
              byte[] byteArray = Encoding.UTF8.GetBytes(postData);
              HttpWebRequest request = (HttpWebRequest) WebRequest.Create(url);
              request.Method = "POST";
              request.ContentType = "application/json";
              request.Headers["Authorization"] = "Bearer" + "{인증 토큰}"
  			
              request.Timeout = 30 * 1000;
              request.ContentLength = byteArray.Length;
              using(Stream reqStream = request.GetRequestStream())
  			{
                  reqStream.Write(byteArray, 0, byteArray.Length);
              }
              Stream dataStream = request.GetRequestStream();
              dataStream.Write(byteArray, 0, byteArray.Length);
              dataStream.Close();
              WebResponse response = request.GetResponse();
              Console.WriteLine(((HttpWebResponse) response).StatusDescription);
              
              using(dataStream = response.GetResponseStream()) {
                  StreamReader reader = new StreamReader(dataStream);
                  string responseFromServer = reader.ReadToEnd();
                  Console.WriteLine(responseFromServer);
              }
          }
      }
  }
  ```



- PHP

  ```PHP
  <?php
      $sms = array("message" => "SMS 테스트");
      $content = array("sms" => $sms);
   
      $data = array();
      $data["account"] = "test";
      $data["refkey"] = "1234";
      $data["type"] = "sms";
      $data["from"] = "07000000000";
      $data["to"] = "01000000000";
      $data["content"] = $content;
   
      echo 'Request :';
      echo '<pre>';
      print_r($data);
      echo '</pre>';
          
      $json_data = json_encode($data, JSON_UNESCAPED_SLASHES);
      
      //$url    =    'https://api.bizppurio.com/v2/message';
      $url    =    'https://dev-api.bizppurio.com:10443/v2/message';
      
      $oCurl = curl_init();
      curl_setopt($oCurl,CURLOPT_URL,$url);
      curl_setopt($oCurl,CURLOPT_RETURNTRANSFER, true);
      curl_setopt($oCurl,CURLOPT_NOSIGNAL, 1);
      curl_setopt($oCurl, CURLOPT_SSL_VERIFYHOST, false);
      curl_setopt($oCurl, CURLOPT_SSL_VERIFYPEER, false);
      curl_setopt($oCurl, CURLOPT_FOLLOWLOCATION, true);
      curl_setopt($oCurl, CURLOPT_HTTPHEADER, 
  				array('Accept: application/json', 'Content-Type: application/json',
  					'Authorization: Bearer'. $token));
      curl_setopt($oCurl, CURLOPT_VERBOSE, true);
      curl_setopt($oCurl, CURLOPT_POSTFIELDS, $json_data);
      curl_setopt($oCurl, CURLOPT_TIMEOUT, 3);
   
      $response = curl_exec($oCurl);
      $curl_errno = curl_errno($oCurl);
      $curl_error = curl_error($oCurl);
   
      curl_close($oCurl);
   
      echo 'Response :';
      echo '<pre>';
      print_r(json_decode($response));
      print_r($curl_error);
      echo '</pre>';
  ?>
  ```

  

- Node.js

  ```js
  var http = require(“https");
  
  const data = 
  	JSON.stringify({“account":"test","refkey":"1234","type":"sms","from":"07000000000","to":"01000000000",
                      “content":{“sms":{“message":"SMS TEST"}}});
  
  var options = {
  	//운영, hostname: 'api.bizppurio.com'
  	hostname: 'dev-api.bizppurio.com',
  	//운영, port: 443
  	port: 10443,
  	path: '/v2/message',
  	method: 'POST',
  	headers: {
  		'Content-Type': 'application/json',
  		'Content-Length': Buffer.byteLength(data),
  		'Authorization': 'Basic' + {인증 토큰}
  	},
  	//운영, rejectUnauthorized: true
  	rejectUnauthorized: false
  };
  
  var req = http.request(options, function(res) {
  	console.log('Response/Status : ' + res.statusCode);
  	res.setEncoding('utf8');
  	res.on('data', function (body) {
  		console.log('Response/Body : ' + body);
  	});
  });
  
  req.on('error', function€ {
  	console.log('problem with request: ' + e.message);
  });
   
  req.write(data);
  req.end();
  ```



### 11. 개정 이력

| 버전 | 변경 일자  | 내용                                                         |
| ---- | ---------- | ------------------------------------------------------------ |
| 1.0  | 2019-09-11 | 최초 제정                                                    |
| 1.1  | 2019-09-20 | 내용 수정                                                    |
| 1.2  | 2020-01-15 | 내용 수정                                                    |
| 1.3  | 2020-04-08 | 내용 수정                                                    |
| 1.4  | 2020-05-20 | 내용 수정                                                    |
| 2.0  | 2020-10-29 | 인증 토큰 기능 적용으로 인한 내용 수정<br />리포트 재요청 기능 관련 내용 수정<br />RCS 대행사ID (agencyid) 입력 관련 내용 추가 |
| 2.1  | 2020-11-02 | RCS 공통 메시지 포맷 내용 수정                               |
| 2.2  | 2020-11-03 | 인증 토큰 발급 Headers 내용 수정                             |
| 2.3  | 2020-11-10 | 인증 토큰 발급 예시 추가                                     |
| 2.4  | 2021-04-16 | 알림톡 바로 연결 버튼 설명 추가                              |
| 2.5  | 2021-07-15 | 알림톡 버튼 내용 수정                                        |
| 2.6  | 2021-08-31 | 국제 발송 설명 추가, RCS 주요 변경사항 추가                  |
| 2.7  | 2021-10-12 | 매뉴얼 레이아웃 변경                                         |

