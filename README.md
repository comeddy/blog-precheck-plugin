# blog-review 플러그인 (blog-precheck 스킬)

AWS 테크 블로그 저자가 리뷰를 요청하기 전에 본인 원고(docx 또는 md)를 먼저 점검하는 Claude Code 플러그인입니다. 원고를 6개 기준으로 채점하고, WordPress에 붙여넣을 HTML로 변환하고, 저자가 고칠 코멘트 목록을 만듭니다. WordPress에는 아무것도 올리지 않습니다.

## 설치

Claude Code에서 마켓플레이스를 추가한 뒤 플러그인을 설치합니다.

```
/plugin marketplace add comeddy/blog-precheck
/plugin install blog-review@blog-precheck
```

터미널에서는 다음과 같이 설치합니다.

```bash
claude plugin marketplace add comeddy/blog-precheck
claude plugin install blog-review@blog-precheck
```

업데이트는 `/plugin marketplace update blog-precheck`로 받습니다.

docx 원고를 다루려면 `python-docx`가 필요합니다 (`pip install python-docx`).

## 사용

설치하면 `/blog-review:blog-precheck` 스킬이 생깁니다. 원고 경로를 함께 넘기거나, "블로그 사전 점검", "리뷰 전 점검", "워드프레스 변환"처럼 요청하면 됩니다.

```
/blog-review:blog-precheck ~/Downloads/my-draft.docx
```

진행 순서는 다음과 같습니다.

1. **게이트 확인**: 작업 폴더에 `gate-report.md`가 없으면 아래 6개 기준으로 채점해 만듭니다.
2. **원고 추출**: docx는 python-docx로 구조를 추출합니다.
3. **구조 변환**: WordPress 코드 편집기에 붙여넣을 HTML을 `wordpress.md`로 만듭니다. 이미지는 `images/`에 번호를 붙여 정리합니다.
4. **내용 리뷰**: 같은 주제의 블로그를 검색해 비교하고 저자용 코멘트를 씁니다.
5. **게시 후**: 게시가 끝났다고 알리면 Slack 공지 문구를 씁니다.

### 산출물

```
<작업 폴더>/<YYYY-MM-DD>-<블로그-타이틀>/
├── src/              # 원본 원고
├── wordpress.md      # WordPress 붙여넣기용 변환본
├── review.md         # 저자가 고칠 코멘트
├── gate-report.md    # 셀프 게이트 채점 결과
└── images/           # 본문 이미지 (리뷰어가 그대로 업로드)
```

## 셀프 게이트 기준 (gate-report.md)

원고를 리뷰어에게 넘기기 전에 6개 기준으로 점수를 매기고, 통과 조건에 못 미치면 저자가 먼저 고칩니다. 전체 가이드는 [gate.md](plugins/blog-review/skills/blog-precheck/gate.md)에 있습니다.

### 통과 조건

- 총점(6개 평균)이 70 이상이어야 합니다.
- 모든 항목이 최저선 이상이어야 합니다. 총점이 높아도 한 항목이 최저선에 못 미치면 반려입니다.
- 자기검열은 최대 2회까지 반복합니다. 2회 후에도 미달이면 미달 항목을 `gate-report.md`에 적은 채 리뷰어에게 넘깁니다.

### 기준 6개

| # | 기준 | 측정 방식 | 최저선 |
|---|---|---|---|
| 1 | 정합성 | 그림/표 번호가 등장 순서와 일치, 본문 수치와 표·그림·코드 수치 재계산 일치, 본문 심볼과 코드 심볼 일치, 링크 전부 200, 아키텍처 흐름 번호 참조 일치 | 80 |
| 2 | 구조 완결성 | 체크리스트 7개 각 약 14점: 도입(문제 제기), 솔루션 개요, 실전 단계, 결과 증거(캡처/수치), 리소스 정리, 비용 언급, 다음 단계·CTA | 70 |
| 3 | 표기 규범 | 서비스 정식명(첫 등장 시 Amazon/AWS 접두사), AWS 한국어 공식 용어, 용어 혼용 전수 검색, 용어 3층위(기능명·문서명·API값) 구분, 섹션 번호 하드참조 없음 | 70 |
| 4 | 중복과 군더더기 | `scripts/redundancy_metrics.py` 실행 결과에서 재진술 1건당 5점 감점 | 60 |
| 5 | 차별성과 근거 | 유사 블로그 3개 이상 확인 후 차별점이 본문에 한 문장으로 있는가(40), 실측 데이터·실패 경험이 있는가(40), 사용 모델·버전 등 재현 정보가 있는가(20) | 60 |
| 6 | 가독성 | 비문(주어-서술어 호응) 0건(40), 번역투 표현 3건 이하(30), 한 문단 5문장 이하(30) | 60 |

### 채점 원칙

- 1, 3, 4번은 스크립트와 전수 검색으로 재현할 수 있는 값만 씁니다.
- 2, 5, 6번은 판단이 들어가므로 근거 문장을 인용해서 점수를 적습니다. 근거 없는 점수는 무효입니다.
- 감점 사유는 "원문 인용 → 수정안" 쌍으로 남깁니다.

### 중복과 군더더기 지표

```bash
python3 -I plugins/blog-review/skills/blog-precheck/scripts/redundancy_metrics.py <원고.docx|.md>
```

보일러플레이트 문단 반복, 문장 반복, 메타 강조 문장, 따옴표 조어 밀도, 6문장 이상 문단, 상투어 밀도를 측정합니다. 같은 뜻을 다른 문장으로 되풀이하는 재진술은 스크립트가 잡지 못하므로 채점자가 직접 셉니다. AI가 썼는지가 아니라 읽기에 방해되는지만 봅니다.

### gate-report.md 형식

```markdown
# Gate Report: <블로그 제목>
회차: 1 / 2 | 판정: 통과 / 반려 / 2회 초과 전달

| 기준 | 점수 | 최저선 | 판정 |
|---|---|---|---|
| 정합성 | 85 | 80 | 통과 |
| ... |
| **총점** | 71 | 70 | **반려 (중복과 군더더기 미달)** |

## 감점 사유 (원문 인용 → 수정안)
### 중복과 군더더기 (55)
> (원문)
→ (수정안) — 이유

## 중복과 군더더기 지표 원본
(스크립트 출력 붙여넣기)
```

### 자기검열 루프

1. 채점하고 `gate-report.md`를 저장합니다.
2. 미달 항목의 감점 사유만 고칩니다. 통과한 항목은 건드리지 않습니다.
3. 다시 채점하고 회차를 올려 리포트를 갱신합니다.
4. 통과하거나 2회를 넘기면 리뷰어에게 넘깁니다.

게이트를 통과한 원고의 리뷰어는 오타, 정합성, 문체를 다시 보지 않고 기술적 정확성과 고객 관점에 집중합니다.

## 저장소 구조

```
.claude-plugin/marketplace.json           # 마켓플레이스 정의
plugins/blog-review/
├── .claude-plugin/plugin.json            # 플러그인 정의
└── skills/blog-precheck/
    ├── SKILL.md                          # 워크플로우 본문
    ├── gate.md                           # 셀프 게이트 가이드
    └── scripts/redundancy_metrics.py     # 중복과 군더더기 지표
```

## 출처

리뷰어용 [ssMinji/skills](https://github.com/ssMinji/skills)의 blog-review 플러그인을 바탕으로 저자용으로 고친 버전입니다. 채점 기준과 변환 규칙은 같고, WordPress 미디어 업로드 단계만 뺐습니다.
