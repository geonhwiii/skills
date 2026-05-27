# Dan Skills

> 실제 작업에 쓰기 위한 [Claude Code](https://docs.claude.com/en/docs/claude-code/overview)
> 및 Cursor Agent 스킬 모음입니다. 작고 조합 가능하며, 중요한 판단은
> 개발자에게 남기도록 설계합니다.

이 저장소는 단일 스킬 플러그인입니다. 스킬은 `skills/` 아래에 두고,
실제로 사용하는 버킷만 만들어 관리합니다.

## 설치

이 저장소를 하나의 플러그인으로 설치합니다.

```shell
/plugin install geonhwiii/skills
/reload-plugins
```

## 왜 만들었나

Agent 스킬은 거대한 프로세스를 대신할 때보다, 반복 가능한 작은 워크플로를
담을 때 가장 유용합니다. 이 저장소는 그런 워크플로를 쉽게 고치고 믿고 쓸 수
있을 만큼 작게 유지합니다.

목표는 에이전트가 일을 소유하게 만드는 것이 아닙니다. 에이전트에게 더 나은
기본값을 주되, 중요한 판단은 개발자에게 남기는 것입니다.

## 스킬

### Engineering

코드 작업에 사용하는 스킬입니다.

- **[growth-guardian](./skills/engineering/growth-guardian/SKILL.md)** —
  코드를 생성하기 전에 짧은 설계 체크포인트를 둡니다. 숨어 있는 설계 결정을
  드러내고, 먼저 자신의 초안을 생각하게 한 뒤, 트레이드오프가 있는 선택지를
  제시해 판단을 외주화하지 않도록 돕습니다.

## 저장소 구조

```text
my-skills/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── engineering/
│       ├── README.md
│       └── growth-guardian/
│           ├── SKILL.md
│           └── examples.md
├── CLAUDE.md
├── README.md
└── LICENSE
```

## 새 스킬 추가

1. 스킬의 사용 맥락에 맞는 버킷을 고르거나 새로 만듭니다.
2. `skills/<bucket>/<skill-name>/SKILL.md`를 만듭니다.
3. `SKILL.md`는 간결하게 유지합니다. 예시나 긴 참고 자료는 같은 폴더의 별도
   파일로 둡니다.
4. 배포할 준비가 된 스킬이라면 다음 세 곳에 등록합니다.
   - 이 README
   - `skills/<bucket>/README.md`
   - `.claude-plugin/plugin.json`
5. 개인용 초안과 deprecated 스킬은 공개 manifest에 넣지 않습니다.
6. 커밋하기 전에 검증합니다.

```shell
claude plugin validate .
```

## 라이선스

MIT
