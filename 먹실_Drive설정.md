# 먹실 · Google Drive 연동 설정 (1회)

모바일에서 쓰려면 ① 앱을 **HTTPS 주소로 띄우고** ② **본인 Google OAuth 자격증명**을 한 번 만들어야 해.
코드는 그대로고, 아래 두 가지만 준비하면 PC·폰 어디서든 같은 드라이브 폴더로 집필할 수 있어.

---

## 1. 앱을 HTTPS로 올리기 (택1)

- **Cloudflare Pages / Netlify / Vercel**: `먹실_집필작업실.html`을 `index.html`로 이름만 바꿔 드래그 업로드 → `https://...pages.dev` 같은 주소 발급.
- **GitHub Pages**: repo에 올리고 Pages 활성화. (HTML 껍데기만 공개돼도 데이터는 네 드라이브에 있으니 안전)

발급된 주소(예: `https://muksil.pages.dev`)를 메모해둬. 다음 단계에서 등록해야 해.

---

## 2. Google Cloud에서 자격증명 만들기

[console.cloud.google.com](https://console.cloud.google.com) 접속.

1. **프로젝트 생성** (이름 아무거나, 예: `muksil`).
2. **API 사용 설정**: "API 및 서비스 → 라이브러리"에서 **Google Drive API** 검색 → 사용.
3. **OAuth 동의 화면**:
   - User Type: **외부(External)**.
   - 앱 이름/이메일 입력.
   - **테스트 사용자**에 **본인 Google 계정 이메일 추가** → 게시는 "테스트" 상태로 둠 (이러면 Google 검증 없이 본인 계정으로 바로 사용 가능).
   - 범위(scope)는 따로 추가 안 해도 됨. 먹실이 `drive.file`만 요청해.
4. **사용자 인증 정보 → 사용자 인증 정보 만들기**:
   - **OAuth 클라이언트 ID** → 유형 **웹 애플리케이션**.
     - **승인된 JavaScript 원본**에 1단계 주소 추가 (예: `https://muksil.pages.dev`).
       - 로컬 테스트도 할 거면 `http://localhost:8000` 도 같이 추가.
     - 리디렉션 URI는 필요 없음(GIS 토큰 방식).
     - 생성되면 **클라이언트 ID** 복사. (비밀 아님 — 앱에 넣어도 됨)
   - **API 키**도 하나 더 만들기 (Picker 폴더 선택에 필요). 만든 뒤 키 제한에서 "Google Picker API" 정도로 제한하면 더 안전.

---

## 3. 먹실에서 연결

1. 앱 열기 → **설정**(또는 시작화면 ☁ 버튼).
2. **OAuth Client ID**, **API Key** 붙여넣고 저장.
3. **Drive 연결** → 구글 로그인/동의 (테스트 사용자라 "확인되지 않은 앱" 경고가 떠도 *고급 → 이동*으로 통과).
4. **폴더 선택** → 집필 파일을 둘 드라이브 폴더 지정.
5. **이 폴더에 새 집필 파일** 또는 **이 폴더에서 열기**.

이후 저장은 그 폴더의 `.db` 파일로 자동 동기화(입력 멈춤 5초 후 + 앱 전환 시 + Ctrl/⌘+S).

---

## 동작 / 한계 메모

- **권한 범위 `drive.file`**: 먹실이 만들거나 Picker로 직접 고른 항목에만 접근. 폴더에 *다른 경로로* 넣어둔 파일은 안 보임(네가 고른 정책).
- **토큰 1시간**: 만료되면 자동 재요청. 가끔 로그인 팝업이 한 번 더 뜰 수 있어.
- **Client ID / API Key**: 이 브라우저(localStorage)에만 저장돼. 기기 바꾸면 설정에서 다시 입력.
- **폴더 안 폴더/파일 직접 다루기**: 콘솔에서 `driveCreateFolder(name, 부모ID)`, `driveCreateFile(name, 내용, 부모ID)`, `driveList(부모ID)`, `driveReadFile(파일ID)`, `driveReadText(파일ID)` 사용 가능. 작업 폴더 ID는 `cfg.folderId`.
- **동시 편집 주의**: PC와 폰에서 같은 .db를 동시에 열어 고치면 마지막 저장이 이김. "한 번에 한 기기"를 권장.
