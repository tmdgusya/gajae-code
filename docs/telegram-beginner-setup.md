---
title: GJC Telegram 초보자 설치 가이드
description: BotFather 토큰 발급부터 gjc notify setup, Telegram 그룹 생성, Topics 설정, 그룹 chat id 저장까지 순서대로 따라 하는 한국어 가이드.
---

# GJC Telegram 초보자 설치 가이드

이 가이드는 **처음 하는 사람이 화면을 보면서 그대로 따라 하는 절차**로 적었다.

긴 개발자용 스크립트는 없다. 필요한 명령은 `gjc` 설정 명령 몇 개뿐이다.

## 먼저 알아야 할 것

GJC Telegram 연동은 최종적으로 **개인 DM이 아니라 Topics가 켜진 Telegram 그룹**을 써야 한다.

흐름은 이렇게 간다.

```text
1. BotFather에게 봇 token 발급
2. gjc notify setup에 token 입력
3. Telegram 그룹 만들기
4. 그룹에서 Topics 켜기
5. 봇을 그룹에 넣고 관리자 권한 주기
6. 그룹 chat id(-100...) 찾기
7. gjc notify setup에 그룹 chat id 다시 저장
8. daemon 재시작
9. GJC 세션 토픽에서 대화
```

중요한 차이:

```text
개인 DM chat id 예시: 6352039836
그룹 chat id 예시: -1001234567890
```

최종 설정에는 `-100...`으로 시작하는 그룹 chat id가 들어가야 한다.

## 1. BotFather에게 봇 token 발급받기

Telegram에서 한다.

1. Telegram을 연다.
2. `@BotFather`를 검색한다.
3. BotFather 채팅을 연다.
4. 아래 명령을 보낸다.

```text
/newbot
```

5. BotFather가 봇 이름을 물어보면 원하는 이름을 적는다.

예시:

```text
gajae
```

6. BotFather가 봇 username을 물어보면 `bot`으로 끝나는 이름을 적는다.

예시:

```text
gajae-r-bot
```

7. BotFather가 token을 준다.

![BotFather가 새 봇 token을 발급한 화면](./assets/telegram-beginner/01-botfather-token.webp)

사진에서 검은색으로 가린 부분이 token이다.

이 token은 비밀번호다. 남에게 보여주면 안 된다.

형태는 보통 이렇게 생겼다.

```text
1234567890:AAExampleExampleExampleExampleExample
```

이 값을 복사해 둔다.

## 2. `gjc notify setup`에 봇 token 넣기

터미널을 연다.

먼저 GJC가 되는지 확인한다.

```sh
gjc --version
gjc --smoke-test
```

이제 Telegram 알림 설정을 시작한다.

```sh
gjc notify setup
```

그러면 이런 질문이 나온다.

```text
Telegram BotFather token:
```

여기에 1단계에서 복사한 BotFather token을 붙여넣고 Enter를 누른다.

그 다음 GJC가 봇에게 메시지를 보내라고 안내한다.

Telegram에서:

1. 방금 만든 봇 개인 채팅을 연다.
2. **Start** 버튼을 누르거나 아래처럼 보낸다.

```text
/start
```

터미널에 이런 식으로 나오면 1차 설정은 된 것이다.

```text
Token validated.
Notifications enabled. botToken=1234…(len 46) chatId=6352039836
```

여기까지는 token 확인과 기본 연결 단계다.

주의: 이때 저장된 `chatId`가 양수면 개인 DM이다. 최종 GJC 세션 알림은 그룹 토픽으로 보내야 하므로, 뒤에서 그룹 chat id로 다시 저장한다.

## 3. Telegram 그룹 만들기

Telegram에서 새 그룹을 만든다.

1. 왼쪽 위 새 메시지 / 연필 아이콘을 누른다.
2. **New Group**을 누른다.

![Telegram에서 New Group을 누르는 화면](./assets/telegram-beginner/02-new-group.png)

3. 그룹 이름을 적는다.

예시:

```text
gajae
```

4. 그룹을 만든다.

이 그룹이 GJC 알림을 받을 방이다.

## 4. 그룹 정보 화면에서 Edit 열기

방금 만든 그룹을 연다.

1. 그룹 이름을 누른다.
2. 그룹 정보 화면이 열린다.
3. 오른쪽 위 **Edit**을 누른다.

![Telegram 그룹 정보 화면의 Edit 버튼](./assets/telegram-beginner/03-group-edit.webp)

앞으로 그룹 설정을 바꿀 때는 이 화면에서 **Edit**을 누른다고 생각하면 된다.

## 5. 그룹에서 Topics 켜기

GJC는 세션마다 Telegram 토픽을 만들기 때문에 Topics가 필요하다.

그룹 정보 화면에서:

1. 오른쪽 위 **Edit**을 누른다.
2. **Topics** 또는 **토픽** 항목을 찾는다.
3. 켠다.
4. 저장한다.

Topics가 안 보이면 먼저 이것부터 확인한다.

- 내가 그룹 owner/admin인지 확인한다.
- Telegram 앱을 최신 버전으로 업데이트한다.
- Telegram Desktop에서 다시 시도한다.
- 그래도 안 되면 새 그룹을 만들고 바로 Topics를 켠다.

## 6. 봇을 그룹에 추가하기

그룹 정보 화면에서:

1. **Add** 또는 **Add Members**를 누른다.
2. 1단계에서 만든 봇 username을 검색한다.

예시:

```text
@gajae-r-bot
```

3. 봇을 그룹에 추가한다.

