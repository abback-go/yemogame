# 아르카나 아카데미 — 개발 지침 (Claude 필독)

## 프로젝트 목표 & 제약 (매우 중요)
- **현재:** 개발자가 군 사지방에서 **웹으로만** 개발 중. 단일 `index.html`(HTML5 Canvas + vanilla JS, 의존성 0, 빌드 0).
- **최종:** **전역 후 유니티(Unity)로 이식·보완하여 스팀(Steam)에 판매**.
- 따라서 지금 웹 빌드의 진짜 산출물은 *코드*가 아니라 **검증된 디자인 + 엔진 독립적 데이터 + 이식 스펙**이다.
- 상세 전략: [`docs/strategy.md`](docs/strategy.md)

## 개발 원칙 (모든 작업에 적용)
1. **데이터는 선언적·JSON 직렬화 가능하게.** `MAPS`·`SKILL`·`QUESTS`·`MOB_DEFS`·`ABILITIES`·`CHARMS`·`TEACHERS`·`P`(물리)는 순수 데이터(숫자·문자열·배열) 위주로 유지. 로직 함수(`pd`/`unlock`/`req` 등)는 최소화하고 데이터와 분리 — 유니티에선 C#로 재구현되므로 데이터만 넘어가면 된다.
2. **단일 파일 로컬 실행을 깨지 마라.** `file://`로 직접 열어 플레이하므로 `fetch`로 외부 JSON 로드 금지. 데이터는 JS 안에 두되 **구조를 깨끗하게**. 데이터 반출은 게임 내 `exportGameData()`(콘솔/자동 다운로드)로 한다.
3. **밸런스·물리 수치는 문서화.** 이동 필 상수, 데미지/쿨다운, 맵 치수·연결, 게이트 매트릭스는 [`docs/strategy.md`](docs/strategy.md) "이식 스펙"에 최신 상태로 유지.
4. **웹 전용 기술에 과투자 금지.** 최종 아트/애니메이션·오디오·셰이더·컨트롤러 리매핑·세이브 인프라·로컬라이제이션·성능 최적화는 **유니티 단계로 미룬다.** 웹에선 게임 필·레벨·게이팅·스토리 페이싱·경제 곡선·UX만 검증.
5. **세이브 데이터 하위호환 필수.** `SAVE_KEY` 유지, 로드시 마이그레이션. 업데이트로 기존 진행이 날아가면 안 된다.
6. **아트는 최종 규격에 맞춰 플레이스홀더.** 스프라이트 치수·프레임 수를 미리 고정해, 나중에 최종 도트를 "끼워넣기"만 하면 되게.

## 코드 구조 (index.html, ~5700줄)
- `MAPS` = 존 데이터(platforms/stairs/signs/portals/npcs/mobs + 기믹: shrines/rails/updrafts/braziers/crackWalls/walls/veils/cold). `mapGroundY`=존별 주 지면.
- `SKILL`·`SKILL_ORDER`·`MOB_DEFS`·`QUESTS`(+`pd` 마이그레이션)·`TEACHERS`(수업)·`ABILITIES`(이동술 6종, 사당 습득)·`CHARMS`(각인).
- 게임 루프: 고정 타임스텝(`DT=1/120`) + rAF. `updatePlayer`(물리·이동술)·`render`(월드→후처리→HUD).
- **UI 키트:** `uiPanel`/`uiBar`/`headerText`/`shade`/`THEME_ART`(테마별 그레이드·안개·앰비언트). 새 패널은 반드시 `uiPanel` 사용.
- 치트: 게임 중 `YEMO` 타이핑 → 전 스킬·이동술·결정·각인·퀘스트 완료.

## 테스트 (헤드리스)
- Playwright(`/opt/pw-browsers/chromium`)로 `pressed[]`/`keys[]` 전역을 직접 구동. 스크래치패드에 `*.mjs` 스모크 테스트 다수.
- **변경 후 항상:** 전 존 렌더 스윕(에러 0) + 관련 회귀(튜토리얼/고급 체인/치트/세이브 마이그레이션).
- 존 로드는 **맵 ID**로(테마명 아님): frost/icevil/starmine/morugol/garden/ashstreet/temple 등.

## 문서
- `docs/arcane-academy-design.md`(메인 기획서) · `docs/world-atlas.md`(12존) · `docs/strategy.md`(웹→유니티→스팀 전략·이식 스펙).
- md 수정 시 `docs/*.html` 재생성(스크래치패드 `mkdoc*.mjs`).

## 워크플로
- 기본 브랜치 `main`에 직접 커밋·푸시. 커밋 메시지는 한국어, 상세히. PR은 요청 시에만.
- 사용자는 한국어로 소통. 답변도 한국어.
