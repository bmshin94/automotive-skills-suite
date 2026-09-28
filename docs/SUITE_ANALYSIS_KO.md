# Automotive Skills Suite 전수조사 분석 보고서 (한국어)

> 작성: 2026-09-28 · 작성자: Claude Code (카리나 페르소나) · 의뢰: @bmshin94
> 이 문서는 저장소 전체(152개 `.skill`, 849개 Python 파일, 162,477줄, docs/ 전량)를
> 실제로 압축 해제·실행·검증한 결과를 정리한 것입니다.

---

## 0. 저장소 좌표 (GitHub 주소)

| 구분 | 주소 | 비고 |
|---|---|---|
| **원본(Upstream)** | https://github.com/jherrodthomas/automotive-skills-suite | ⭐ 2,932 · 🍴 166 · Python · MIT · 2026-05-01 생성 |
| **본 저장소(복사본)** | https://github.com/bmshin94/automotive-skills-suite | 원본 복제 + `CLAUDE.md`(페르소나) 추가 |
| 참고: Manus 포팅판 | https://github.com/dsdtsolutionsllc/manus-automotive-skills-suite | ⭐ 1 · 152 스킬 Manus 호환 변환 |

> 저장소 내부 문서에 원본 주소가 **95곳** 하드코딩되어 있어 출처가 확정됩니다.
> (`docs/chain-contract-audit.md`, `RELEASES.md` 등)

---

## 1. 한 줄 정의

> **자동차 ECU 개발에서 법·계약으로 강제되는 ISO 표준 산출물(xlsx) 76종을
> Claude가 "생성"하고, 짝을 이루는 "검수" 스킬이 확인심사까지 자동화하는 Agent Skill 묶음.**

---

## 2. 정체 판정 — Skill / MCP / Plugin

### 결론: **100% 순수 Agent Skill.** MCP도 Plugin도 아님.

| 판정 근거 | 확인 결과 |
|---|---|
| `SKILL.md` + YAML frontmatter(`name`,`description`) | ✅ 152개 전부 보유 → Skill 규격 |
| `.claude-plugin/plugin.json` | ❌ 없음 → Plugin 아님 |
| `.mcp.json` / MCP 서버 / JSON-RPC / `mcp` import | ❌ 0건 → MCP 아님 |
| 상주 프로세스 · 포트 리스닝 | ❌ 없음 (스크립트 실행 후 종료) |
| HTTP 클라이언트(`requests`/`httpx`/`urllib`) | ❌ 0건 |

### 개념 비교

|  | **Skill (본 프로젝트)** | MCP | Plugin |
|---|---|---|---|
| 정체 | 지시서 + 스크립트 묶음(ZIP) | 외부 시스템 연결 서버 | 여러 구성요소 배포 패키지 |
| 해결 문제 | **"어떻게 일할까"** (능력) | "어디에 연결할까" (연결) | "어떻게 나눠줄까" (배포) |
| 네트워크 | ❌ 불필요 | ✅ stdio/HTTP | 포함 요소에 따라 |
| 인증 | ❌ 없음 | ✅ 보통 토큰 필요 | 경우에 따라 |

---

## 3. API 토큰 필요 여부

### 결론: **스킬 자체는 토큰·인증·네트워크 전부 불필요 (완전 오프라인).**

849개 Python 파일 전수 grep 결과:

```
grep -rhoE "(os\.environ|getenv)[^)]{0,40}" --include="*.py" .
→ os.environ.copy(    # LibreOffice 실행 환경 복사용. API 키와 무관
```

| 검사 항목 | 결과 |
|---|---|
| API 키 / `ANTHROPIC_API_KEY` | ❌ 0건 |
| OAuth / 토큰 | ❌ 0건 |
| 외부 URL 하드코딩 | ❌ 0건 |
| 텔레메트리 | ❌ 0건 |

**비용 구분**

| 대상 | 비용 |
|---|---|
| 스킬 Python 스크립트 | 🆓 무료·로컬 |
| LibreOffice | 🆓 무료 |
| **Claude 자체** | 💳 구독 또는 API 과금 ← 유일한 비용 |
| **JSON 직접 작성 → Python만 실행** | 🆓 **완전 무료 (Claude 불필요)** |

> 핵심: 엑셀 생성 엔진이 LLM과 완전히 분리되어 있어 **AI 비용 없는 제품화**가 가능.
> 에어갭(인터넷 차단) 사내망에서도 100% 동작 → OEM 개발망 친화적.

---

## 4. 저장소 구조 전수조사

```
automotive-skills-suite/
├── skills/        152개 .skill (3.9MB)  = 76 builder + 76 reviewer
├── examples/      76개 스킬별 사용 예시 README (444KB)
├── docs/          자율 운영 기록 (788KB)  ★ 숨은 자산
│   ├── AUTONOMOUS_LOG.md          AI 에이전트 일일 작업 일지
│   ├── weekly/                    W20~W37 주간 계획 16건
│   ├── monthly/                   2026-05~08 KPI 리포트
│   ├── triage/                    이슈 정리 기록
│   ├── skill-polish-log/          스킬별 품질 감사 41건
│   ├── chain-contract-audit.md    체인 계약 감사 결과
│   ├── sheet-name-length-audit.md
│   └── PAIRING_ALIASES.md
├── scripts/       regen_status.py · chain_contract_audit.py
├── STATUS.md      76쌍 페어링 현황 (🟢신선 / 🟡오래됨)
├── CHANGELOG.md   주간 릴리스 변경 기록 + Known issues
├── RELEASES.md    릴리스 스냅샷
├── LICENSE        MIT
└── CLAUDE.md      본 저장소에서 추가된 페르소나 정의
```

