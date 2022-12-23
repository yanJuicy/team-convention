# Convention

## 목차
[Java Convention](##Java-Convention)

---

## Java Convention

### 1. Dto 네이밍

- [Request or Response] + [Create or Update or Find or Delete] + [도메인 이름]
    - ex) `RequestCreateUser`, `RequestUpdateUser`

### 2. 도메인 패키지 구조

- 도메인 패키지 하위에 controller, service, repository, entity, dto, mapper, exception
![image](https://user-images.githubusercontent.com/43159295/209247747-99cc6b44-45e4-4f91-b119-b6a673f92052.png)

### 3. Mapper 클래스

- Dto → Entity, Entity → Dto 변환의 책임을 Mapper 클래스가 가진다.
- [도메인] + Mapper
    - ex) `PostingMapper`
- 예시
    
    ```java
    @Component
    public class PostingMapper {
    
       public ResponseCreatePostingDto toResponse(Posting posting) {
          ResponseCreatePostingDto response = ResponseCreatePostingDto.builder()
                .id(posting.getId())
                .title(posting.getTitle())
                .writer(posting.getWriter())
                .contents(posting.getContents())
                .lastModifiedAt(posting.getLastModifiedAt())
                .commentList(new ArrayList<>())
                .build();
    
          return new GenericResponseDto<>(CREATE_POSTING_SUCCESS_MSG, response);
       }
    
       public Posting toPosting(RequestCreatePostingDto requestDto, Member member) {
          return new Posting(requestDto.getTitle(),
                requestDto.getWriter(),
                requestDto.getContents(),
                requestDto.getPassword(),
                member);
       }
    }
    
    ```
    

### 4. ExceptionHandler 클래스

