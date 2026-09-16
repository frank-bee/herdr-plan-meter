<div align="center">

# Plan Meter

### Claude Code·Codex 요금제 한도를 herdr 안에서 한눈에.

가장 빠듯한 한도는 탭 바에 늘 떠 있고, 키 하나로 모든 한도를 펼쳐 봅니다.

[![CI](https://github.com/JunSeo99/herdr-plan-meter/actions/workflows/ci.yml/badge.svg)](https://github.com/JunSeo99/herdr-plan-meter/actions/workflows/ci.yml) [![MIT](https://img.shields.io/badge/license-MIT-2f6650)](LICENSE) ![herdr 0.9+](https://img.shields.io/badge/herdr-0.9%2B-6E56CF) ![Python 3.9+, stdlib only](https://img.shields.io/badge/python-3.9%2B%20stdlib-3776AB) ![macOS · Linux](https://img.shields.io/badge/platforms-macOS%20·%20Linux-lightgrey)

[English](README.md) · **한국어**

<img src="assets/popup.png" width="509" alt="Plan Meter 팝업: Claude Max 20x 5시간 48%·주간 30%·주간 Fable 32%, Codex Plus 13%·6%, 한도마다 막대와 초기화 시각">
<br>
<img src="assets/tab-bar.png" width="352" alt="탭 바: Claude 48% 사용·2h 25m 후 초기화, Codex 13% 사용·2h 57m 후 초기화">

<sub>실제 계정(Claude Max 20x · ChatGPT Plus)의 사용량, 브랜드 아이콘 켬. UI는 한국어로도 나옵니다.</sub>

</div>

```bash
herdr plugin install JunSeo99/herdr-plan-meter
herdr plugin action invoke junseo99.plan-meter.setup
```

두 번째 명령을 실행하면 이 설치에 맞춘 설정 스니펫이 팝업으로 뜹니다. `c`로 복사해 `~/.config/herdr/config.toml`에 붙여 넣고 `herdr server reload-config`를 실행하세요.

## 무엇이 보이나

| 위치 | 내용 |
|---|---|
| 탭 바 | 요금제마다 한도에 가장 가까운 창의 사용률과 초기화까지 남은 시간. 15분 넘게 갱신되지 않은 값에는 `~`가 붙습니다. |
| 팝업 (`prefix+u`) | 모든 창(5시간·주간·모델별 주간)을 여유에 따라 색이 바뀌는 막대와 초기화 시각(내 시계 기준)으로 보여 줍니다. `Tab` 상세/압축 전환, `r` 새로고침, `q` 닫기. |
| 오류 | 로그인 만료, 429, 응답 형식 변경은 팝업에 표시되고, 마지막 정상값은 그대로 남습니다. |

로그인한 요금제만 나옵니다.

**왜 또 하나의 사용량 플러그인인가?** 의존성도 빌드도 없는 파이썬 파일 하나입니다. Claude Code의 statusLine을 건드리지 않고, Claude 세션이 떠 있지 않아도 숫자가 나오며, 인증 정보는 읽기만 합니다.

## 설정

1. `herdr plugin install JunSeo99/herdr-plan-meter`
2. `herdr plugin action invoke junseo99.plan-meter.setup` 실행 후 `c`
3. `~/.config/herdr/config.toml`에 붙여 넣기. 이미 `[ui]` 테이블이 있다면 그 안으로 키를 옮기세요. TOML은 `[ui]`가 두 번 나오는 걸 허용하지 않습니다.
4. `herdr server reload-config`

스니펫은 세 가지를 합니다.

- 플러그인 설정 디렉터리의 `tab-bar.sh`를 실행하는 `ui.tab_bar_right` 항목을 추가합니다.
- 탭 바를 하단으로 옮깁니다. 선택 사항이라, 위에 두려면 그 줄을 지우면 됩니다.
- `prefix+u`에 팝업을 연결합니다. 이 키를 쓰는 사용량 플러그인이 여럿이니 겹치면 바꾸세요.

`tab-bar.sh`는 현재 설치본을 가리키는 한 줄짜리 shim입니다. herdr의 플러그인 설치 경로는 내부 구현이라, Plan Meter가 herdr 시작 때마다 shim을 다시 씁니다. 그래서 config를 고칠 일이 없습니다.

### 갱신 방식

사용량은 에이전트가 일할 때만 움직입니다. 그래서 에이전트 턴이 끝날 때(herdr `pane.agent_status_changed` 이벤트) 갱신하고, 최소 간격은 2분입니다. 여기에 더해 5분마다 폴링해서 초기화와 다른 머신에서 쓴 사용량도 잡습니다.

탭 바는 15초마다 캐시에서 다시 그리므로 갱신 결과가 몇 초 안에 보입니다. 직접 갱신하려면 팝업에서 `r`을 누르거나 새로고침 액션에 키를 연결하세요.

```toml
[[keys.command]]
key = "prefix+shift+u"
type = "plugin_action"
command = "junseo99.plan-meter.refresh"
description = "요금제 사용량 새로고침"
```

수동 갱신은 30초에 한 번까지입니다.

### 옵션

`$(herdr plugin config-dir junseo99.plan-meter)/config.toml`:

```toml
lang = "auto"   # auto | en | ko. auto는 macOS UI 언어, 그다음 LANG을 따릅니다.
icons = "text"  # text | nerd
```

### 브랜드 아이콘

`icons = "nerd"`로 두면 이름 대신 Nerd Fonts 3.5의 Claude·OpenAI 로고(`cod-claude` U+EC82, `cod-openai` U+EC81)가 나옵니다. 터미널 폰트에 이 글리프가 있어야 합니다. Ghostty 1.3에 내장된 Nerd Font는 버전이 낮으니, 심볼 폰트를 설치하고 두 코드포인트를 매핑하세요.

```bash
brew install --cask font-symbols-only-nerd-font
```

```ini
# Ghostty 설정에 추가한 뒤 다시 불러오기 (macOS ⌘⇧,)
font-codepoint-map = U+EC81-U+EC82=Symbols Nerd Font Mono
```

## 동작 방식

| | Claude Code | Codex |
|---|---|---|
| 읽는 로그인 | Claude Code가 쓰는 macOS 키체인 항목(`Claude Code-credentials-<hash>`), Linux에서는 `~/.claude/.credentials.json`. `CLAUDE_CONFIG_DIR`를 따릅니다. | ChatGPT 로그인으로 생긴 `$CODEX_HOME/auth.json` (기본 `~/.codex`) |
| 엔드포인트 | `api.anthropic.com/api/oauth/usage` | `chatgpt.com/backend-api/wham/usage` |
| 창 | 5시간, 주간, 모델별 주간 | 5시간, 주간 |

- **토큰은 제자리에.** 조회할 때마다 토큰을 읽어 해당 회사에만 보내고, 보관하지 않습니다. 리다이렉트는 거부하므로 토큰이 3xx를 따라 다른 곳으로 가지 않습니다. 토큰을 쓰거나 갱신하는 일은 없습니다. 갱신은 CLI의 몫이고, 밖에서 refresh token을 돌리면 CLI 로그인이 풀릴 수 있습니다. 토큰이 만료되면 "로그인 만료 — Claude Code를 실행하면 갱신돼요"라고 표시됩니다.
- **캐시에는 숫자만.** `~/.cache/herdr-plan-meter/usage.json`(`$XDG_CACHE_HOME`이 있으면 그 아래)에는 요금제 이름, 사용률, 초기화 시각만 있습니다. 읽고 쓰는 파일 전체 목록은 [SECURITY.md](SECURITY.md)에 있습니다.
- **예의 바른 조회.** 파일 잠금으로 탭 바, 팝업, 이벤트 훅이 중복 호출하지 않습니다. 429를 받으면 `Retry-After`만큼, 최소 5분을 기다립니다.
- **statusLine의 `rate_limits`를 쓰지 않는 이유.** Claude Code가 공식 문서로 제공하긴 하지만, 살아 있는 세션이 필요하고 플러그인이 사용자의 statusLine을 차지해야 합니다. 모델별 창도 없습니다.

## 주의할 점

- **비공개 엔드포인트.** 두 회사 모두 이 사용량 엔드포인트를 문서화하지 않았고, 예고 없이 바뀔 수 있습니다. 응답을 해석할 수 없게 되면 팝업에 그 사실이 표시되고 마지막 값은 남습니다. 이슈를 남겨 주세요.
- **회색지대.** Claude Code의 OAuth 토큰을 Claude Code 밖에서 Anthropic 사용량 엔드포인트로 보냅니다. 할당량을 읽기만 하고 모델 요청은 하지 않지만, 쓰기 전에 적용되는 약관을 확인하세요.
- **키체인 확인 창.** macOS에서는 처음 읽을 때 Claude Code 항목 접근을 허용할지 물을 수 있습니다.
- Plan Meter는 Anthropic, OpenAI, herdr와 제휴 관계가 없습니다.

## 문제 해결

| 증상 | 확인할 것 |
|---|---|
| 탭 바에 아무것도 없음 | `$(herdr plugin config-dir junseo99.plan-meter)/tab-bar.sh`를 직접 실행해 보고, 스니펫이 하나뿐인 `[ui]` 테이블 안에 있는지 확인 |
| Claude가 "로그인 만료" | Claude Code를 한 번 실행하세요. 스스로 토큰을 갱신하고, 다음 조회에 반영됩니다. |
| 요금제 하나가 안 보임 | 구독 로그인이 없는 요금제는 숨깁니다. API 키 로그인에는 요금제 한도가 없습니다. |
| 로고 대신 네모나 한자가 보임 | 폰트에 Nerd Fonts 3.5 글리프가 없습니다. [브랜드 아이콘](#브랜드-아이콘)을 보거나 `icons = "text"`로 두세요. |
| 로그 | `herdr plugin log list --plugin junseo99.plan-meter` |

## 업데이트와 삭제

```bash
herdr plugin install JunSeo99/herdr-plan-meter   # 다시 설치하면 업데이트
herdr plugin uninstall junseo99.plan-meter
```

삭제한 뒤에는 `config.toml`에서 스니펫을 지우고 `~/.cache/herdr-plan-meter`를 삭제하세요.

## 요구 사항

- herdr 0.9 이상 (팝업 pane, `tab_bar_right` command 항목)
- Python 3.9 이상, 표준 라이브러리만 사용
- macOS 또는 Linux. Linux는 CI에서 샘플 데이터 테스트만 돌리고, 실제 계정으로는 아직 확인하지 않았습니다.
- 구독으로 로그인한 Claude Code 그리고/또는 Codex CLI

## 계정 없이 써 보기

```bash
PLAN_METER_DEMO=1 python3 meter.py panel
```

데모 모드는 인증 정보를 읽거나 네트워크에 접속하지 않고 샘플 데이터를 그립니다.

## 개발

```bash
python3 -m unittest discover -s tests -v
herdr plugin link "$PWD"   # 작업 트리를 설치본으로 사용
```

## 라이선스

[MIT](LICENSE)
