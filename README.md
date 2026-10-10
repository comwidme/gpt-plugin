# AI4You Plugins

프롬프트 설계 등 AI 활용 워크플로우를 공유하는 플러그인 marketplace입니다.

- GitHub 저장소: https://github.com/comwidme/gpt-plugin
- Marketplace 식별자: `ai4you-plugins`
- Marketplace 표시 이름: **AI4You Plugins**

## 포함된 플러그인

| 플러그인 | 식별자 | 버전 | 스킬 |
|---|---|---|---|
| Prompt Design Tools | `prompt-design-tools` | 1.0.0 | `prompt-design` |

Prompt Design Tools는 프롬프트 신규 설계, 진단, 고도화, 압축을 지원합니다.
MCP 서버나 외부 서비스 연결을 포함하지 않습니다.

## 설치

Codex CLI가 설치된 환경에서 marketplace를 등록합니다.

```bash
codex plugin marketplace add comwidme/gpt-plugin
codex plugin marketplace list
```

지원되는 클라이언트의 플러그인 목록에서 **AI4You Plugins** marketplace를 선택하고
**Prompt Design Tools**를 설치합니다. Codex CLI에서는 `codex`를 실행한 뒤
`/plugins`로 플러그인 목록을 열 수 있습니다. 설치 후 새 대화를 시작하세요.
데스크톱에서 목록이 갱신되지 않으면 앱을 재시작하세요.

현재 사용 중인 CLI가 명령을 제공하지 않으면 `codex plugin --help`를 확인하고,
해당 명령을 지원하는 버전을 사용하세요.

## 사용 예시

지원되는 입력창에서 `@`를 입력하고 **Prompt Design Tools**를 선택한 뒤 요청합니다.

> prompt-design 스킬을 사용해 아래 초안을 고도화해 주세요.
> 초안: 중소기업 대표를 위한 AI 업무활용 교육안을 작성해줘.
> 대상, 교육목표, 2시간 구성, 실습 결과물을 포함해줘.
> 교육안 자체를 작성하거나 실행하지 말고 개선된 프롬프트만 제시해줘.

Codex에서 스킬 선택이 지원되는 경우 `$prompt-design`을 사용할 수 있습니다.
동명의 개인용 스킬도 설치되어 있다면 플러그인 선택 표시나 스킬 로딩 경로로
어느 스킬이 사용됐는지 확인하세요.

## 업데이트

```bash
codex plugin marketplace upgrade ai4you-plugins
```

갱신 후 클라이언트에서 설치 상태를 확인하고 새 대화에서 사용하세요.

## 파일 구성

| 경로 | 역할 |
|---|---|
| `.agents/plugins/marketplace.json` | Marketplace 목록 |
| `plugins/prompt-design-tools/plugin.json` | Portable 플러그인 manifest |
| `plugins/prompt-design-tools/.codex-plugin/plugin.json` | Codex 호환 manifest |
| `plugins/prompt-design-tools/skills/prompt-design/` | 실행용 스킬·참고자료·자산 |
| `plugins/prompt-design-tools/assets/original-prompt-design/` | 원본 개인용 스킬 보존 사본 |

Marketplace의 `source.path`는 저장소 루트 기준입니다.
단일 저장소 안의 플러그인을 사용하므로 `source: local`로 선언되어 있으며,
GitHub marketplace 등록 시 받아온 저장소 내부에서 해당 경로를 해석합니다.
`.agents`와 `.codex-plugin` 같은 숨김 디렉터리도 포함해야 합니다.

## 원본 보존 및 호환성

원본 SKILL.md, agents/openai.yaml, 참고자료 2개, icon.svg 총 5개를
`assets/original-prompt-design/`에 원래 구조와 내용 그대로 보존했습니다.

실행용 `agents/openai.yaml`은 등록 시스템이 허용하지 않는
`policy.products` 목록만 제거했습니다. 표시 정보, 기본 프롬프트, 아이콘 경로와
`allow_implicit_invocation: true`는 유지했습니다.
본문·참고자료·자산은 원본과 동일합니다.
이 저장소를 배포해도 기존 개인용 스킬은 삭제되거나 수정되지 않습니다.

## 검증 범위

- 개인 계정의 등록 릴리스 1.0.0 파일을 기준으로 구성
- 원본 보존 파일 및 실행용 본문·참고자료·자산 일치 검사
- Marketplace 경로, manifest 경로, 참고자료·아이콘 상대경로 검사
- MCP 서버와 외부 앱 연결이 없는 스킬 전용 구성

**GitHub 배포는 새 Work 또는 웹 Chat의 호출 성공을 입증하지 않습니다.**
새 Work·새 웹 Chat·다른 사용자의 설치와 호출은 실제 확인 전까지 미검증입니다.
이전에 시도한 테스트 브라우저의 Google 로그인은 502 Bad Gateway 연결 오류로 중단됐습니다.
GitHub marketplace 배포와 ChatGPT 공개 플러그인 디렉터리 게시는 별개의 절차입니다.

## 라이선스

기존 저장소의 [MIT License](LICENSE)를 유지합니다.

## 공식 문서

- [플러그인 패키징과 Git marketplace](https://developers.openai.com/plugins/build/plugins)
- [공개 디렉터리 제출](https://developers.openai.com/plugins/deploy/submission)