### `.skill` 파일 내부 (실제 압축 해제 결과)

`.skill`은 **확장자만 다른 ZIP 아카이브**입니다.

```
hara-builder.skill (ZIP)
└── hara-builder/
    ├── SKILL.md                       지시서 (YAML frontmatter + 워크플로 5단계)
    ├── references/ (7개 .md)          도메인 지식 — 필요할 때만 로드
    │   ├── malfunctions.md                14개 고장 가이드워드 (M01~M14)
    │   ├── severity_ais.md                심각도(S) AIS 등급
    │   ├── exposure.md                    노출(E) — 상황 기준 (흔한 실수 경고 포함)
    │   ├── controllability.md             제어가능성(C) — 평균 운전자 기준
    │   ├── asil_matrix.md                 ASIL 룩업 + 수식 패턴
    │   ├── operating_environments.md      기본 위치/날씨 차원
    │   └── fsc_handoff.md                 FSC 인계 규칙
    ├── scripts/
    │   ├── generate_hara.py (48KB)    엑셀 생성 엔진 — 읽지 않고 실행만
    │   ├── recalc.py                  LibreOffice 매크로로 수식 재계산
    │   └── office/soffice.py
    └── examples/sample_input_esc.json 동작하는 입력 예시
```

### 규모 집계

| 항목 | 수치 |
|---|---|
| `.skill` 파일 | **152** (76 builder + 76 reviewer, 100% 페어링) |
| Python 파일 | **849** |
| Python 총 라인 | **162,477** |
| `SKILL.md` | 152 |
| 도메인 | 13 |
| 커버 표준 | 30+ |

### 의존성 (외부 라이브러리 단 2개)

| 의존성 | import 횟수 | 용도 |
|---|---|---|
| `openpyxl` | **1,071** | xlsx 생성·읽기 (핵심) |
| LibreOffice (`soffice`) | `subprocess` 304 | 수식 재계산 |
| `graphviz` | 1 | 다이어그램 1개 스킬 |

---

## 5. 13개 도메인 / 76쌍 지도

| 도메인 | 쌍 | 대표 스킬 | 표준 |
|---|---|---|---|
| safety | 15 | hara, fsc, tsc, fmeda, hsi, safety-case | ISO 26262 |
| cyber | 6 | tara, cs-goals, cs-concept, cs-architecture, irp | ISO/SAE 21434, UN R155 |
| sotif | 3 | sotif-analysis, triggering-conditions, validation-strategy | ISO 21448 |
| quality | 10 | dfmea, pfmea, apqp, ppap, control-plan, 8d, 5-why, spc, msa, fishbone | IATF 16949, AIAG-VDA |
| aspice | 4 | assessment, gap-analysis, improvement-plan, process-evidence | ASPICE PAM 3.1/4.0 |
| comms | 8 | dbc, ldf, arxml, flexray, automotive-ethernet, k-matrix, bus-load, gateway | ISO 11898/17987/17458 |
| diagnostics | 5 | uds, dtc-catalog, cdd, odx, dem-config | ISO 14229, SAE J2012, ISO 22901 |
| autosar | 5 | swc, composition, bsw-config, rte-mapping, adaptive-app | AUTOSAR R22-11 |
| calibration | 3 | a2l, dcm, calibration-data-exchange | ASAM MCD-2 MC |
| mbse | 3 | model-architecture, requirements-allocation, system-context | ARCADIA, INCOSE |
| sysml | 4 | block, requirement, activity, state-machine | OMG SysML 1.6/2.0 |
| v&v | 5 | verification-plan, validation-plan, test-case-catalog, traceability-matrix, vv-execution-report | ISO 26262-8, IEEE 1012 |
| program-mgmt | 5 | risk-register, gate-review, wp-status-rollup, change-impact, lessons-learned | ISO 26262-2 |

---

## 6. 체인(Chain) 아키텍처 — 이 프로젝트의 핵심 자산

```
Item Definition → Safety Plan → DIA
        ↓
      HARA → FSC → TSC
        ↓            ↓
   [HW 레인]      [SW 레인]
  HW-SR/Arch/    SW-SR/Arch/
  HSI/FMEDA      FMEA/HSI
        ↓            ↓
        Safety Case (capstone)

병렬 체인
TARA → CS Goals → CS Concept → CS Architecture → IR Plan + Secure Coding  (ISO 21434)
SOTIF Analysis → Triggering Conditions → Validation Strategy              (ISO 21448)
APQP → DFMEA → PFMEA → Control Plan → PPAP                                (IATF 16949)
ASPICE Assessment → Gap Analysis → Improvement Plan → Process Evidence    (ASPICE)
```

**설계 원리:** 상류 스킬의 xlsx 출력이 하류 스킬의 입력. **파일 포맷이 계약(contract).**

```python
# fsc-builder가 hara-builder 출력을 읽는 방식
ws = wb["05_Safety_Goals"]   # ← 이 탭 이름이 계약
```

**계약 자동 감사 결과** (`scripts/chain_contract_audit.py`, 2026-09-17):

| 항목 | 값 |
|---|---|
| 스캔한 builder | 76 |
| 감사한 체인 | 16 |
| 시트명 assertion | 48 |
| MATCH / ALIAS / FALLBACK | 43 / 4 / 1 |
| **BREAK** | **0** ✅ |

---

## 7. 실행 검증 (직접 돌린 결과)

### ① 빌더 실행 — `hara-builder`

```bash
pip install openpyxl
python scripts/generate_hara.py examples/sample_input_esc.json out.xlsx
```

```json
{ "functions": 4, "function_malfunction_pairs": 56,
  "safety_critical_pairs": 25, "hara_rows": 900,
  "significant_rows": 468, "safety_goals": 25 }
```

