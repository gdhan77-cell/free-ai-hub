# 무료 AI 자동 전환 허브 (free-ai-hub)

무료 AI API 키를 여러 개 묶어 두고, 한도가 찬 키는 건너뛰어 다음 키로 **자동 전환(폴백)** 해 주는 단일 HTML 웹앱입니다.
PC 설치 없이 폰·PC 브라우저에서 바로 열어 씁니다.

- 실행 주소: **https://gdhan77-cell.github.io/free-ai-hub/**
- 파일: `index.html` 하나 (외부 CDN·폰트·아이콘 없음, 100% 인라인)

## 무엇을 하나요

1. **작업실** — 질문을 넣으면 무료 키를 우선순위대로 호출합니다. 한도 초과(429)·키 오류(401/403)·서버 오류(5xx)·시간 초과가 나면 **즉시 다음 키로 넘어가고**, 답변 위에 `제미나이 #1 · 한도 초과(429) → Groq #1 · 응답` 처럼 배지로 남깁니다. TXT·DOCX 첨부, 빠른 프리셋(블로그 초안·요약·시낭송 멘트·번역·편지)이 있습니다.
2. **키 풀** — 업체·별칭·모델·API 키를 넣고 위/아래로 우선순위를 바꿉니다. 키별 [연결 테스트], 활성 토글, 수정·삭제, JSON 백업/복원(기기 간 이동).
3. **절약·로그** — 무료로 처리한 질문 수, 클로드로 냈으면 들었을 값(토큰 기준 추정), 자동 전환 횟수와 최근 50건 전환 이력, 키별 상태.
4. **키 발급** — Google AI Studio · Groq Console · OpenRouter · Anthropic 바로가기와 3단계 안내.

## 지원 업체

| 업체 | 호출 주소 | 기본 모델 |
| --- | --- | --- |
| Google 제미나이 | `generativelanguage.googleapis.com/v1beta/models/{model}:generateContent` | `gemini-3.5-flash` (보조 `gemini-3-flash-preview`, `gemini-3.8-flash`, `gemma-4-26b-a4b-it`) |
| Groq | `api.groq.com/openai/v1/chat/completions` | `openai/gpt-oss-120b` (보조 `qwen/qwen3.8-27b`, `openai/gpt-oss-20b`) |
| OpenRouter | `openrouter.ai/api/v1/chat/completions` | `openrouter/free` (공식 무료 자동 라우터, 보조 `:free` 모델들) |
| Anthropic 클로드 (최후 보루) | `api.anthropic.com/v1/messages` | `claude-sonnet-5-5` |

- 클로드는 **"무료가 모두 막혔을 때만 클로드 사용"** 토글을 켰을 때만, 무료 키가 전부 실패한 뒤에 호출합니다.
- `claude-sonnet-5-5`는 `temperature` 파라미터를 보내면 400이 나므로 **temperature 없이** 호출하고, 404면 `claude-sonnet-4-6`으로 한 번 더 시도합니다.
- 브라우저 직접 호출용 헤더 `anthropic-dangerous-direct-browser-access: true`를 붙입니다.

## 사용 모드

- **무료만**: 무료 키만 사용. 모두 실패하면 멈춥니다.
- **무료 우선**(기본): 무료 키 → 모두 실패 시 클로드(토글 켜짐 + 키 있음).
- **클로드 최후 보루**: 무료 키 전부 실패 시 클로드가 반드시 답합니다.

## 저장 방식

- 모든 설정은 브라우저 `localStorage`의 **`han_free_ai_pool_v1`** 한 곳에 저장됩니다.
  ```json
  {
    "version": 1,
    "mode": "free_first",
    "claudeFallbackEnabled": false,
    "claudeKey": "",
    "claudeModel": "claude-sonnet-5-5",
    "providers": [
      { "id": "…", "type": "gemini", "label": "제미나이 무료 #1", "apiKey": "…",
        "model": "gemini-3.5-flash", "enabled": true, "priority": 1 }
    ],
    "stats": { "freeCalls": 0, "claudeSaved": 0, "failovers": 0, "logs": [] }
  }
  ```
- 같은 `gdhan77-cell.github.io` 오리진의 다른 웹앱(ai-debate-room · bonae-goodnote · han-ai-office)도 이 키 이름을 쓰도록 맞추면 키를 다시 넣지 않고 함께 쓸 수 있습니다.
- 키는 서버로 보내지 않고, 호출할 때 해당 업체 주소로만 전송됩니다. JSON 내보내기로 폰↔PC 이동이 됩니다.

## 옛 모델명 자동 변경

- 2026-11-25 이전에 저장해 둔 키의 모델명(`gemini-2.5-flash`, `gemini-2.0-flash`, `gemini-2.5-flash-lite`, `llama-3.3-70b-versatile`, `llama-3.1-8b-instant`, `qwen/qwen3-32b`, `meta-llama/llama-3.3-70b-instruct:free`, `google/gemma-3-27b-it:free`, `deepseek/deepseek-r1-0528:free`, `qwen/qwen3-235b-a22b:free`)는 모두 제공사에서 종료되어 404가 납니다.
- 이 허브와 세 앱은 저장된 키 풀을 읽을 때 그런 옛 모델명을 새 기본 모델(`gemini-3.5-flash` · `openai/gpt-oss-120b` · `openrouter/free`)로 자동 치환하고 곧바로 다시 저장합니다. 키를 지웠다 넣을 필요 없이 새로고침만 하면 됩니다.

## 검증 (2026-11-25)

Playwright 헤드리스 브라우저로 데스크톱(1440×900)·모바일(390×844)에서 40개 항목 통과:

- 콘솔 에러·페이지 예외 0건, 가로 스크롤 0(모든 탭·모바일)
- 키 없이 데모 시연(429 → 다음 키 자동 전환 배지·로그)
- 모의 API: 제미나이 429 → Groq 성공, 401+500 → 클로드 최후 전환
- 클로드 요청 본문 `model: claude-sonnet-5-5`, `temperature` 없음, 404 시 `claude-sonnet-4-6` 재시도
- TXT·DOCX 첨부 본문이 실제 요청에 포함
- 키 추가·우선순위 이동·마스킹·JSON 내보내기/가져오기
