# 먹실 · Google Drive 연동 설정 (1회)

PC·폰 어디서든 같은 폴더로 집필하려면 ① 앱을 **HTTPS 주소로 띄우고** ② **본인 Google OAuth Client ID**를 한 번 만들면 돼.
예전엔 API Key·프로젝트 번호까지 필요했지만, 이제 **Client ID 하나면** 끝이야. (폴더는 먹실이 직접 `Muksil` 폴더를 만들어 쓰니까 Picker·API Key가 필요 없어졌어.)

---

## 1. 앱을 HTTPS로 올리기 (택1)

- **GitHub Pages**: repo에 `index.html`로 올리고 Pages 활성화 → `https://<아이디>.github.io/<repo>/`.
- **Cloudflare Pages / Netlify / Vercel**: `먹실_집필작업실.html`을 `index.html`로 바꿔 드래그 업로드.

발급된 주소(예: `https://dinosaurroot.github.io/muksil/`)를 메모해둬. 2단계에서 등록해야 해.

---

## 2. Google Cloud에서 Client ID 만들기

[console.cloud.google.com](https://console.cloud.google.com) 접속.

1. **프로젝트 생성** (이름 아무거나, 예: `muksil`).
2. **Drive API 사용 설정** — "API 및 서비스 → 라이브러리"에서 **Google Drive API** 검색 → **사용** 클릭. ⚠️ 이걸 안 켜면 폴더 연결 시 403 오류가 나.
3. **OAuth 동의 화면**:
   - User Type: **외부(External)**.
   - 앱 이름·이메일 입력.
   - **테스트 사용자**에 **본인 Google 계정 이메일 추가** → 게시는 "테스트" 상태로 둠 (Google 검증 없이 본인 계정으로 바로 사용 가능).
   - 범위(scope)는 따로 추가 안 해도 됨. 먹실이 알아서 `drive`(Drive 전체) 권한을 요청해. 동의 화면에 "Drive 보기·관리"로 표시되는데, 테스트 사용자(본인)면 검증 없이 그대로 쓰면 돼.
4. **사용자 인증 정보 → 사용자 인증 정보 만들기 → OAuth 클라이언트 ID**:
   - 유형 **웹 애플리케이션**.
   - **승인된 JavaScript 원본**에 1단계 주소 추가 (예: `https://dinosaurroot.github.io`).
     - ⚠️ 경로(`/muksil/`)는 빼고 **도메인까지만** 넣어. (`https://dinosaurroot.github.io`)
     - 로컬 테스트도 할 거면 `http://localhost:8000` 도 같이 추가.
   - 리디렉션 URI는 필요 없음(GIS 토큰 방식).
   - 생성되면 **클라이언트 ID** 복사. (비밀 아님 — 앱에 넣어도 됨)

> **API Key·프로젝트 번호는 이제 안 만들어도 돼.**

---

## 3. 먹실에서 연결

1. 앱 열기 → 시작화면 **Google Drive로 작업하기** (또는 설정 → Google Drive).
2. **OAuth Client ID** 붙여넣고 저장.
3. **Drive 연결** → 구글 로그인/동의 (테스트 사용자라 "확인되지 않은 앱" 경고가 떠도 *고급 → 이동*으로 통과).
4. **폴더 연결** → Drive 최상위에 `Muksil` 폴더를 만들거나 기존 폴더를 찾아 자동 연결.
5. **이 폴더에 새 집필 파일** 또는 **이 폴더에서 열기**.

이후 저장은 그 폴더의 `.db` 파일로 자동 동기화(입력 멈춤 5초 후 + 앱 전환 시 + Ctrl/⌘+S).

---

## 동작 / 한계 메모

- **권한 범위 `drive`(전체)**: 먹실이 Drive 전체에 접근. 그래서 **Drive 웹에서 직접 만든 `Muksil` 폴더나, 거기 수동으로 넣은 `.db`도 인식**해서 열 수 있어. (이전 `drive.file`은 앱이 만든 것만 보여서 못 열던 문제가 있었음 → v0.4.1에서 확대.)
- **토큰 1시간**: 만료되면 자동 재요청. 가끔 로그인 팝업이 한 번 더 뜰 수 있어.
- **Client ID**: 이 브라우저(localStorage)에만 저장돼. 기기 바꾸면 설정에서 다시 입력.
- **폴더 안 폴더/파일 직접 다루기**: 콘솔에서 `driveCreateFolder(name, 부모ID)`, `driveCreateFile(name, 내용, 부모ID)`, `driveList(부모ID)`, `driveReadFile(파일ID)`, `driveReadText(파일ID)` 사용 가능. 작업 폴더 ID는 `cfg.folderId`.
- **동시 편집 주의**: PC와 폰에서 같은 .db를 동시에 열어 고치면 마지막 저장이 이김. "한 번에 한 기기"를 권장.
