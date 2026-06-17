# AGENTS.md — 먹실(MUKSIL)

한국식 장르소설(웹소설) 집필 앱. 산출물은 **`먹실_집필작업실.html` 단일 파일**(빌드·번들러·프레임워크 없음).
상세 설계·스키마·확장법은 **`먹실_개발문서.md`** 참고(작업 전 1회 정독). 기능을 바꾸면 그 문서도 같이 갱신할 것.

## 절대 규칙
1. **전역 함수 유지**: 인라인 `onclick`이 전역 함수를 호출한다(예: `onclick="openWorkForm()"`). 핸들러는 반드시 최상위 `function` 선언으로 두고, 이름 변경 시 HTML 문자열도 함께 수정. 클로저/모듈로 숨기지 말 것.
2. **`esc()` 이스케이프**: `innerHTML`에 들어가는 모든 사용자 텍스트는 `esc()`로 감싼다. 인라인 핸들러 인자에 들어가는 문자열은 `.replace(/'/g,"\\'")`도 추가.
3. **쓰기는 `run()`/`insert()`**로만(자동저장 `markDirty` 트리거). `db.run` 직접 호출 지양.
4. **색·폰트는 CSS 변수만**(`--ink/--paper/--celadon/--serif/--sans` 등). 하드코딩 금지. 라이트테마는 `:root.light`에서 오버라이드.
5. **UI 문구는 한국어·캐주얼 존댓말**. 토스트는 짧게.
6. **반응형 분기 760px**(이하 드로어/터치). 모바일 진입 시 자동 포커스 금지.

## 구조 요약
- 단일 HTML: `<style>`(CSS 전체) + 마지막 `<script>`(앱 로직). 외부 스크립트 3개: sql.js 1.12.0, GIS(`gsi/client`), gapi(`api.js`).
- DB: sql.js(SQLite WASM), 메모리 인스턴스 `db`. 테이블: works/chapters/characters/lore/plots/notes/writing_log/meta.
- 저장: 로컬(File System Access API) ↔ Google Drive(scope `drive.file`). 분기 = `isDrive()`. 전역 상태: `state`, `cfg`(localStorage `muksil-drive`), `gauth`(토큰, 메모리).
- 라우팅: `VIEWS[]` → `go(view)` → `RENDER[view]()`. 렌더는 `innerHTML` 재생성(가상DOM 없음).

## 변경 후 검증 (반드시 실행)
```bash
# 1) 앱 스크립트 추출 + 문법
python3 -c "import re;h=open('먹실_집필작업실.html',encoding='utf-8').read();open('app.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>',h,re.S)[-1])"
node --check app.js

# 2) jsdom 로드(런타임 에러/핸들러 결선) + sql.js 스모크(스키마/집계) + Drive REST 목킹
npm i jsdom sql.js@1.12.0
#   JSDOM(html,{runScripts:'dangerously',url:'https://localhost/'}) 로 로드 — jsdomError 0 확인
#   상세 테스트 패턴은 먹실_개발문서.md §10 참고
rm -f app.js
```
- **자동 검증 불가(사람이 브라우저로 확인)**: 실제 OAuth 동의·Picker·File System 저장. CLI는 `python -m http.server 8000`까지만.
- Codex.ai 아티팩트 프리뷰에선 localStorage/FS/Drive 전부 미동작 → **독립 실행/호스팅에서 테스트**.

## 주의 (전체 목록은 개발문서 §11)
- **개발 단계 — DB 스키마 자유 변경 허용**: 현재 `내소설.db`의 데이터는 전부 임시(테스트)다. 마이그레이션 부담 없이 스키마를 자유롭게 바꿔도 된다 — 컬럼 추가/삭제, 테이블 재생성, `.db` 통째로 재생성 모두 OK. 시드/스키마(`CREATE TABLE`)에 컬럼을 더하고, 기존 `.db`가 안 맞으면 새로 만들면 된다.
  - **정식 데이터 단계로 넘어가면** 이 자유는 끝난다: 열 때 `CREATE TABLE IF NOT EXISTS`라 기존 .db에 누락 컬럼이 자동 추가되지 않으므로, 그때는 `meta.schema_version` 기반 `ALTER TABLE` 마이그레이션 루틴을 신설하고 `SCHEMA_VERSION`을 올릴 것.
- Drive 자동저장은 매번 DB 전체 업로드. 동시 편집은 last-write-wins(충돌 감지 없음).
- `file://`에서는 Drive(OAuth) 불가 → `http://localhost:포트`/HTTPS로 서빙.
