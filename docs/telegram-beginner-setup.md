---
title: GJC Telegram 초보자 설치 가이드
description: BotFather부터 Telegram 그룹 토픽, chat id, gjc notify setup까지 사람이 따라 하는 한국어 가이드.
---

# GJC Telegram 초보자 설치 가이드

이 문서는 Telegram으로 GJC 알림을 받고, Telegram에서 GJC에 답장하기 위한 초보자용 가이드다.

개발자용 긴 스크립트는 넣지 않는다. 사람이 Telegram 화면을 보면서 따라가는 순서로 설명한다.

## 핵심 요약

GJC Telegram 알림은 현재 **개인 DM이 아니라 그룹 토픽**에서 제대로 동작한다.

필요한 것은 이것이다.

1. BotFather로 만든 Telegram 봇
2. Topics가 켜진 Telegram 그룹
3. 그 그룹에 들어간 봇
4. 봇의 관리자 권한
5. 봇의 **Manage Topics / 토픽 관리** 권한
6. `-100...`으로 시작하는 그룹 chat id
7. 그 chat id를 저장한 `gjc notify setup`

헷갈리는 부분은 이것이다.

```text
gjc notify setup은 개인 DM으로도 성공할 수 있다.
하지만 실제 GJC 세션 알림은 개인 DM이 아니라 Topics 켜진 그룹으로 가야 한다.
```

개인 DM chat id는 보통 양수다.

```text
6352039836
```

실제로 필요한 그룹 chat id는 보통 `-100`으로 시작한다.

```text
-1001234567890
```

## 스크린샷은 필요한가?

필수 아니다.

스크린샷이 있으면 Telegram UI 위치를 설명하기 좋지만, 이 가이드는 스크린샷 없이 따라가게 작성했다.

문제가 생기면 스크린샷보다 아래 두 출력이 더 중요하다.

```sh
gjc notify status
gjc daemon status telegram
```

## 전체 흐름

처음 하는 사람은 이 순서대로 하면 된다.

```text
GJC 설치
→ BotFather에서 봇 생성
→ Telegram 그룹 생성
→ 그룹에서 Topics 켜기
→ 봇을 그룹에 초대
→ 봇을 관리자로 만들고 Manage Topics 켜기
→ 그룹 chat id 확인
→ gjc notify setup에 그룹 chat id 저장
→ daemon reload
→ GJC 실행
→ Telegram 그룹의 세션 토픽에서 대화
```

## 1. GJC 설치 확인

GJC가 이미 설치되어 있으면 이 단계는 확인만 한다.

```sh
gjc --version
gjc --smoke-test
```

둘 다 정상 출력되면 다음 단계로 간다.

GJC가 없다면 설치한다.

```sh
bun install -g gajae-code
```

Bun이 없다면 먼저 Bun을 설치해야 한다.

macOS / Linux:

```sh
curl -fsSL https://bun.sh/install | bash
```

Windows PowerShell:

```powershell
powershell -c "irm bun.sh/install.ps1|iex"
```

설치 후 터미널을 닫았다가 다시 연다.

## 2. BotFather에서 봇 만들기

Telegram에서 진행한다.

1. Telegram을 연다.
2. `@BotFather`를 검색한다.
3. BotFather 채팅을 연다.
4. 아래 명령을 보낸다.

```text
/newbot
```

5. BotFather가 봇 이름을 물어보면 원하는 이름을 입력한다.

예시:

```text
My GJC Bot
```

6. BotFather가 봇 username을 물어보면 `bot`으로 끝나는 이름을 입력한다.

예시:

```text
my_gjc_helper_bot
```

7. BotFather가 token을 준다.

token 모양은 대충 이렇게 생겼다.

```text
1234567890:AAExampleExampleExampleExampleExample
```

이 token은 비밀번호다. 공개하지 않는다.

## 3. 봇이 그룹에 들어갈 수 있게 설정

대부분 기본값으로 가능하지만 확인한다.

BotFather에서:

```text
/mybots
```

그 다음:

1. 방금 만든 봇 선택
2. **Bot Settings** 선택
3. **Allow Groups?** 선택
4. 그룹 허용 상태인지 확인

추가로 추천하는 설정:

```text
/setprivacy
```

봇을 선택하고 **Disable**을 고른다.

이 설정은 필수는 아니지만, 그룹에서 봇이 메시지를 못 보는 문제를 줄인다.

## 4. Telegram 그룹 만들기

Telegram에서 새 그룹을 만든다.

1. 새 그룹 생성
2. 본인 추가
3. 그룹 이름 지정

예시:

```text
GJC Remote Control
```

## 5. 그룹에서 Topics 켜기

이 단계가 중요하다.

