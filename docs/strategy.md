# 아르카나 아카데미 — 개발 전략 & 유니티 이식 스펙

> **목표:** 군 사지방에서 **웹으로 디자인을 완성** → 전역 후 **유니티로 이식·보완** → **스팀 판매**.
> 이 문서는 (1) 개발 전략과 (2) 유니티 이식용 스펙 시트를 겸한다. 개발 지침은 [`CLAUDE.md`](../CLAUDE.md) 참고.

---

## Part 1. 전략

### 핵심 원칙
> **웹 빌드의 산출물은 "코드"가 아니라 "검증된 디자인 + 엔진 독립적 데이터 + 이식 스펙"이다.**
> 유니티 이식 = 웹 코드를 번역하는 게 아니라, *증명된 설계와 데이터를 C#로 재구현*하는 일.

### 웹에서 할 것 / 유니티로 미룰 것
| 웹에서 검증 (싸고 빠른 반복) | 유니티로 미룸 (웹 투자 = 낭비) |
|---|---|
| 이동 필·타격감 튜닝 | 최종 아트·애니메이션 |
| 레벨 레이아웃·맵 연결·게이팅 | 오디오(BGM·SFX)·셰이더·라이팅 |
| 퀘스트/스토리 페이싱, 전투 리듬 | 컨트롤러·키 리매핑 |
| 성장·경제 곡선, 난이도 | 세이브 인프라·로컬라이제이션 |
| UX 플로우 | 성능 최적화·스팀 연동 |

### 필 패리티 = 최대 이식 리스크
유니티 2D 플랫포머 필은 **Rigidbody2D 기본값으로는 재현 불가.** 반드시 **고정 타임스텝 커스텀 캐릭터 컨트롤러**로 Part 2의 물리 상수를 그대로 재구현. 전역 직후 **이동만 먼저 포팅해 필 일치를 검증**(make-or-break)하고 나머지를 진행할 것.

### 단계 로드맵
- **Phase A (웹, 지금):** 디자인 완성 — 필 확정·전 시스템 검증·맵 그래프·스토리 스파인·경제 튜닝. 산출물: 이 스펙 + 데이터(JSON export) + 레벨.
- **Phase B (유니티 프리프로덕션):** 데이터 임포터 + 커스텀 컨트롤러(필 재현) + **존 1개를 최종 퀄리티(아트·오디오)로** 완성 = 버티컬 슬라이스. **스팀 페이지 오픈(위시리스트 수집 시작).**
- **Phase C (프로덕션):** 검증된 파이프라인으로 존·아트·오디오 양산 + 최종 보스/엔딩 확정.
- **Phase D (폴리시·출시):** 컨트롤러·접근성·세이브 슬롯·업적·로컬라이제이션 → **넥스트페스트 데모** → 출시.

### 상업화 체크
- **스팀 페이지는 최대한 일찍** (위시리스트가 출시 성패를 좌우).
- **웹 빌드를 itch.io 웹 데모로** 활용해 트래픽→위시리스트 전환 가능.
- 스코프: 팔리는 메트로배니아 ≈ 6~12시간 + 응집된 아트 + 오디오 + 진짜 엔딩. **양산 전 버티컬 슬라이스로 기둥 증명.**

### 아트 파이프라인
- 코드 생성 도트는 이식 불가 → **최종 스프라이트 규격을 지금 고정**(캐릭터 해상도·팔레트·프레임 수·타일 크기), 웹 플레이스홀더를 그 치수에 맞춤.
- Aseprite(스프라이트시트+JSON) 권장 → 웹·유니티 공용. 레벨은 추후 **LDtk/Tiled**(웹 로더+유니티 임포터 존재)로 옮기면 공짜 이식.

---

## Part 2. 유니티 이식 스펙 시트 (실측 수치 — 변경 시 갱신)

> 최신 수치는 코드가 진실원. 데이터 전량은 게임에서 **`exportGameData()`**(브라우저 콘솔) 실행 → `arcane-data.json` 다운로드로 반출.