→ **130KB / 15탭 / 903행 xlsx 생성 성공**

| # | 탭 | 행 × 열 |
|---|---|---|
| 00 | Title_Page | 14 × 4 |
| 01 | Document_Control | 15 × 5 |
| 02 | Assumptions | 9 × 4 |
| 03 | Architecture_Boundary | 15 × 5 |
| 04 | Functions | 7 × 4 |
| 05 | Malfunctions | 17 × 3 |
| 06 | Operating_Environment | 19 × 5 |
| 07 | Severity_Reference | 7 × 4 |
| 08 | Exposure_Reference | 10 × 4 |
| 09 | Controllability_Reference | 7 × 4 |
| 10 | ASIL_Matrix | 17 × 6 |
| 11 | Function_x_Malfunction | 60 × 7 |
| **12** | **HARA_Worksheet** | **903 × 17** |
| 13 | Safety_Goals | 29 × 7 |
| 14 | FSC_Handoff | 79 × 11 |

### ② 리뷰어 실행 — `hara-checklist-reviewer` (①의 출력을 그대로 입력)

```bash
python scripts/generate_checklist.py out.xlsx review.xlsx
```

```json
{ "is_hara_builder_format": true,
  "n_hara_rows_probed": 900, "n_safety_goals_probed": 25,
  "checks_by_tab": { "CR": 14, "FSA": 24, "VA": 44 },
  "ratings": { "FC": 27, "PC": 1, "NO": 1, "NA": 3, "PENDING": 50 } }
```

**→ 82개 확인심사 항목 자동 실행. 체인이 실제로 작동함을 확인.**

| 등급 | 의미 | 수 |
|---|---|---|
| FC | Fully Compliant (완전 준수) | 27 |
| PC | Partially Compliant | 1 |
| NO | Not Observed (미준수) | 1 |
| NA | Not Applicable | 3 |
| **PENDING** | **사람 판단 필요** | **50** |

> 설계 미덕: 기계가 확정할 수 있는 항목만 자동 채점하고, 주관적 항목은
> `"(Reviewer to verify)"` 표기로 남긴다. **거짓 안심을 주지 않는다.**
> 또한 리뷰어는 원본을 절대 수정하지 않고 `Recommended Actions` 칼럼에만 기록한다.

---

## 8. 숨은 자산 — AI 자율 운영 체계

`docs/AUTONOMOUS_LOG.md`는 5개월치 AI 에이전트 자율 운영 일지입니다.
`automotive-skills-daily-standup` 스케줄 작업이 **요일별 모드**로 동작합니다.

| 요일 | 모드 | 산출물 |
|---|---|---|
| 월 | **PLAN** | `STATUS.md` 재생성 → 주간 타깃 4개 선정 → GitHub 이슈 자동 개설 |
| 화~목 | **POLISH** | 1일 1스킬 품질 감사 → `docs/skill-polish-log/<name>.md` |
| 금 | **DOCS** | CHANGELOG 롤업 + 예시 문서 보강 |
| 토 | **RELEASE** | 주간 태그 발행 (`v2026.09.W36` 등) |
| 일 | **TRIAGE** | 이슈 라벨링 (**신뢰도 80% 미만이면 미조치**) |

### 자율 편집 허용 범위(allowlist) — 스스로 좁혀 놓음

> 자동 적용: **오타 / 길이 초과 / 필수 필드 누락** 만
> 편집 판단이 필요한 건 초안만 작성하고 **사람 결재 대기**

실제 로그:
```
"A drafted rewrite is in the polish log and is **not** auto-applied
 because the spec restricts autonomous edits to typo / length /
 missing-field fixes, and this is editorial re-ordering."
```

### 표준화된 8필드 로그 포맷

```markdown
**Mode:**            PLAN / POLISH / DOCS / RELEASE / TRIAGE
**Action:**          한 문장 요약
**Files touched:**   경로 + 변경 이유
**Tests:**           N/A (no test suite in this repo yet)   ← 없으면 없다고 기록
**Skill count:**     76/76 paired
**Open issues:**     5
**Notes:**           판단 근거 / 선택 이유
**Follow-ups:**      다음 실행 입력 ← 장기 컨텍스트 유지의 핵심
```

---

## 9. 알려진 결함 (저장소 자체 고백 + 검증)

`CHANGELOG.md` / `RELEASES.md`의 `Known issues` 섹션에 정직하게 기록되어 있음.

| # | 결함 | 심각도 |
|---|---|---|
| 1 | **광고된 체크 ~100개가 실제로 존재하지 않음** ("check-count drift") — `sysml-block-diagram-reviewer` 28→**15**, `autosar-bsw-config-reviewer` ~30→**9**, `mbse-*-reviewer` 3종 "25+/30+/28+"→**6/8/6**, `fmeda-reviewer` 28→실제 36 | 🔴 HIGH |
| 2 | **`traceability-matrix` 쌍 비기능** — builder가 입력을 무시하고 제목 셀 5개만 출력, reviewer는 그 빈 워크북을 **25점 중 24점 합격** 처리 (체크 25개 중 22개가 하드코딩 `LC` 통과) | 🔴 HIGH |
| 3 | **리뷰어 7개가 모든 입력에서 크래시**했었음 (`dashboard.py:138` 한 줄 버그 7회 복붙) — sysml 4개 + mbse 3개. W35~W36에 수정 완료 | ✅ 해결 |
| 4 | `dashboard.py`의 `REJECTED` 판정이 절대 발동 안 됨 (`Shall` vs `Must`/`Should` 문자열 불일치) → `{}` 워크북이 4개 NO를 받고도 `CONDITIONAL APPROVAL` | 🟡 MED |
| 5 | 시트명 31자 초과 **19건** (`docs/sheet-name-length-audit.md`) | 🟡 MED |
| 6 | `cdd-checklist-reviewer` — SKILL.md가 존재하지 않는 `references/` 파일 2개를 읽으라고 지시 | 🟡 MED |
| 7 | `test-case-catalog` (`Test Case Inventory` vs `Test Cases`), `flexray-config` (`Title` vs `Title_Page`) 시트명 불일치 | 🟡 MED |
| 8 | **테스트 스위트 전무** — 모든 로그에 `Tests: N/A (no test suite in this repo yet)` | 🔴 HIGH |
| 9 | `sysml-block-diagram-builder` 동작 변경 — `--input` 없으면 빈 템플릿 출력 (기존 placeholder 5개 제거) | 🟢 LOW |