Telegram에서:

1. 방금 만든 그룹을 연다.
2. 그룹 이름을 누른다.
3. Edit / 수정 / 연필 아이콘을 누른다.
4. **Topics** 또는 **토픽** 설정을 찾는다.
5. 켠다.
6. 저장한다.

Topics가 켜져야 GJC가 세션별 토픽을 만들 수 있다.

Topics 설정이 안 보이면:

- 내가 그룹 owner/admin인지 확인한다.
- Telegram 앱을 최신 버전으로 업데이트한다.
- Telegram Desktop에서 다시 시도한다.
- 새 그룹을 만들고 바로 Topics를 켜는 편이 더 빠를 때가 있다.

## 6. 봇을 그룹에 초대

그룹에서:

1. 그룹 정보 열기
2. Add Members / 멤버 추가 선택
3. 봇 username 검색

예시:

```text
@my_gjc_helper_bot
```

4. 봇 추가

봇 검색이 안 되면 먼저 봇 개인 DM을 열고 **Start**를 누른다.

그래도 안 되면 브라우저에서 아래 링크를 연다. username은 자기 봇 username으로 바꾼다.

```text
https://t.me/my_gjc_helper_bot?startgroup=true
```

## 7. 봇을 관리자로 만들고 토픽 관리 권한 켜기

봇이 그룹에 들어온 것만으로는 부족하다. 토픽을 만들 수 있어야 한다.

그룹에서:

1. 그룹 정보 열기
2. Administrators / 관리자 열기
3. 봇을 관리자로 추가
4. 아래 권한 켜기
   - Send Messages / 메시지 보내기
   - Manage Topics / Manage Forum Topics / 토픽 관리
5. 저장

`Manage Topics`가 안 보이면 Topics가 아직 제대로 켜지지 않은 것이다. 5단계를 다시 확인한다.

## 8. 그룹에서 봇에게 명령 보내기

그룹 chat id를 얻으려면 Telegram이 봇에게 그룹 업데이트를 보여줘야 한다.

그룹에 아래처럼 보낸다. username은 자기 봇 username으로 바꾼다.

```text
/start@my_gjc_helper_bot
```

그냥 `안녕` 같은 일반 메시지는 봇 privacy 설정 때문에 안 보일 수 있다. 반드시 `/start@봇username` 형태로 보낸다.

## 9. 그룹 chat id 찾기

chat id는 Telegram API의 `getUpdates` 화면에서 찾는다.

긴 터미널 스크립트 없이 브라우저로 확인한다.

1. 아래 주소를 복사한다.

```text
https://api.telegram.org/bot<TOKEN>/getUpdates
```

2. `<TOKEN>` 부분을 BotFather가 준 token으로 바꾼다.

예시:

```text
https://api.telegram.org/bot1234567890:AAExampleExampleExample/getUpdates
```

3. 이 주소를 브라우저 주소창에 붙여넣고 연다.

4. 화면에서 내가 만든 그룹 이름을 찾는다.

브라우저 찾기 기능을 쓰면 편하다.

```text
Ctrl+F 또는 Command+F
```

5. 그룹 이름 근처에 이런 부분이 있다.

```json
"chat":{"id":-1001234567890,"title":"GJC Remote Control","type":"supergroup","is_forum":true}
```

6. 여기서 `id` 값을 복사한다.

```text
-1001234567890
```

주의:

- `type`이 `supergroup`이어야 한다.
- 가능하면 `is_forum`이 `true`여야 한다.
- 양수 id는 개인 DM일 가능성이 높다.
- token이 브라우저 주소창에 들어가므로 공용 컴퓨터에서는 하지 않는다.
- 실수로 token을 공유했다면 BotFather에서 재발급한다.

`getUpdates`에 그룹이 안 보이면:

1. 봇이 그룹에 들어갔는지 확인한다.
2. 그룹에서 `/start@봇username`을 다시 보낸다.
3. 브라우저 새로고침을 한다.
4. 그래도 안 되면 BotFather에서 `/setprivacy`를 Disable로 바꾸고 다시 보낸다.

## 10. GJC에 그룹 chat id 저장

중요하다. chat id가 `-`로 시작하므로 `--chat-id=-100...`처럼 등호를 써야 한다.

틀린 예:

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id -1001234567890
```

맞는 예:

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
```

