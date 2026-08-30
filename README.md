# expert-panel-ebp-plus

근거 기반(Evidence-Based Practice) 전문가 패널 토론 스킬 for [Claude Code](https://claude.com/claude-code).

[itsbluetic/expert-panel-by-ebp](https://github.com/itsbluetic/expert-panel-by-ebp) v3.1.0을 역공학 리뷰한 뒤, 그 리뷰에서 나온 개선점을 반영한 포크입니다. 원본의 8단계 구조·근거 태그 체계·설계 철학은 그대로 유지하고, 실행 시 발견된 내부 모순과 근거 검증 가능성 위주로 손을 댔습니다. 원본 저장소가 살아있는 한 그쪽이 정본이며, 이 저장소는 별도 파생판입니다.

주어진 주제에 대해 **관련 학문 분야의 근거를 먼저 탐색**한 후, 해당 근거에 기반한 6인 전문가 패널이 토론하고, 사용자 맥락에 맞춘 최소 실험(MVE)까지 설계합니다.

> "전문가가 이렇다고 해서 무조건 따르지 않는다. 근거 없는 전문가 조언은 안 하느니만 못할 수 있다." — EBP 원칙

## 왜 만들었나

LLM에게 "전문가처럼 답해줘"를 시키면 권위 인용에 의존한 그럴듯한 답변이 나오기 쉽습니다. 이 스킬은 그 단계를 분리해 (1) **근거를 먼저 깔고** → (2) 그 위에서 전문가가 토론하게 하고 → (3) **사용자 자신의 맥락에 맞춘 최소 실험**으로 닫습니다.

## 6축 매핑 (Harness Engineering)

| 축 | 어디서 적용되나 |
|----|----------------|
| **구조** | 7단계 layered phase + 단계별 ✓ Gate + 미달 시 fallback |
| **맥락** | 3단계 — `AskUserQuestion`으로 사용자 상황 수집 |
| **계획** | 1-2단계 — 학문 지도 → 근거 지형 (탐색 전 설계) |
| **실행** | 4-6단계 — 근거 태그 강제, Core/Peripheral 분리, MVE 설계 |
| **검증** | ✓ Gate + Hard Rules + Checklist Before Stopping |
| **개선** | 7단계 저장 → PBE(Practice-Based Evidence) 루프로 N-of-1 축적 |

## 설치

폴더 기반 SKILL.md 형식이라 `~/.claude/skills/`에 복사만 하면 끝입니다.

```bash
# 사용자 전역 설치
cp -r skills/expert-panel ~/.claude/skills/

# 또는 프로젝트 로컬 설치
cp -r skills/expert-panel .claude/skills/
```

## 사용법

```
/expert-panel [주제]                          → 전체 8단계(0-7) 실행
/expert-panel [주제] --quick                  → 학문지도·근거지형·깊은제약(1-3단계) 생략, Mirror(0)만 유지
/expert-panel [주제] --experts [전문가1, ...]  → 전문가 직접 지정
```

트리거 키워드: `전문가소환`, `전문가 패널`, `expert panel`, `근거 기반 분석`, `EBP 분석`.

## 8단계 흐름 (분할 User Gate)

0. **Goal Mirror & 현재 상황** — 텍스트 mirror + `AskUserQuestion` 1개 *(Pre-flight Gate)*
1. **학문 지도** — 주제 관련 학문 분야 2-4개 식별, 주 근거원 선택
2. **근거 지형** — WebSearch로 메타분석/체계적 리뷰 탐색 + WebFetch로 원문 검증, 5단계 태그와 원문 URL 부여
3. **깊은 제약 수집** — 자원·도구·측정기준 `AskUserQuestion` *(Deep Gate)*
4. **패널 토론** — 6인 전문가, 근거 태그(+ URL) 동반 4라운드, 민감 주제는 실명/아키타입 선택 가드
5. **구현 설계** — Core/Peripheral Practice 분리, 사용자 맥락 적응
6. **실천 실험** — MVE + 성공/실패 기준 + 반복 주기
7. **저장** — `AskUserQuestion`으로 저장 위치 확인, 외부 저장 커맨드 미가용 시 자동 폴백 *(User Gate)*

각 단계 종료 시 `✓/✗ Gate [N]` 한 줄을 실제로 출력해 통과 여부를 눈으로 확인할 수 있게 합니다 (내부 판단만으로 넘어가지 않음).

> **분할 게이트 설계**: harness `specify`/`scaffold`의 "L0 Goal → L1 scan → L2 deep interview" 패턴 차용. 0단계가 1-2단계의 학문/근거 탐색을 사용자 맥락에 좁혀주고, 3단계는 근거를 본 다음에야 의미 있는 답을 얻을 수 있는 자원·제약·측정기준만 수집.

## 근거 수준 체계

| 태그 | 의미 |
|------|------|
| **[Strong]** | 메타분석, 체계적 리뷰, 복수의 재현된 실험 |
| **[Moderate]** | 복수의 RCT 또는 대규모 관찰 연구 |
| **[Emerging]** | 단일 연구, 예비 결과, 파일럿 스터디 |
| **[Expert]** | 전문가 합의, 임상 경험 기반 (근거 약함 — 경고 동반) |
| **[Contested]** | 상반된 연구 결과가 존재 |

학술적 배경: `skills/expert-panel/references/evidence-framework.md` (OCEBM Levels of Evidence 기반).

## 예시

`examples/EBP_타이핑vs말하기속도비교.md` — 실제 산출물.

**같은 입력으로 재현하기**:
```
/expert-panel 타이핑 vs 말하기 속도 비교
```
3단계 `AskUserQuestion`에 다음과 같이 응답:
- 현재 상황: "측정 방법이 통일되지 않은 상태, 본인 기준선 없음"
- 원하는 변화: "동일 단위로 직접 비교 측정"

→ Speech Science + HCI 도메인 식별 → Pellegrino, Ruan, Dhakal, 신지영 등 6인 패널 → "완성 음절/분 통일 + 총 시간 기반 측정" 합의 도출.

## 폴더 구조

```
expert-panel-ebp-plus/
├── README.md                          # 이 파일
├── CHANGELOG.md                       # 버전별 변경 이력
├── LICENSE                            # MIT
├── skills/
│   └── expert-panel/
│       ├── SKILL.md                   # 메인 스킬 (v3.3.0)
│       └── references/
│           ├── evidence-framework.md  # OCEBM 기반 근거 수준 학술 배경
│           └── implementation-guide.md # Implementation Science / PBE 심화
├── examples/
│   └── EBP_타이핑vs말하기속도비교.md   # 실제 산출물 예시 (v3.1.0 기준, URL 인용 규칙 이전)
└── legacy/
    └── 전문가소환.md                   # 최초 배포판 (파일 내부 표기 v2.0, 참조용)
```

## v3.3.0 변경점 (vs v3.2.0)

질문 방식 조정: 필요한 정보는 짐작하지 말고 `AskUserQuestion`으로 묻되, 한 호출에 질문을 몰아넣지 않고 하나씩 순차 진행 + 답변 후 더 깊은 정보가 필요하면 후속 질문으로 이어가도록 명시 (Hard Rule #8). 자세한 배경은 `CHANGELOG.md` 참조.

## v3.2.0 변경점 (vs v3.1.0)

원본 저장소를 역공학 리뷰하며 발견한 7가지 지점을 반영했습니다. 자세한 배경은 `CHANGELOG.md` 참조.

1. **`--quick` 모드 사양 모순 수정** — Dispatch 표(`▶[1/5]`)와 상세 섹션(`▶[0/4]`)이 서로 다른 단계 구성을 가리키던 것을 `Quick Flow (0→4→5→6→7)`로 통일
2. **근거 인용에 원문 URL 필수화** — `[Strong]`/`[Moderate]` 태그는 원문 링크 없이는 부여 불가(하향 또는 `(원문 미확인)` 병기)
3. **`allowed-tools`에 `WebFetch` 추가** — 검색 스니펫만으로 태그를 매기지 않고 원문을 열어 저자·연도·수치를 대조
4. **실명 인물 시뮬레이션 가드 강화** — 정책/법률/의료 등 민감 주제에 실명/아키타입 선택 질문 추가, 시뮬레이션 고지를 저장물까지 유지
5. **Obsidian 저장 외부 의존성 명시 + 자동 폴백** — `/obsidian-save` 미설치 시 조용히 실패하지 않고 로컬 파일 저장으로 폴백, 사용자에게 안내
6. **Gate를 출력형 자기검증으로 전환** — 각 단계 끝에 `✓/✗ Gate [N]` 한 줄을 실제로 출력하도록 강제 (Hard Rule #6)
7. **버전 라벨 정리** — `legacy/전문가소환.md`의 자체 표기(v2.0)와 실제 위상(최초 배포판) 간 혼동을 README에서 명확화

## Credits

- [itsbluetic/expert-panel-by-ebp](https://github.com/itsbluetic/expert-panel-by-ebp) — 원본 스킬 저자(anmen), v3.1.0까지의 8단계 구조·근거 태그 체계·6축 매핑 설계
- 김창준 — EBP 접근법과 "근거 없는 전문가 조언의 위험성"에 대한 사고의 영감 (원본 Credits에서 계승)
- team-attention/harness — 6축(구조·맥락·계획·실행·검증·개선) 사이클 프레임워크 (원본 Credits에서 계승)

## License

MIT — 원본 저장소도 MIT로 명시되어 있었으나 LICENSE 파일이 없어, 이 저장소에는 원 저자 표기를 포함한 LICENSE 파일을 추가했습니다.