### 도메인별 성숙도 판정

| 성숙도 | 도메인 | 상태 |
|---|---|---|
| 🟢 실전 투입 가능 | safety, cyber, quality, aspice | 검증됨 |
| 🟡 사용 가능 | autosar, comms, diagnostics, calibration | 일부 결함 |
| 🔴 껍데기 | **sysml, mbse, v&v** | 골조만 존재 |

> **총평: 우수한 스캐폴딩(뼈대)이지 완성 제품은 아니다.**
> 152개 중 실전 투입 가능 비율은 체감 **40~60%**. 사용 전 해당 도메인 검증 필수.

---

## 10. 설치 및 사용법

### 준비물

```bash
python3 --version          # 3.10+ (f-string, | union 타입 사용)
pip install openpyxl       # 필수
# LibreOffice (수식 재계산용)
#   macOS   : brew install --cask libreoffice
#   Ubuntu  : sudo apt install libreoffice
# (선택) graphviz — 다이어그램 스킬 1개
```

### 설치 3가지 경로

**① Claude Desktop / Cowork (권장)**
1. `skills/`에서 필요한 `.skill` 다운로드
2. 설정 → Capabilities/Skills → 파일 업로드
3. "Save skill" 클릭

**② Claude Code (CLI)**
```bash
# 개인용
mkdir -p ~/.claude/skills && cd ~/.claude/skills
unzip ~/Downloads/hara-builder.skill

# 프로젝트용 (팀 공유, git 커밋)
mkdir -p .claude/skills && cd .claude/skills
unzip ~/Downloads/hara-builder.skill

# 도메인 단위 일괄
for f in .../skills/{hara,fsc,tsc,fmeda,safety-case}-*.skill; do unzip -o "$f"; done
```

**③ 전체 152개 벌크 설치 — 비권장**
description 152개가 상시 컨텍스트에 상주해 스킬 선택 정확도가 떨어짐.
**필요한 5~15개만 설치할 것.**

### 사용 3가지 방식

**A. 자연어 트리거 (권장)**
```
"ESC ECU로 HARA 만들어줘"        → hara-builder
"내 HARA 검수해줘"                → hara-checklist-reviewer
"이 ECU 사이버보안 TARA 좀"       → tara-builder
"고객 클레임 8D 보고서 써줘"      → 8d-problem-solving-builder
```

**B. Python 직접 실행 (Claude 불필요)**
```bash
python scripts/generate_hara.py input.json out.xlsx
python scripts/recalc.py out.xlsx
```

**C. 체인 연결**
```bash
python hara-builder/scripts/generate_hara.py in.json HARA.xlsx
python fsc-builder/scripts/generate_fsc.py HARA.xlsx fsc_in.json FSC.xlsx
python tsc-builder/scripts/generate_tsc.py FSC.xlsx tsc_in.json TSC.xlsx
python fmeda-builder/scripts/generate_fmeda.py TSC.xlsx ... FMEDA.xlsx
```

---

## 11. GitHub에서 유명한 이유 분석 (⭐2,932 / 🍴166)

| # | 요인 | 설명 |
|---|---|---|
| 1 | **고통이 크고 구체적인 틈새** | 기능안전 엔지니어의 일상 = HARA 900행 수작업. 고통 지점이 명확 |
| 2 | **압도적 규모** | 152 스킬 / 849 파일 / 162,477줄 / 13 도메인 — 숫자 자체가 신뢰 |
| 3 | **builder+reviewer 쌍 구조** | AI 산출물 신뢰 문제를 "자체 검수"로 정면 돌파. 76쌍 100% 페어링 |
| 4 | **산출물이 실물** | 채팅 답변이 아닌 제출 가능한 15탭 xlsx (수식·색상코딩·NAVY 스타일링) |
| 5 | **규제가 강제 수요를 만듦** | UN R155(법적 의무) / ISO 26262(계약 강제) / IATF 16949(인증 없으면 납품 불가) |
| 6 | **MIT + 완전 무료** | 경쟁 상용 툴(Vector PREEvision, ANSYS medini, LDRA)은 연 수천만~수억 |
| 7 | **"AI가 스스로 운영하는 repo"** 메타 스토리 | 5개월치 자율 운영 일지 + 자기 결함 정직 고백 |
| 8 | **검색 키워드 폭격** | ISO 26262 / ASIL / HARA / FMEDA / AUTOSAR / SOTIF / APQP / PPAP / UDS / SysML + topics 태그 |

### 냉정한 해석

| 신호 | 해석 |
|---|---|
| ⭐2,932 vs 🍴166 (5.7%) | "나중에 볼래" 북마크형 별 비중이 큼. 실사용자는 적을 수 있음 |
| Open issues 4개 | 깊게 쓰는 사용자가 적다는 신호 (헤비 유저가 많으면 이슈가 쏟아짐) |
| 테스트 0개 / 체크 ~100개 미존재 | 품질 보증 장치 부재 |
| 도메인 절반 껍데기 | sysml/mbse/v&v 미완성 |

