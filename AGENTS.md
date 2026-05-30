# AGENTS.md

GitHub 프로필 README 저장소입니다. GitHub 프로필 페이지에 표시되는 `README.md` 파일을 관리합니다.

## 프로젝트 구조

```
.
├── README.md          # GitHub 프로필 페이지에 렌더링되는 메인 파일
├── AGENTS.md          # AI 에이전트 가이드 (이 파일)
├── CLAUDE.md          # 루트 가이드 (.claude/CLAUDE.md 참조)
├── SETUP.md           # 환경 설정 안내
├── .claude/
│   ├── CLAUDE.md      # Claude Code 에이전트 가이드
│   └── settings.json  # 권한 설정
└── .rtk/
    └── filters.toml   # 필터 설정
```

## 언어

- 모든 커밋 메시지, 코드 주석, AI 응답은 **한국어**를 사용합니다.
- `README.md`의 본문 콘텐츠도 한국어로 작성합니다.
- 기술 용어와 코드 식별자(변수명, 함수명 등)는 원문 그대로 유지합니다.

## 커밋 컨벤션

Gitmoji + 한국어 설명 형식을 사용합니다:

```
<gitmoji> <type>: <한국어 설명>
```

예시:
- `📝 docs: Connect With Me 섹션 한국어로 변경`
- `🔧 fix: GitHub Stats 이미지 URL 대체 인스턴스로 변경`
- `🔥 chore: 깨진 GitHub Streak 이미지 제거`

### Gitmoji 매핑

| Gitmoji | Type | 용도 |
|---------|------|------|
| 📝 | docs | 문서 작성/수정 |
| 🔧 | fix | 설정 파일, 이미지 URL 등 기술적 수정 |
| 🔥 | chore | 코드/파일 제거 |
| ✨ | feat | 새 기능 추가 |
| 💄 | style | UI/스타일 변경 |
| ♻️ | refactor | 코드 리팩토링 |

## README.md 편집 규칙

1. GitHub 마크다운 문법을 사용하며, GitHub에서만 렌더링되는 특수 기능(GitHub Stats 카드, Shields.io 뱃지 등)을 활용할 수 있습니다.
2. 외부 이미지/뱃지는 HTTPS URL을 사용하고, 깨진 링크가 없는지 확인합니다.
3. 섹션 순서는 About Me → GitHub Statistics → Tech Stack → Current Focus → Featured Projects → GitHub Activity → Connect With Me 순서를 유지합니다.

## AI 에이전트 행동 규칙

1. README.md 외에 다른 파일을 생성할 때는 사용자에게 먼저 확인합니다.
2. gitmoji는 적절한 것을 선택하되, 애매한 경우 `📝 docs`를 기본으로 사용합니다.
3. 커밋은 사용자가 명시적으로 요청할 때만 생성합니다.
