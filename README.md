# blog-precheck-plugin

AWS 기술 블로그 저자용 사전 점검 스킬 blog-precheck을 Claude Code 플러그인으로 배포하는 저장소입니다. 마켓플레이스를 추가한 뒤 플러그인을 설치하면 됩니다.

> 이 저장소는 개인 프로젝트이며 Amazon Web Services(AWS)의 공식 도구나 제품이 아닙니다. AWS가 보증하거나 지원하지 않습니다.

## 설치

```
/plugin marketplace add comeddy/blog-precheck-plugin
/plugin install blog-precheck@comeddy
```

터미널에서는 같은 명령을 `claude plugin marketplace add comeddy/blog-precheck-plugin`, `claude plugin install blog-precheck@comeddy`로 실행합니다. 업데이트는 `/plugin marketplace update comeddy`로 받습니다. 설치하면 `/blog-precheck:blog-precheck` 스킬이 생깁니다.

## 사용 예시

설치한 뒤 Claude Code 세션에서 슬래시 명령이나 평소 말투로 요청합니다. docx 원고를 다루려면 `python-docx`가 필요합니다(`pip install python-docx`).

### 1. 원고 전체 사전 점검

원고 경로를 슬래시 명령에 넘깁니다.

```
/blog-precheck:blog-precheck ~/Downloads/my-draft.docx
```

슬래시 명령 없이 이렇게 요청해도 같은 스킬이 실행됩니다.

```
~/Downloads/my-draft.md 리뷰 요청 전에 블로그 사전 점검해줘
```

작업 폴더를 먼저 묻고(정해 둔 곳이 없으면 `~/Documents/Work/Blog/`를 제안), 그 아래 `<YYYY-MM-DD>-<블로그-타이틀>/` 폴더에 `gate-report.md`, `wordpress.md`, `review.md`, `images/`를 만듭니다. 원본 원고는 이 폴더의 `src/`로 옮겨집니다.

### 2. 반려된 항목만 고치고 다시 채점

`gate-report.md` 판정이 반려면 감점 사유를 고친 뒤 다시 요청합니다.

```
gate-report.md 미달 항목 고쳤어. 블로그 사전 점검 다시 해줘
```

미달 항목만 다시 채점하고 회차를 올려 리포트를 갱신합니다. 통과한 항목은 다시 보지 않으며, 재채점은 최대 2회입니다.

### 3. WordPress HTML 변환

```
이 원고 워드프레스 변환해줘: ~/Downloads/my-draft.docx
```

결과물 `wordpress.md`를 WordPress 코드 편집기(HTML 모드)에 통째로 붙여넣습니다. 맨 위 주석에 제목과 추천 영어 URL slug가 들어 있습니다. 이미지 `src`는 `images/01_….png` 같은 로컬 경로로 남고, 미디어 업로드와 URL 치환은 리뷰어가 합니다. `gate-report.md`가 없으면 변환 전에 채점부터 합니다.

### 4. 게시 후 Slack 공지 문구

```
게시 완료했어. 슬랙 publish 문구 써줘
```

작성자 정보(팀·직함, 성함, Slack 핸들)와 게시된 블로그 URL을 먼저 묻습니다. 답하면 바로 붙여넣을 수 있는 3~4문장짜리 완성본을 줍니다.

### 5. 중복과 군더더기 지표만 실행

스킬을 거치지 않고 지표 스크립트만 돌릴 수도 있습니다. 이 저장소를 clone한 폴더에서 실행합니다.

```bash
python3 -I plugins/blog-precheck/skills/blog-precheck/scripts/redundancy_metrics.py my-draft.md
```

보일러플레이트 문단 반복, 문장 반복, 메타 강조 문장, 따옴표 조어 밀도, 6문장 이상 문단, 상투어 밀도를 마크다운 점수표로 출력합니다. 같은 뜻을 다른 문장으로 되풀이하는 재진술은 잡지 못하므로 직접 세어 1건당 5점을 뺍니다.

## 플러그인

| 플러그인 | 설치 | 설명 |
|---|---|---|
| [blog-precheck](plugins/blog-precheck/README.md) | `/plugin install blog-precheck@comeddy` | AWS 기술 블로그 저자가 리뷰 요청 전에 본인 원고를 먼저 점검합니다. 6개 기준 셀프 게이트 채점, WordPress HTML 변환, 내용 리뷰, 게시 후 Slack 공지 문구 작성을 다룹니다. |

### blog-precheck 셀프 게이트 기준 요약

blog-precheck은 원고를 리뷰어에게 넘기기 전에 6개 기준으로 채점해 `gate-report.md`를 만듭니다. 총점(6개 평균)이 70 이상이고 모든 항목이 최저선 이상이어야 통과합니다. 총점이 높아도 한 항목이 최저선에 못 미치면 반려입니다.

| # | 기준 | 보는 것 | 최저선 |
|---|---|---|---|
| 1 | 정합성 | 그림/표 번호 순서, 본문 수치와 표·코드 수치 일치, 심볼 일치, 링크 200 | 80 |
| 2 | 구조 완결성 | 도입, 솔루션 개요, 실전 단계, 결과 증거, 리소스 정리, 비용, 다음 단계·CTA | 70 |
| 3 | 표기 규범 | 서비스 정식명, AWS 한국어 공식 용어, 용어 혼용, 섹션 번호 하드참조 | 70 |
| 4 | 중복과 군더더기 | 스크립트 지표에서 재진술 1건당 5점 감점 | 60 |
| 5 | 차별성과 근거 | 유사 블로그 대비 차별점, 실측 데이터·실패 경험, 재현 정보 | 60 |
| 6 | 가독성 | 비문 0건, 번역투 3건 이하, 한 문단 5문장 이하 | 60 |

미달 항목만 고쳐 다시 채점하고, 이 자기검열은 최대 2회까지 반복합니다. 2회 후에도 미달이면 미달 항목을 리포트에 적은 채 리뷰어에게 넘깁니다. 배점, 채점 원칙, 리포트 형식은 [blog-precheck README](plugins/blog-precheck/README.md#셀프-게이트-기준-gate-reportmd)에 있습니다.

## 저장소 구조

```
.claude-plugin/marketplace.json           # 마켓플레이스 정의
plugins/blog-precheck/
├── .claude-plugin/plugin.json            # 플러그인 정의
├── README.md                             # 사용법과 게이트 기준 전체
└── skills/blog-precheck/
    ├── SKILL.md                          # 워크플로우 본문
    ├── gate.md                           # 셀프 게이트 가이드
    └── scripts/redundancy_metrics.py     # 중복과 군더더기 지표
```

push하기 전에 `claude plugin validate .`로 매니페스트를 검증합니다.

## 라이선스와 출처

[MIT License](LICENSE)로 배포합니다. 이 플러그인은 [ssMinji/skills](https://github.com/ssMinji/skills)의 blog-review 플러그인을 바탕으로 저자용으로 고친 것입니다.