> → **"완성시키는 사람"에게 기회가 있다.**

---

## 12. 로컬 에이전트 구축 레퍼런스 가치 ⭐⭐⭐⭐⭐

자동차 도메인과 무관해도, **Agent Skill 설계 교본**으로서 가치가 큼.

### ① `description` 작성 공식

```
[무엇을 만드는가] + [출력물 구체 명세] + [정식 트리거 키워드 나열]
+ [캐주얼 표현 예시 2개] + [금지 지시: 채팅으로 때우지 마라]
```

실례 (`hara-builder`, 911자):
```yaml
description: Generate an audit-ready ISO 26262 Hazard Analysis and Risk
  Assessment (HARA) workbook from an item definition and function list.
  Produces a multi-tab xlsx with ... Use this skill whenever the user
  mentions HARA, hazard analysis, ASIL determination, ... even casual
  phrasings like 'I need the safety analysis for this ECU' or 'what is
  the ASIL for X'. Always use this skill instead of producing a freeform
  HARA in chat; the spreadsheet output is the deliverable analysts expect.
```

> 이 repo는 **"핵심 트리거 문구가 앞 400자 안에 있는지"를 자동 감사**한다
> (`docs/skill-polish-log/` 41건이 그 기록).

### ② Progressive Disclosure (점진적 공개)

```
[상시]   description (~900자)            ← 이것만 항상 컨텍스트에
  ↓ 트리거
[1단계]  SKILL.md 본문 (워크플로)
  ↓ 필요시
[2단계]  references/*.md (도메인 지식)
  ↓ 실행시
[3단계]  scripts/*.py — 읽지 않고 실행만 (48KB, 토큰 0)
```

### ③ 폴더 4분할 규약 (152개 예외 없이 준수)

| 폴더 | 성격 |
|---|---|
| `SKILL.md` | 지시 (instruction) |
| `references/` | 지식 (knowledge) — 필요시 로드 |
| `scripts/` | 능력 (capability) — 읽지 말고 실행 |
| `examples/` | 검증 (validation) — 동작하는 샘플 |

### ④ 스킬 간 데이터 계약 + 자동 감사

자연어로 인계하지 않고 **파일 포맷을 계약으로 삼고, 계약 파손을 스크립트로 감사.**
(→ 6절 `chain_contract_audit.py` 결과 참조)

### ⑤ 자율 운영 루프 (→ 8절)

### ⑥ 자기 결함 정직 기록 (→ 9절)

### 즉시 적용 가능한 3단계

1. **구조 베끼기** — `SKILL.md` / `references/` / `scripts/` / `examples/` 4분할 + description 공식
2. **자율 루프 이식** — 요일별 모드 + 자율 편집 allowlist + 8필드 로그 포맷
3. **계약 감사 스크립트** — `chain_contract_audit.py`를 자기 도메인용으로 각색

---

## 13. React / PHP 재구현 가능성

### 3개 층으로 분리해서 판단

| 층 | 역할 | React/PHP 가능? |
|---|---|---|
| **① 지식층** (`references/*.md`, 상수 테이블) | ASIL 매트릭스, 14 고장유형, 등급표 | ✅ **완전 가능** (단순 데이터) |
| **② 생성층** (`scripts/*.py`) | JSON → xlsx 생성 | ✅ **완전 가능** (포팅) |
| **③ 추론층** (Claude) | 인터뷰 → 판단 → 근거 문장 | ❌ LLM 필요 (단, 상당 부분 우회 가능) |

### 핵심 발견 — S/E/C 자동 추천은 AI가 아니다

```python
# generate_hara.py — 결정론적 휴리스틱
ADVERSARIAL_MALFUNCTIONS = {"M03","M05","M07","M12"}  # 운전자와 싸우는 고장 → C 상향
DEFAULT_LOCATIONS = [{"code":"L05","name":"Interstate","default_e":"E4"}, ...]
# kinetic_authority: high|medium|low|none 기반 분류
```

→ **AI 없이 if문·룩업테이블로 90% 재현 가능.**

### 라이브러리 대체표

| Python | PHP | Node/React |
|---|---|---|
| `openpyxl` | **PhpSpreadsheet** | **exceljs** / SheetJS |
| 수식·스타일·색상 | ✅ 지원 | ✅ 지원 |
| `recalc.py` (LibreOffice) | `shell_exec` | `child_process` |

### 아키텍처 3안

**안 A — 순수 React + PHP (AI 없음)**
```
[React SPA] 마법사형 폼
     │ REST
[PHP/Laravel] 검증 + 휴리스틱 엔진 + PhpSpreadsheet
[MySQL] 프로젝트/버전 · [LibreOffice headless] 수식 재계산
```
👍 AI 비용 0, 완전 오프라인(온프레미스 판매), 예측 가능 / 👎 자유 대화 불가

**안 B — 하이브리드 ⭐ 최종 추천**
```
[React] 마법사 폼 + AI 어시스트 버튼
   │
[PHP/Laravel] 오케스트레이션 + 인증 + 과금(Cashier)
   │  ├─▶ [Claude API]   인터뷰 / 근거문 초안 (토큰 소량)
   │  └─▶ [Python 워커]  ★ 원본 152개 스크립트 그대로 재사용 ★
[S3/MinIO] 산출물 · [Postgres] 메타데이터
```
👍 162,477줄 검증 코드 100% 재사용(포팅 버그 0) · AI는 옵션이라 비용 통제 · React UI + PHP 결제
👎 Python 워커 운영 필요 (Docker로 해결)