실행한다.

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
```

정상 출력 예시:

```text
Token validated. Message your bot now from the private Telegram chat to pair notifications.
Using provided chat id -1001234567890 (non-interactive).
Notifications enabled. botToken=1234…(len 46) chatId=-1001234567890
```

첫 줄에 private chat이 언급되어도 괜찮다. `--chat-id`를 넘겼으면 제공한 그룹 id가 저장된다.

## 11. Telegram daemon 재시작

```sh
gjc daemon reload telegram --force
```

정상 출력 예시:

```text
telegram reload: ok — spawned fresh telegram daemon (owner_spawned)
```

상태를 확인한다.

```sh
gjc daemon status telegram
gjc notify status
```

정상 상태 예시:

```text
telegram: running ...
Notifications
  enabled: true
  botToken: 1234…(len 46)
  chatId: -1001234567890
  redact: false
```

## 12. 실제로 되는지 확인

GJC를 실행한다.

```sh
gjc
```

또는 tmux 모드:

```sh
gjc --tmux
```

정상이라면 Telegram 그룹 안에 GJC 세션 토픽이 생긴다.

토픽 이름은 보통 repo 이름이나 세션 제목을 포함한다.

예시:

```text
my-project/main - 작업 제목
```

## 13. Telegram에서 GJC에 말 거는 법

아무 데나 쓰면 안 된다.

해야 하는 것:

1. 설정한 Telegram 그룹을 연다.
2. GJC가 만든 세션 토픽을 연다.
3. 그 토픽 안에서 메시지를 쓴다.
4. GJC가 질문을 보냈다면 inline button을 누르거나 그 토픽에 답장한다.

하지 말아야 하는 것:

- 봇 개인 DM에 메시지 보내기
- 그룹의 General 토픽에 메시지 보내기
- 설정한 그룹이 아닌 다른 그룹에 메시지 보내기

정상 접수되면 Telegram 메시지에 반응이 붙을 수 있다.

```text
eyes reaction = daemon이 메시지를 받음
check-mark reaction = GJC 세션이 메시지를 소비함
```

## 자주 생기는 문제

### setup은 성공했는데 Telegram에 아무것도 안 옴

확인한다.

```sh
gjc notify status
```

`chatId`가 양수면 개인 DM id일 가능성이 높다.

```text
chatId: 6352039836
```

그룹 id로 다시 설정한다.

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
gjc daemon reload telegram --force
```

### `Option '--chat-id' argument is ambiguous`

`-100...` 값을 넘길 때 등호를 안 써서 그렇다.

틀림:

```sh
gjc notify setup --token '<TOKEN>' --chat-id -1001234567890
```

맞음:

```sh
gjc notify setup --token '<TOKEN>' --chat-id=-1001234567890
```

### 그룹 chat id가 안 보임

그룹에서 다시 보낸다.

```text
/start@YourBotUsername
```

그 다음 브라우저의 `getUpdates` 페이지를 새로고침한다.

### Topics가 켜졌는지 모르겠음

그룹 설정에서 Topics가 켜져 있는지 다시 본다.

GJC가 토픽을 못 만들면 대부분 여기서 막힌다.

### 봇 권한이 맞는지 모르겠음

그룹 관리자 설정에서 봇 권한을 다시 본다.

필수:

```text
관리자
메시지 보내기
토픽 관리 / Manage Topics
```

### daemon이 stale이라고 나옴

```sh
gjc daemon status telegram
```

`stale`이면:

```sh
gjc daemon reload telegram --force
```

### Telegram 409 conflict가 나옴

같은 bot token으로 Telegram 업데이트를 읽는 프로그램이 둘 이상이라는 뜻이다.

해결:

1. 예전에 켜둔 수동 Telegram bridge나 봇 스크립트를 끈다.
2. 다시 실행한다.

```sh
gjc daemon reload telegram --force
```

### token을 실수로 공개함

BotFather에서 재발급한다.

1. BotFather에 `/mybots` 입력
2. 봇 선택
3. **API Token** 선택
4. Revoke/regenerate
5. GJC에 새 token 저장

```sh
gjc notify setup --token '<NEW_BOTFATHER_TOKEN>' --chat-id=-1001234567890
gjc daemon reload telegram --force
```

## 최종 체크리스트

완료 상태:

```text
[ ] gjc --smoke-test 성공
[ ] Telegram 봇 생성 완료
[ ] Telegram 그룹 생성 완료
[ ] 그룹 Topics 켜짐
[ ] 봇이 그룹에 들어옴
[ ] 봇이 그룹 관리자임
[ ] 봇에 Manage Topics / 토픽 관리 권한 있음
[ ] getUpdates에서 -100... 그룹 chat id 확인
[ ] gjc notify status가 chatId: -100... 표시
[ ] gjc daemon status telegram이 running 표시
[ ] Telegram 그룹에 GJC 세션 토픽 생성됨
[ ] 그 토픽에 쓴 메시지가 GJC 세션으로 들어감
```

이 체크리스트가 맞으면 끝이다.