봇이 검색되지 않으면:

1. 봇 개인 채팅을 먼저 연다.
2. **Start**를 누른다.
3. 다시 그룹에서 봇을 검색한다.

그래도 안 되면 브라우저에서 아래 주소를 연다. username은 자기 봇 username으로 바꾼다.

```text
https://t.me/gajae-r-bot?startgroup=true
```

## 7. 봇을 관리자로 만들고 Topics 권한 주기

봇이 그룹에 들어온 것만으로는 부족하다. GJC가 토픽을 만들려면 봇에게 권한이 필요하다.

그룹 정보 화면에서:

1. 오른쪽 위 **Edit**을 누른다.
2. **Administrators / 관리자**를 연다.
3. 봇을 관리자로 추가한다.
4. 아래 권한을 켠다.

```text
Send Messages / 메시지 보내기
Manage Topics / Manage Forum Topics / 토픽 관리
```

5. 저장한다.

`Manage Topics`가 안 보이면 5단계의 Topics 설정이 아직 안 된 것이다.

## 8. 그룹에서 봇에게 `/start` 보내기

그룹 chat id를 찾으려면 봇이 그룹 메시지를 한 번 받아야 한다.

그룹에 아래처럼 보낸다. `gajae-r-bot` 부분은 자기 봇 username으로 바꾼다.

```text
/start@gajae-r-bot
```

그냥 `안녕` 같은 메시지는 봇이 못 볼 수 있다. 반드시 `/start@봇username` 형태로 보낸다.

## 9. 그룹 chat id 찾기

터미널 스크립트 말고 브라우저로 확인한다.

1. 아래 주소를 복사한다.

```text
https://api.telegram.org/bot<TOKEN>/getUpdates
```

2. `<TOKEN>` 부분을 BotFather token으로 바꾼다.

예시:

```text
https://api.telegram.org/bot1234567890:AAExampleExampleExample/getUpdates
```

3. 브라우저 주소창에 붙여넣고 연다.
4. `Command+F` 또는 `Ctrl+F`로 그룹 이름을 찾는다.
5. 그룹 이름 근처에서 `chat`의 `id`를 찾는다.

이런 모양이면 맞다.

```json
"chat":{"id":-1001234567890,"title":"gajae","type":"supergroup","is_forum":true}
```

여기서 복사할 값은 이것이다.

```text
-1001234567890
```

반드시 `-100`으로 시작하는 그룹 id를 써야 한다.

`getUpdates`에 그룹이 안 보이면:

1. 그룹에서 `/start@봇username`을 다시 보낸다.
2. 브라우저를 새로고침한다.
3. 봇이 그룹에 들어갔는지 확인한다.
4. 봇이 관리자이고 Topics 권한이 있는지 확인한다.

## 10. `gjc notify setup`에 그룹 chat id 다시 저장하기

이 단계가 최종 설정이다.

아래 명령에서 두 값을 바꾼다.

- `<BOTFATHER_TOKEN>`: 1단계에서 받은 token
- `-1001234567890`: 9단계에서 찾은 그룹 chat id

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
```

중요: `chat id`가 `-`로 시작하므로 `--chat-id=-100...`처럼 등호를 붙여 쓴다.

틀린 예:

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id -1001234567890
```

맞는 예:

```sh
gjc notify setup --token '<BOTFATHER_TOKEN>' --chat-id=-1001234567890
```

정상 출력 예시:

```text
Using provided chat id -1001234567890 (non-interactive).
Notifications enabled. botToken=1234…(len 46) chatId=-1001234567890
```

## 11. Telegram daemon 재시작하기

설정을 바꿨으니 daemon을 다시 읽힌다.

```sh
gjc daemon reload telegram --force
```

상태를 확인한다.

```sh
gjc notify status
gjc daemon status telegram
```

정상 상태는 이런 식이다.

```text
Notifications
  enabled: true
  botToken: 1234…(len 46)
  chatId: -1001234567890

telegram: running
```

`chatId`가 양수면 아직 개인 DM으로 되어 있는 것이다. 10단계를 다시 한다.

## 12. 실제로 GJC 실행하기

이제 GJC를 실행한다.

```sh
gjc
```

또는 tmux로 실행한다.

```sh
gjc --tmux
```

정상이라면 Telegram 그룹 안에 GJC 세션 토픽이 생긴다.

GJC에게 답장할 때는:

1. Telegram 그룹을 연다.
2. GJC가 만든 세션 토픽을 연다.
3. 그 토픽 안에 메시지를 쓴다.

봇 개인 DM이나 그룹의 일반 채팅에 쓰면 세션으로 안 들어갈 수 있다.

## 마지막 체크리스트

아래가 다 맞으면 끝이다.

```text
[ ] BotFather에서 token 발급받음
[ ] gjc notify setup에 token 넣음
[ ] Telegram 그룹 만들었음
[ ] 그룹 Edit 화면에서 Topics 켰음
[ ] 봇을 그룹에 추가했음
[ ] 봇을 관리자로 만들었음
[ ] 봇에게 Manage Topics / 토픽 관리 권한 줬음
[ ] 그룹에서 /start@봇username 보냈음
[ ] getUpdates에서 -100... 그룹 chat id 찾았음
[ ] gjc notify setup --token ... --chat-id=-100... 다시 실행했음
[ ] gjc notify status에서 chatId가 -100...으로 보임
[ ] gjc daemon status telegram이 running임
[ ] Telegram 그룹에 GJC 세션 토픽이 생김
```
