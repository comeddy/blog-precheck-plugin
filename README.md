# blog-precheck-plugin

AWS 테크 블로그 저자용 사전 점검 스킬 blog-precheck을 Claude Code 플러그인으로 배포하는 저장소입니다. 마켓플레이스를 추가한 뒤 플러그인을 설치하면 됩니다.

## 설치

```
/plugin marketplace add comeddy/blog-precheck-plugin
/plugin install blog-precheck@comeddy
```

터미널에서는 같은 명령을 `claude plugin marketplace add comeddy/blog-precheck-plugin`, `claude plugin install blog-precheck@comeddy`로 실행합니다. 업데이트는 `/plugin marketplace update comeddy`로 받습니다. 설치하면 `/blog-precheck:blog-precheck` 스킬이 생깁니다.

## 플러그인

| 플러그인 | 설치 | 설명 |
|---|---|---|
| [blog-precheck](plugins/blog-precheck/README.md) | `/plugin install blog-precheck@comeddy` | AWS 테크 블로그 저자가 리뷰 요청 전에 본인 원고를 먼저 점검합니다. 6개 기준 셀프 게이트 채점, WordPress HTML 변환, 내용 리뷰, 게시 후 Slack 공지 문구 작성을 다룹니다. |

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