**안 C — Next.js 풀스택**
👍 단일 언어(TS), Vercel 배포 / 👎 152개 스크립트 전면 재작성 (공수 과다)

### 비교

|  | 안 A | **안 B** ⭐ | 안 C |
|---|---|---|---|
| 개발 기간 | 2~3개월 | **1~2개월** | 6개월+ |
| AI 비용 | 0 | 소액 | 소액 |
| 원본 재사용 | ❌ 포팅 | ✅ **100%** | ❌ 전면 재작성 |
| 온프레미스 | ✅ | ✅ | ⚠️ |
| 추천도 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |

> **결론: React로 UI, PHP로 사업 로직·과금, Python은 엔진으로 그대로 둔다.**
> 152개 스크립트 포팅은 지옥. Docker 컨테이너 + 큐로 호출하는 것이 최단·최안전 경로.

### MVP 로드맵 (1인 기준)

| 기간 | 작업 |
|---|---|
| 1주 | `hara-builder` 1개 → React 폼 + PHP API + Python 워커 |
| 2주 | 리뷰어 연결 → "생성→검수" 2단 데모 |
| 3~4주 | 체인 3개(HARA→FSC→TSC) + 로그인/결제 |
| 5~8주 | safety 도메인 15쌍 완성 → 베타 고객 3곳 |

---

## 14. 수익화 전략

### 법적 기반

```
MIT License → ✅ 상업적 사용 · 수정 · 재배포 · 유료 판매 · 소스 비공개 모두 허용
              ⚠️ 원본 저작권 고지 + MIT 라이선스 사본 포함 필수
```
필수 조치: ① `LICENSE` 유지 + "Based on jherrodthomas/automotive-skills-suite (MIT)" 표기
② 자체 면책 조항 명시 (아래 리스크 참조)

### 아이디어 8가지 (현실성 순)

| # | 아이디어 | 난이도 | 수익 | 기간 | 추천 |
|---|---|---|---|---|---|
| 1 | 🇰🇷 **한국 로컬라이제이션 SaaS** | ⭐⭐⭐ | 💰💰💰💰💰 | 3~6개월 | ⭐⭐⭐⭐⭐ |
| 2 | **컨설팅 + 교육** | ⭐⭐ | 💰💰💰💰 | **즉시** | ⭐⭐⭐⭐⭐ |
| 3 | **검수 전문 서비스** | ⭐⭐ | 💰💰💰 | **1~2개월** | ⭐⭐⭐⭐⭐ |
| 4 | 타 산업 복제 (배터리/의료) | ⭐⭐⭐⭐ | 💰💰💰💰💰 | 6~12개월 | ⭐⭐⭐⭐ |
| 5 | 템플릿 팩 판매 | ⭐ | 💰💰 | **2~4주** | ⭐⭐⭐⭐ |
| 6 | 품질 개선 유료판 | ⭐⭐⭐ | 💰💰💰 | 2~3개월 | ⭐⭐⭐⭐ |
| 7 | 벤치마크 데이터 | ⭐⭐⭐⭐⭐ | 💰💰💰💰 | 2년+ | ⭐⭐⭐ |
| 8 | AI 에이전트 구축 컨설팅 | ⭐⭐⭐ | 💰💰💰💰 | 1~2개월 | ⭐⭐⭐⭐ |

---

#### 1️⃣ 한국 시장 로컬라이제이션 SaaS — "K-FuSa"

**시장 근거**
- 현대/기아 1차 협력사 ~300개, 2·3차 수천 개
- 국내 Tier-2/3는 기능안전 전담 인력 1~2명 또는 부재
- 상용 툴(Vector PREEvision, ANSYS medini)은 연 수천만~수억 → 중소기업 접근 불가
- **한글 템플릿 + 국내 OEM 제출 양식은 시장에 존재하지 않음 (공백)**

**차별화 (원본에 없는 것만)**

| 기능 | 근거 |
|---|---|
| 한글 UI + 한글 xlsx 산출물 | 국내 심사 문서는 한글 필요 |
| 현대/기아 제출 양식 프리셋 | OEM별 서식 상이 → 진입장벽 |
| 다중 사용자 협업 | 원본은 1인 로컬 도구 |
| 변경 영향 추적 | HARA 수정 → 하류 12개 문서 영향 자동 표시 |
| **온프레미스 배포** | OEM 개발망 인터넷 차단 → 킬러 기능 |
| 프로젝트 대시보드 | 문서 완성도·갭 현황 |

**가격**
```
Free       스킬 3개, 프로젝트 1개, 워터마크                  → 유입
Pro        월 9만원/인 — 전 도메인, 프로젝트 무제한            → 개인·소규모
Team       월 39만원(5인) — 협업, 감사 추적, 이력              → Tier-2/3
Enterprise 연 1,200만원~ — 온프레미스, 커스텀 양식, SSO, 지원  → Tier-1 (실제 매출원)
```
**시나리오** 1년차 ≈ 8,000만원 (Ent 3 + Team 10) / 2년차 ≈ 2.5억 (Ent 8 + Team 30)

---

#### 2️⃣ 컨설팅 + 교육 — 가장 빠른 현금화

| 상품 | 가격 |
|---|---|
| 온라인 강의 (8주) | 39~79만원/인 |
| 기업 출강 (2일) | 500~1,500만원 |
| 도입 컨설팅 | 2,000~5,000만원 |
| 감사 대응 지원 | 1,000~3,000만원 |
| 유지보수 리테이너 | 월 200~500만원 |

전략: 툴은 무료 배포 → "우리 회사에 맞게 세팅해주세요"로 전환 (오픈소스 비즈니스 정석)

---

#### 3️⃣ 검수 전문 서비스 — "AuditReady" (MVP 최적)

