1. 게시판 목록 조회 (GET /api/board/{category})
```mermaid
sequenceDiagram
    participant User
    participant Frontend as Frontend (React)
    participant Backend as Backend (Spring Boot)
    participant DB as Database (MySQL)

    User->>Frontend: /board/news 접속
    Frontend->>Backend: GET /api/board/news
    Backend->>DB: findByBoardCategoryAndStatusOrderByPidDesc("news", PUBLIC)
    DB-->>Backend: BoardPost Entity List
    Backend-->>Frontend: JSON(BoardPost List)
    Frontend-->>User: 게시글 목록 렌더링 (썸네일/제목/날짜)
```
2. 게시글 상세 조회 (GET /api/board/detail/{pid})
```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: 특정 게시글 클릭
    Frontend->>Backend: GET /api/board/detail/3
    Backend->>DB: 조회수 증가 update(viewCount + 1)
    DB-->>Backend: OK
    Backend->>DB: findById(3)
    DB-->>Backend: BoardPost Entity
    Backend-->>Frontend: JSON(BoardPost)
    Frontend-->>User: 제목/본문/이미지/조회수 표시
```
3. 게시글 작성 (POST /api/board/write)
```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: 글 작성 입력 + 이미지 업로드
    Frontend->>Backend: POST /api/upload (썸네일/본문 이미지)
    Backend->>DB: 파일 저장 (uploads/)
    Backend-->>Frontend: 이미지 URL 반환

    User->>Frontend: "등록" 버튼 클릭
    Frontend->>Backend: POST /api/board/write
        요청 body = { title, content, uid, writer, categoryTag, imageUrl, thumbnailUrl }
        헤더 = X-XSRF-TOKEN + 쿠키 세션
    
    Backend->>DB: save(BoardPost)
    DB-->>Backend: 저장 완료

    Backend-->>Frontend: 저장된 BoardPost JSON
    Frontend-->>User: 해당 카테고리 목록으로 이동
```
4. 게시글 수정 (PUT /api/board/update/{pid})
```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: 수정 버튼 클릭
    Frontend->>Frontend: 수정 form에 기존 데이터 채움

    User->>Frontend: 수정 내용 입력 후 제출
    Frontend->>Backend: PUT /api/board/update/3
        body = { title, content, uid, ... }
        인증 = 세션 쿠키 + CSRF 토큰

    Backend->>Backend: 인증/권한 체크 (admin or owner)
    Backend->>DB: findById(3)
    DB-->>Backend: BoardPost Entity

    Backend->>DB: 변경필드 update + save()
    DB-->>Backend: 저장 완료

    Backend-->>Frontend: 수정된 게시글 JSON
    Frontend-->>User: 상세 페이지로 이동
```
5. 게시글 삭제 (DELETE /api/board/delete/{pid})
```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant Backend
    participant DB

    User->>Frontend: "삭제" 클릭
    Frontend->>Backend: DELETE /api/board/delete/3
        → withCredentials: true
        → header: X-XSRF-TOKEN

    Backend->>Backend: 인증/권한 체크
    Backend->>DB: findById(3)
    DB-->>Backend: BoardPost Entity

    Backend->>DB: deleteById(3)
    DB-->>Backend: 삭제 완료

    Backend-->>Frontend: "삭제 완료"
    Frontend-->>User: 해당 카테고리 목록으로 이동
```