### 2.1 코어 물리 (커스텀 컨트롤러로 재현)
- 타임스텝: **고정 `DT = 1/120`s** + 렌더 보간. 중력 `GRAV = 2600` px/s².
- 플레이어 히트박스 `w=30, h=46`.

| 상수 | 값 | 의미 |
|---|---|---|
| runSpeed | 392 | 최고 이동속도 px/s |
| accel / friction | 4200 / 3600 | 가·감속 |
| airControl | 0.90 | 공중 제어 계수 |
| jumpV | -910 | 점프 초기 속도 |
| jumpCut | 0.42 | 가변 점프(키 떼면 상승 컷 비율) |
| coyote | 0.10s | 코요테 타임 |
| jumpBuf | 0.12s | 점프 버퍼 |
| maxFall | 1560 | 낙하 최고속 |
| ledgeSnap | 8 | 낮은 턱 자동 승월 |
| dashSpeed / dashTime / dashCd | 780 / 0.16s / 0.72s | 대시 |
| airJumpV | -820 | 이중 점프 |
| wallSlideMax / wallJumpVX / wallJumpVY | 240 / 430 / -860 | 벽 슬라이드·벽 점프 |
| meteorSpeed / meteorTime / meteorCd | 1500 / 0.34s / 1.1s | 유성 질주 |
| glideFall | 190 | 활공 낙하속 |
| blinkDist / blinkCd | 210 / 0.9s | 섬화(점멸) |
| maxHp / maxMp / mpRegen | 150 / 100 / 7 | 자원 |
| CRIT_MUL | 1.5 | 크리 배수 |

### 2.2 이동 술식 6종 (사당·스승 습득 = 메트로배니아 열쇠)
이중점프 → 벽점프 → 도깨비불(어둠 시야) → 유성 질주(균열 암벽 파괴·레일 주행) → 활공(상승 기류) → 섬화(가시덤불·재의 장막 통과). 능력이 곧 게이트 해제.

### 2.3 데이터 스키마 (JSON export 대상)
- **skills**: `{key, cd, mp, dmg[min,max], range/radius/dur, name, desc}` (bolt·fireball·field·lance·nova·inferno·meltdown)
- **mobs**: `{name, hp, w, h, speed, dmg, exp, color, contact, float}`
- **quests[54]**: `{name, desc, type(walk/act/visit/talk/kill/inspect/lesson), target, need, rewards, exp, lines?, spot?}`
- **teachers**: `{id, name, title, lessons:[{skill, task, need, req, lines}]}`
- **abilities**: `{id, name, key, desc}` / **charms**: `{id, name, cost, desc}`
- **maps(12존)**: `{name, w, h, ground, theme, tall, platforms, stairs, signs, portals, npcs, mobs, + 기믹: shrines/rails/updrafts/braziers/crackWalls/walls/veils/cold}`
- **stats**: 랭크 `D-~SSS`(15단), `RANK_NEED[]`, 스탯별 배수 함수(str/mag/agi/vit/wil).

### 2.4 월드 그래프 (12존)
아르카나 언덕(허브)·깊은 숲·지하 미궁·노을 해안·불의 마을 + 서리고개·얼음 마을·별빛 광산·모루골·부유 정원·재의 거리·첫 불꽃의 신전. 연결·게이트 매트릭스는 [월드 아틀라스](atlas.html) §5 참조.

### 2.5 유니티 재구현 노트
- 씬/타일맵: 존 = 씬(또는 대형 타일맵). `platforms`→Tilemap+Composite Collider, `oneway`→PlatformEffector2D, `stairs`→경사 콜라이더/커스텀.
- 이동술 게이트: 아이템/능력 플래그로 콜라이더·트리거 토글.
- 세이브: 현재 localStorage 스키마(스킬·스킬레벨·능력·결정·각인·퀘스트idx·brokenWalls·litBraziers·맵·좌표)를 그대로 C# 직렬화로 이식.
- 로직 함수(`pd`/`unlock`/`req`)는 데이터가 아니라 규칙 → C#로 재작성.