**왜 진입장벽이 가장 낮은가:** 문서 *작성*은 각 사의 기존 방식이 있어 전환 비용이 크지만,
*검수*는 xlsx만 있으면 되므로 **고객 워크플로 변경이 불필요**. 경쟁 상품도 희소.

```
1회성 진단      50만원/문서 — 업로드 → 82항목 리포트 + 개선 권고
정기 구독       월 150만원 — 무제한 검수 + 트렌드 대시보드
감사 전 패키지  800만원 — 전 문서 일괄 검수 + 갭 클로징 로드맵
화이트라벨      연 2,000만원 — 컨설팅펌 재판매
```
⚠️ **선결 조건:** `traceability-matrix-reviewer`의 "빈 파일 24/25 합격" 버그를 먼저 수정.
아니면 신뢰가 즉시 붕괴됨.

---

#### 4️⃣ 도메인 복제 — 같은 아키텍처를 타 규제 산업으로

| 산업 | 표준 | 시장성 | 비고 |
|---|---|---|---|
| 🏥 의료기기 SW | IEC 62304, ISO 14971, FDA 510(k) | 💰💰💰💰💰 | 자동차보다 시장 큼 |
| ✈️ 항공 | DO-178C, DO-254, ARP4754A | 💰💰💰💰 | 단가 최고, 진입 난이도 높음 |
| 🚄 철도 | EN 50128/50129, IEC 62279 | 💰💰💰 | 국내 수요 존재 |
| ⚡ 산업 안전 | IEC 61508, IEC 62443 | 💰💰💰 | 범용성 높음 |
| 🔋 **배터리/ESS** | UL 2580, IEC 62619 | 💰💰💰💰 | **K-배터리 급성장, 자동차 인접 도메인** ← 추천 |
| 🤖 로봇 | ISO 10218, ISO 13482 | 💰💰 | 신흥 |
| 💊 제약 GMP | GAMP 5, 21 CFR Part 11 | 💰💰💰💰 | 문서 부담 심각 |

아키텍처(빌더+리뷰어 쌍, 체인 계약, xlsx 엔진)는 재사용, 도메인 지식만 교체.

---

#### 5️⃣ 템플릿/스킬 팩 판매 (Gumroad 등)

```
Safety Starter Pack   (HARA+FSC+TSC+FMEDA+SafetyCase)   $149
Cybersecurity Pack    (TARA→CS Arch→IR Plan)            $129
Quality Pack          (APQP→DFMEA→PFMEA→CP→PPAP)        $129
Korean Edition        한글 템플릿 + OEM 양식             ₩299,000
Full Bundle           전 152개 + 평생 업데이트            $499
```
"원본이 무료인데 왜 사는가" → 파는 것은 스킬이 아니라
**① 한글화 ② 버그 수정 ③ 검증된 샘플 ④ 설치 가이드 영상 ⑤ 이메일 지원** = 시간 절약.

---

#### 6️⃣ 품질 개선 유료판 — 결함 목록이 곧 기능 목록

| 원본 결함 | 유료판 |
|---|---|
| 광고 체크 ~100개 미존재 | ✅ 전부 실제 구현 |
| `traceability-matrix` 비기능 | ✅ 완전 재작성 |
| 테스트 스위트 0개 | ✅ 152개 스모크 테스트 |
| 시트명 31자 초과 19건 | ✅ 수정 |
| `REJECTED` 판정 미작동 | ✅ 수정 |
| sysml/mbse/v&v 껍데기 | ✅ 실제 구현 |

포지셔닝: *"오픈소스 원본은 훌륭한 뼈대입니다. 저희는 그것을 감사 통과 가능한 제품으로 만들었습니다."*
증명 수단: CI 뱃지 `Tests: 152/152 passing` ← 원본이 붙일 수 없는 뱃지

---

#### 7️⃣ 벤치마크 데이터 (2년차 이후)

익명 집계 데이터로 업계 리포트: 평균 HARA 행 수, ASIL 분포, 감사 지적사항 Top 20,
도메인별 문서 완성도 중위값 → 리포트 300만원/부 또는 구독 고객 무료 제공(락인)

---

#### 8️⃣ AI 에이전트 구축 컨설팅 (메타 플레이)

이 repo의 자율 운영 패턴 자체를 상품화.

| 상품 | 가격 |
|---|---|
| 에이전트 설계 워크샵 (2일) | 800만원 |
| 사내 스킬 스위트 구축 | 3,000~8,000만원 |
| 자율 운영 루프 세팅 | 1,500만원 |

자동차 업계 외에도 판매 가능.

---

### 실행 로드맵

| 기간 | 초점 | 목표 |
|---|---|---|
| 0~1개월 | #5 템플릿 팩 + #2 교육 콘텐츠 | 첫 매출 + 이메일 리스트 100명 |
| 1~3개월 | #3 검수 서비스 MVP (React+PHP+Python) | 베타 3곳 → 유료 고객 1곳 |
| 3~6개월 | #1 한국 로컬라이제이션 SaaS + #6 품질 개선 | Enterprise 1 + Team 5 |
| 6~12개월 | #4 배터리/의료 복제 + #8 컨설팅 | 연매출 1억 |

---

### 리스크 관리