- 도메인 별로 ExceptionHandler 존재
- 위치
    - …/domain/exception
    ![image](https://user-images.githubusercontent.com/43159295/209247769-71e9882e-289a-449a-9ee4-fa8f956ac9fd.png)
    
- 클래스 네이밍
    - 도메인 + Exception + Handler
- 메서드 네이밍
    - handle + [에러 추상화 명] (ex. `handleBadRequest`)
- 예시
    
    ```java@Order(Ordered.HIGHEST_PRECEDENCE)
          @RestControllerAdvice(assignableTypes = BoardController.class)
          public class BoardExceptionHandler {

            @ResponseStatus(BAD_REQUEST)
            @ExceptionHandler(IllegalArgumentException.class)
            public ResponseEntity<ExceptionResponse> handle(IllegalArgumentException e) {
              ExceptionResponse response = new ExceptionResponse(BAD_REQUEST.value(), e.getMessage());
              return new ResponseEntity<>(response, BAD_REQUEST);
            }

            @ResponseStatus(BAD_REQUEST)
            @ExceptionHandler(NoSuchElementException.class)
            public ResponseEntity<ExceptionResponse> handle(NoSuchElementException e) {
              ExceptionResponse response = new ExceptionResponse(BAD_REQUEST.value(), e.getMessage());
              return new ResponseEntity<>(response, BAD_REQUEST);
            }


          }
     ```
    

 

### 5. shared 패키지

- 응답 성공, 에러 메시지를 반환하는 dto를 가지는 패키지
- 엔터티에서 사용하는 Timestamped 위치

![image](https://user-images.githubusercontent.com/43159295/209248095-240c5bcf-e17e-4034-b39c-0be32ace2766.png)

- 응답 메시지 관리는 enum을 활용
    - 성공 응답
        - 위치 …/common/response
        - 예시
        
        ![image](https://user-images.githubusercontent.com/43159295/209248151-992db7f9-8b3e-4f8e-8d26-acffe5069604.png)
        
    - 에러 응답
        - 위치 …/common/exception
        - 예시
        
        ![image](https://user-images.githubusercontent.com/43159295/209248176-d9aeeac5-959d-4017-b9d7-a15526ca7d54.png)
        
    

### 6. 패키지 구조

```java
├─main
    │  ├─java
    │  │  └─com
    │  │      └─hanghae
    │  │          └─blog
    │  │              └─api
    │  │                  ├─common
    │  │                  │  ├─entity
    │  │                  │  ├─exception
    │  │                  │  └─response
    │  │                  └─posting
    │  │                  │   ├─controller
    │  │                  │   ├─dto
    │  │                  │   ├─entity
    │  │                  │   ├─exception
    │  │                  │   ├─mapper
    │  │                  │   ├─repository
    │  │                  │   └─service
		|  |                  └─ ...

```

### 7. Single Import

패키지 import는 single import로 한다.

![image](https://user-images.githubusercontent.com/43159295/209248197-f3348490-8919-4da4-b901-1be1b5266ccd.png)

와일드카드 임포트의 단점은 다음 글을 참조

[https://dev.to/kingori/import-5531](https://dev.to/kingori/import-5531)

### 8. static import

`ResponseMessage`, `ExceptionMessage`등 static 필드의 import는 static import로 한다.

- ex) `import static com.hanghae.blog.common.response.ResponseMessage.*CREATE_POSTING_SUCCESS_MSG*;`

![image](https://user-images.githubusercontent.com/43159295/209248232-a2387570-3caf-4c20-9410-ddba570a0f6d.png)

### 9. ResponseMessage Controller vs Service

ResponseMessage는 controller에서 반환한다. 

![image](https://user-images.githubusercontent.com/43159295/209248284-69ee4682-d754-4bac-80ec-389fee27805c.png)

### 10. 주석

- 블록 주석

-파일 메서드, 자료구조, 알고리즘에 대한 설명 제공 시 사용

-주석 앞 띄어쓰기 한번

```java
/*
* 여기에 블록 주석을 작성하기
*/
```

- 한 줄 주석

-주석이 충분히 짧은 경우 한 줄에 작성

-주석이 설명하는 코드와 같은 단위로 들여쓰기

-주석 앞뒤로 띄어쓰기 한번

```java
if(condition){
	/* 이 조건을 만족하면 실행된다 */
	...
}
```

- 꼬리 주석

-주석 내용이 아주 짧다면 코드와 같은 줄에 넣기

-대신 멀찌감치 넣어 실제 코드와 구분을 꼭 시켜주기

-주석 앞뒤로 띄어쓰기 한번

```java
if (a==2) {
    return TRUE;            /* 특별한 경우 */
} else {
    return FALSE;           /* 그 외의 경우 */
}
```

- 줄 끝 주석

-“//”는 한 줄 모두를 주석 처리하거나, 한 줄의 일부분을 주석 처리해야 할 경우 사용

-어떤 코드의 일부분을 주석 처리할 때, 여러 줄에 걸쳐서 사용하는 경우도 있음

 

```java
if (foo > 1) {
    // double-flip 실행한다. 
    ...
}
else {
    return false;            //프로그램을 종료하는 flag
}
//if (bar > 1) {
//
//    // double-flip 실행한다. 
//    ....
//}
//else {
//    return false;
//}
```

---

## Commit Convention

커밋 메시지 앞에 Tag Name을 작성한다.

- `ex) feat: 로그인 API 기능 추가`
- `ex) refactor: posting service 내 인증 로직 필터로 분리`

| Tag Name | Description |  |
| --- | --- | --- |
| feat | 새로운 기능을 추가 |  |
| fix | 버그 수정 |  |
| design | CSS 등 사용자 UI 디자인 변경 |  |
| !HOTFIX | 급하게 치명적인 버그를 고쳐야하는 경우 |  |
| style | 코드 포맷 변경, 세미 콜론 누락, 코드 수정이 없는 경우 |  |
| refactor | 프로덕션 코드 리팩토링 |  |
| comment | 필요한 주석 추가 및 변경 |  |
| docs | 문서 수정 |  |
| test | 테스트 코드, 리펙토링 테스트 코드 추가, Production Code(실제로 사용하는 코드) 변경 없음 |  |
| chore | 빌드 업무 수정, 패키지 매니저 수정, 패키지 관리자 구성 등 업데이트, Production Code 변경 없음 |  |
| rename | 파일 혹은 폴더명을 수정하거나 옮기는 작업만인 경우 |  |
| remove | 파일을 삭제하는 작업만 수행한 경우 |  |

---

## Issue Convention

- API 별로 이슈 작성
- 코딩 시작 전 이슈를 만들고 코딩 시작할 것
    - 예시
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/68f1264c-addb-4e43-a005-494fea50cae7/Untitled.png)
    
- 이슈 템플릿을 활용해 형식에 맞는 이슈 생성
    - 예시
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5556cec-32b6-4db5-bfeb-3b061b7ffae9/Untitled.png)
    

---

## PR Convention

- API 단위로 PR을 보내자 (레포 → 서비스 → 컨트롤러를 아우르는 한 사이클)
- PR 제목 : [feat/refactor/docs/…] 이슈 이름
    - 다음 이모지 사용
    
    ```java
    🚀 Release
    🐛 Fix
    ✨ Feat
    📝 Doc
    ♻️ Refactor
    🔧 Chore
    ⏪️ Revert
    🧪 Test
    🎉 Init
    ```
    
    - ex) `♻️ **Refactor: ResponseDto 재구조화**`
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b538094-1bff-4ffe-824d-cc72b4aa4da5/Untitled.png)
    
- PR 템플릿 형식에 맞게 작성
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/79be3ad8-2e22-4278-b4a6-578047162618/Untitled.png)
    

---

## 브랜치 전략

- feature/#[이슈번호] → develop
    - 예시 `feature/#1`
    - feature 브랜치에서 기능을 완성을 한 후 develop 브랜치로 PR

- refactor/#[이슈번호] → develop
    - 예시 `refactor/#10`
    - refactor 브랜치에서 refactoring 후 develop 브랜치로 PR

- 프로젝트 끝난 후 develop → main 머지

---

## PR 코드 리뷰

- PR 보낸 후 슬랙에 코드 리뷰 요청
- 적어도 한 명이 코드리뷰 진행
- 코드 리뷰 연습
- 리뷰어 지정은 PR 보낸 사람이 알아서 지정
- PR 머지는 보낸 사람이 머지
