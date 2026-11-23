# claude-plugins

Claude Code 플러그인 모음입니다. 마켓플레이스를 한 번 추가하면 필요한 플러그인을 골라 설치할 수 있습니다.

## 마켓플레이스 추가

```
/plugin marketplace add comeddy/claude-plugins
```

터미널에서는 `claude plugin marketplace add comeddy/claude-plugins`를 실행합니다. 업데이트는 `/plugin marketplace update comeddy`로 받습니다.

## 플러그인

| 플러그인 | 설치 | 설명 |
|---|---|---|
| [blog-precheck](plugins/blog-precheck/README.md) | `/plugin install blog-precheck@comeddy` | AWS 테크 블로그 저자가 리뷰 요청 전에 본인 원고를 먼저 점검합니다. 6개 기준 셀프 게이트 채점, WordPress HTML 변환, 내용 리뷰, 게시 후 Slack 공지 문구 작성을 다룹니다. |

## 플러그인 추가하기

새 플러그인은 `plugins/<이름>/` 아래에 두고 `.claude-plugin/marketplace.json`의 `plugins` 목록에 한 항목을 더합니다.

```
.claude-plugin/marketplace.json       # 마켓플레이스 정의 (플러그인 목록)
plugins/<이름>/
├── .claude-plugin/plugin.json        # 플러그인 정의
├── README.md
└── skills/<스킬>/SKILL.md
```

push하기 전에 `claude plugin validate .`로 매니페스트를 검증합니다.