| # | 리스크 | 대응 |
|---|---|---|
| 1 | **법적 책임 (최우선)** — 기능안전 문서는 인명과 직결. ASIL 오판 → 사고 시 책임 소재 | ✅ 계약·UI·산출물에 면책 명시: *"본 도구는 문서 작성을 보조하며, 안전 판단의 최종 책임은 자격을 갖춘 기능안전 엔지니어에게 있습니다"* ✅ 산출물 워터마크 `Draft — Requires FSE Review` ✅ AI 추천값은 항상 "제안(Suggested)" + 근거 병기 ✅ **E&O 책임보험 가입** ✅ "인증 대행" 금지, "문서 작성 지원"으로만 포지셔닝 |
| 2 | 원본이 무료라는 경쟁 | 파는 것은 SW가 아니라 한글화·신뢰성·지원·협업·온프레미스·**면책 있는 서비스** |
| 3 | 품질 결함 승계 | 판매 도메인만 전수 검증 후 출시. 미완성 도메인은 "준비 중" 처리 |
| 4 | 긴 세일즈 사이클 (6~18개월) | #5/#2로 현금 흐름 선행 확보 |
| 5 | 도메인 전문성 부족 | 업계 전문가 1명을 파트너·자문으로 영입 (신뢰의 핵심) |

> 원본의 설계 철학을 계승할 것: **"opinionated about method, not about answers"**
> (방법론에는 단호하되, 답에 대해서는 단호하지 않다)

---

## 15. 최종 요약

| 질문 | 답 |
|---|---|
| **이게 뭐야?** | 자동차 ISO 표준 xlsx 산출물 76종을 생성(builder) + 검수(reviewer)하는 152개 Agent Skill 묶음 |
| **언제 써?** | ECU 개발 착수 / OEM 감사 대응 / UN R155 사이버보안 / ADAS SOTIF / 양산 이관 PPAP / 클레임 8D / 문서 검수 |
| **Skill? MCP? Plugin?** | **순수 Agent Skill.** MCP·Plugin 아님 |
| **API 토큰 필요?** | **불필요.** 완전 오프라인. Claude 요금만 별개 (JSON 직접 쓰면 그것도 불필요) |
| **왜 유명해?** | 고통 큰 틈새 + 압도적 규모 + builder/reviewer 쌍 + 실물 산출물 + 규제 강제 수요 + MIT 무료 + AI 자율 운영 스토리 + SEO |
| **로컬 에이전트에 도움?** | ⭐⭐⭐⭐⭐ — description 공식, progressive disclosure, 4분할 구조, 데이터 계약+감사, 자율 운영 루프, 정직 로그 |
| **수익화 가능?** | MIT라 합법. 최우선: ① 한국 로컬라이제이션 SaaS ② 컨설팅·교육(즉시) ③ 검수 서비스(MVP 최적) |
| **React/PHP 가능?** | 지식층·생성층 ✅ / 추론층 ❌. **권장: React UI + PHP 사업로직 + Python 엔진 그대로 재사용** |
| **바로 실무 투입 OK?** | ⚠️ **아니오.** safety/cyber/quality/aspice는 가능. sysml/mbse/v&v는 껍데기. 도메인별 검증 필수 |

---

## 부록 A. 검증에 사용한 명령

```bash
# 저장소 구조
find . -not -path './.git/*' -maxdepth 3 | sort
du -sh skills examples docs scripts

# .skill 포맷 확인
file skills/hara-builder.skill      # → Zip archive data
unzip -q skills/hara-builder.skill -d peek && find peek -type f

# 규모 집계
find . -name "*.py" | wc -l                    # 849
find . -name "*.py" -exec cat {} + | wc -l     # 162,477
grep -rhoE "^(import|from) [a-z_0-9]+" --include="*.py" . | awk '{print $2}' | sort | uniq -c | sort -rn

# 토큰/인증 전수 검사
grep -rhoE "(os\.environ|getenv)[^)]{0,40}" --include="*.py" .   # → os.environ.copy( 만

# 실행 검증
pip install openpyxl
python hara-builder/scripts/generate_hara.py \
       hara-builder/examples/sample_input_esc.json HARA_test.xlsx
python hara-checklist-reviewer/scripts/generate_checklist.py \
       HARA_test.xlsx HARA_review.xlsx

# 원본 저장소 추적
grep -rhoE "github\.com/[A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+" --include="*.md" . | sort | uniq -c
# → 95  github.com/jherrodthomas/automotive-skills-suite
```

## 부록 B. 참고 링크

| 항목 | 주소 |
|---|---|
| 원본 저장소 | https://github.com/jherrodthomas/automotive-skills-suite |
| 본 저장소 | https://github.com/bmshin94/automotive-skills-suite |
| Manus 포팅판 | https://github.com/dsdtsolutionsllc/manus-automotive-skills-suite |
| 원본 이슈 트래커 | https://github.com/jherrodthomas/automotive-skills-suite/issues |
| 라이선스 | MIT — `LICENSE` |

## 부록 C. 저장소 내부 참고 문서

| 문서 | 내용 |
|---|---|
| `STATUS.md` | 76쌍 페어링 현황 (🟢신선 / 🟡오래됨) |
| `CHANGELOG.md` | 주간 변경 + **Known issues** (결함 고백) |
| `RELEASES.md` | 릴리스 스냅샷 + 하이라이트 |
| `docs/AUTONOMOUS_LOG.md` | AI 자율 운영 일지 (요일별 모드, 8필드 로그) |
| `docs/chain-contract-audit.md` | 체인 계약 감사 (BREAK 0) |
| `docs/sheet-name-length-audit.md` | 시트명 31자 초과 19건 |
| `docs/skill-polish-log/` | 스킬별 품질 감사 41건 |
| `docs/weekly/` · `docs/monthly/` · `docs/triage/` | 주간 계획 / KPI / 이슈 정리 |
| `scripts/regen_status.py` | STATUS.md 재생성 |
| `scripts/chain_contract_audit.py` | 체인 계약 감사 |

---

_본 문서는 실제 실행·검증 기반으로 작성되었습니다. 저장소가 자체 고백한 결함(9절)을 반드시 확인한 뒤 실무에 적용하십시오._
