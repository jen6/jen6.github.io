---
layout: post
title: "⚒︎Hammer spoon과 🍎Apple script를 사용해 <br>AWS 2FA OTP 인증 자동화 하기"
description: ""
category: 
tags: ["자동화", "hammer spoon", "apple script", "aws", "2fa", "otp", "totp"]
thumbnail: https://drive.google.com/uc?export=view&id=1eE2RxRvJwa8radQT7B3VbNnmGfcc8suB
---

![https://drive.google.com/uc?export=view&id=1eE2RxRvJwa8radQT7B3VbNnmGfcc8suB](https://drive.google.com/uc?export=view&id=1eE2RxRvJwa8radQT7B3VbNnmGfcc8suB)

회사 vpn이나 aws console에 접근 하려면 TOTP(Time-based One-time password)를 이용한 2FA를 반드시 사용해야한다. 출근하자마자 매일 쓰게되는 이 두 가지를 로그인 하는 일은 어~~ㅁ청 귀찮다... 

대부분의 서비스는 자동 로그인이 가능하지만 OTP 코드만은 2FA authenticator인 [authy](https://authy.com/) 를 켜고 복사하고 붙여 넣어야 한다. 문제는 이걸 켜놓게 되면 Command-Tab으로 작업 전환시 크롬대신 authy창이 포커스가 된다. Vimnium을 써서 가능한 키보드만 이용해서 작업 하려는 내 작업 방식에 큰 방해가 되는 요소 였다.

이 귀찮은 일을 자동화 하게 된 계기는 최근 즐겨보는 블로그 중 한 개인 John Grib님의 블로그에 맥에서 이런저런 자동화를 시킬 수 있는 [hammerspoon툴 시리즈](https://johngrib.github.io/wiki/hammerspoon/) 글을 보고 내 매일 아침을 30초 정도 단축할 수 있겠다 싶어서 시작했다.

## 이 글을 10분 동안 따라하면 할 수 있는 것

![aws console](https://drive.google.com/uc?export=view&id=1Z5I2xkIoJeex3klP9eTIu9pnYZnhomyc)

위 두 영상처럼 aws console, tunnelblick등 OTP를 입력해야하는 곳을 자동화 할 수 있다.

## 준비물

맥, 그리고 Hammer spoon

## Hammer spoon과 AppleScript

먼저 [Hammer spoon](https://www.hammerspoon.org/)은 운영체제 단에서 일어나는 wifi, process, file-system등의 event들을 받고 lua를 통해 매크로를 실행시킬 수 있는 툴이다. 설명을 빌려오자면

- `Esc` 키를 누를 때마다 input source를 영문으로 전환하게 한다. Vim 사용자에게 너무 좋은 기능.
- 현재 실행 중인 애플리케이션의 윈도우를 특정 위치로 움직이게 하거나 사이즈를 조절한다.
- 특정 애플리케이션(파인더라던가)이 실행될 때마다 무언가 다른 작업을 수행하게 한다.
- 맥북이 회사 와이파이에 연결되면(출근하면) 맥북의 사운드 볼륨을 0으로 조정한다.
- 애플스크립트를 실행한다.
- iMessage를 전송한다.
- 터미널 명령어를 실행한다.

> 출저 : [https://johngrib.github.io/wiki/hammerspoon-tutorial-00/](https://johngrib.github.io/wiki/hammerspoon-tutorial-00/)

이외에도 특정 기능을 url에 바인딩 시켜 해당 url이 호출 되면 특정 매크로를 실행 시키는 등 편리한 기능들이 많다.

**AppleScript**는 Mac 환경에서 UI 요소들을 통해 자동화를 할 수 있는 도구이다. Apple에서 직접 제공하는거라 상당히 강력한 기능들이 많이 있고 hammer spoon에서는 화면에서 text를 가져온다거나 하는 일을 할 수 없는데 AppleScript에서는 가능하다.

- iTunes에서 가사를 추가 한다
- 이미지를 선택했을 때 포맷을 바꿔주거나 리사이징을 해준다.
- iMessage로 주소록에 있는 모든 유저에게 메세지 보낸다.
- pdf를 열었을 때 구글 번역을 통해 번역된 상태로 보게 한다.
- 크롬에서 특정 페이지 열렸을 때 매크로를 실행시킨다.

> 출저 : [https://www.macscripter.net/](https://www.macscripter.net/)

## Step 1. Authy에서 TOTP URI 뽑아내기

Authy같은 authenticators에 새로운 인증을 등록 하려면 대부분 QR코드 혹은 secret key를 입력해줘야한다. 자동화를 하려면 콘솔 authenticator에 이 키를 등록해줘야하는데 대부분 안가지고 있는 경우가 대부분 이기 때문에 이 secret key를 포함하고 있는 TOTP URI를 Authy에서 뽑아줘야한다.

![https://drive.google.com/uc?export=view&id=1cvYDcdW0maqUOFQZyagf3HbXI-DF2yJO](https://drive.google.com/uc?export=view&id=1cvYDcdW0maqUOFQZyagf3HbXI-DF2yJO)

> 이미지 출저 : [https://www.allthingsauth.com/2018/04/20/a-medium-dive-on-the-totp-spec/](https://www.allthingsauth.com/2018/04/20/a-medium-dive-on-the-totp-spec/)

위 그림에서 보이는 것처럼 URI format이 있고 query parameter로 들어가는 secret부분이 필요하다. 이 글에서는 내가 쓰는 authy기준으로 URI를 뽑는 방법을 설명한다.

![https://drive.google.com/uc?export=view&id=1Ovk7cqxwgIFUkez4xucV9k2-2y4cUj7e](https://drive.google.com/uc?export=view&id=1Ovk7cqxwgIFUkez4xucV9k2-2y4cUj7e)

먼저 Authy chrome app이 설치 돼 있다면 `chrome://extensions/?id=gaedmjdfmmahhbjefcbgaolhhanlaolb` 로 들어가서 Authy의 extension 정보 페이지로 들어가준다. 그 다음 Authy를 실행시켜 주면 '뷰 검사' 항목에 `main.html` 이라는 항목이 생긴다. 이걸 눌러줘서 Authy의 개발자 콘솔로 들어가자.

{% gist a13ef83a5af57e45c4820c3da5ba0e31 authy_export-js %}

위 스크립트를 개발자 콘솔에서 입력을 하게 되면 아래와 같이 authy에 등록 돼 있는 TOTP URI를 얻을 수 있다. 이 URI에서 아까 얘기했던 `secret` query parameter 값을 잘 저장해둔다.

![https://drive.google.com/uc?export=view&id=1qIAi3slxxAZpyLgInjRTTp602sOYkVue](https://drive.google.com/uc?export=view&id=1qIAi3slxxAZpyLgInjRTTp602sOYkVue)

## Step 2. Console authenticator에 키 등록하기

먼저 console에서 OTP 기능을 쓸 수 있게 해주는 oath-toolkit을 설치해준다

```bash
$ brew install oath-toolkit
```

oath-toolkit은 OTP 기능을 사용할 수 있게 해주지만 키를 관리해주거나 하는 등의 역할은 해주지 않는다. 간단한 shellscript로 OTP 키를 등록, 삭제를 할 수 있도록 `.bash_profile` 혹은 `.zshrc` 에 아래 스크립트를 추가 해 준다.

{% gist a13ef83a5af57e45c4820c3da5ba0e31 otp_console.sh %}

위 스크립트를 제대로 추가했는지 확인 해주기 위해 아까 authy에서 뽑아놓은 TOTP secret key를 등록하고 제대로 나오는지 확인해 보자

```bash
# OTP Secret 키를 등록한다 mfa_add {서비스 이름} {TOTP secret key}
$ mfa_add aws XVWXDMXZZXJIXA3XIOHX46VXXKQEXXXI73D6SX7VXXXNVFXXIOYF5F74BYHTI7FK
Secret added to keychain

# OTP 토큰을 가져온다. mfa {등록한 서비스 이름}. 토큰은 클립보드에 저장 돼 있어서 붙여넣기를 하면된다.
$ mfa aws
19:49:58 822269

# 등록한 서비스를 삭제한다. mfa_delete {서비스 이름}
$ mfa_delete aws
password has been deleted.
```

## Step 3. VPN Client OTP 입력 자동화 하기

Tunnelblick은 회사에서 쓰는 VPN client인데 역시 OTP를 사용해야한다. VPN Client OTP 입력은 Hammer spoon의 `Application Watcher` 기능을 이용해서 만들거다.

로직은 간단하게 Tunnelblick이라는 application이 activated 즉 창이 선택 됐을 경우 특정 루틴을 실행시켜주는 역할을 한다. 아까 만들어둔 console authenticator을 실행시켜줘서 OTP 토큰을 복붙한다음 엔터를 쳐주는 매크로이다.

```lua
require("hs.ipc")

if (not hs.ipc.cliStatus()) then
  hs.ipc.cliInstall()
  hs.alert.show("cli Installed")
end

appwatcher = hs.application.watcher.new(function(name,event, app) appwatch(name,event,app) end):start()
function appwatch(appName, eventType, appObject)
    if (eventType == hs.application.watcher.activated) then
        if (appName == "Tunnelblick") then
            hs.execute("mfa openvpn", true)
            hs.application.launchOrFocus('Tunnelblick')
            hs.eventtap.keyStrokes(hs.pasteboard.getContents())
            hs.eventtap.keyStroke({}, "return")
        end
    end
end
```

위 스크립트를 Hammer Spoon Config에 추가한 다음 reload를 해주자. 이제 VPN Client에다가 OTP를 입력해야하는 반복 노동에서 벗어날 수 있다 🎉.

![vpn](https://drive.google.com/uc?export=view&id=10GRUg-yVfQAN6Ji1Z_H3wpAdCxxwPE8a)

> Known Issue :  Tunnelblick의 다른 창이 뜰 경우에도 위 루틴이 동작하기 때문에 종종 이상한 창이 뜨는 경우가 있다. 그래도 매번 authy를 안 켤 수 있는게 귀찮음이 더 적다

## Step 4. AWS Console Login OTP 입력 자동화 하기

위에 Step 3에서 한 방식대로 AWS console 도 해결할 수 있으면 좋겠지만 바로 입력화면이 뜨는 VPN client 와 달리 크롬에서 특정 페이지가 열렸는지를 알아내는건 Hammer spoon로는 해결 할 수 없다. 대신 Apple Script를 이용해서 크롬에서 aws login 페이지가 열렸는지를 확인하고 Hammer spoon의 `url event` 기능을 이용해서 지정해놓은 매크로를 호출하는 방식으로 했다.

나는 기본적으로 chrome의 자동 로그인 기능을 사용해서 id, password까지는 자동 완성이 된다. 엔터 눌러주고 OTP를 자동입력하는 스크립트를 작성해보자.

```lua
require("hs.ipc") 

if (not hs.ipc.cliStatus()) then
  hs.ipc.cliInstall()
  hs.alert.show("cli Installed")
end
-- 위에서 추가 했으면 꼭 추가해주지 않아도 됨

-- urlevent 등록: awsTotp
hs.urlevent.bind("awsTotp", function(eventName, params)
  hs.execute("mfa aws", true)
  hs.application.launchOrFocus('Google Chrome')
  hs.eventtap.keyStrokes(hs.pasteboard.getContents())
  hs.eventtap.keyStroke({}, "return")
  hs.alert.show("🎉login🎉")
end)
```

위에 코드를 Hammer Spoon config에 추가 하게 되면 `hammerspoon://awsTotp` url 로 GET 요청을 날리게 되면 지정한 기능을 실행시켜준다. 이번에는 로그인하면 축하해주기 위해서 성공 문구도 추가해뒀다. 

{% gist a13ef83a5af57e45c4820c3da5ba0e31 chrome_tracking.applescript %}

Apple Script에서는 hammerspoon에서 지정해놓은 매크로를 호출하는 조건을 다음과 같이 설정한다.

1. Chrome이 제일 최상단이고
2.  현재 탭의 URL이 AWS 일본리전(ap-northeast-1)의 로그인 페이지 [`https://ap-northeast-1.signin.aws.amazon.com/`](https://ap-northeast-1.signin.aws.amazon.com/) 로 시작 하는 경우

Apple Script에서도 shell script를 실행 할 수 있지만 bashrc같은 곳에 적용 해놓은 env등이 적용이 안되기 때문에 해당 기능은 hammerspoon으로 분리해서 관리 했다. 만약 분리하고 싶지 않다면 위에서 bashrc에 적용해 둔 각 function을 shellscript로 만든 후 PATH에 추가 해주면 될 것 같다.

![https://drive.google.com/uc?export=view&id=1Yg36lHIDNLJb--mh7QOclm3eVPyrZTpB](https://drive.google.com/uc?export=view&id=1Yg36lHIDNLJb--mh7QOclm3eVPyrZTpB)

기본적으로 apple script는 일 회 실행되고 종료되게 돼 있지만 idle 구문을 통해서 백그라운드에서 계속 실행되게 할 수 있다. 스크립트를 실행 파일로 만들려면 Mac의 `스크립트 편집기` 를 열고 붙여 넣은 다음 `파일` → `내보내기` 에서 파일 포맷을 `응용 프로그램`으로 설정 후 저장 해주면 된다. 그리고 옵션에서 반드시 `처리기 실행 후에 열어놓기`를 체크해야한다.

![https://drive.google.com/uc?export=view&id=1MlTb4N3lL6OeLoTkz0ENMIfDlvqkjD6E](https://drive.google.com/uc?export=view&id=1MlTb4N3lL6OeLoTkz0ENMIfDlvqkjD6E)

그리고 스크립트 자제가 UI elements 들에 접근하고 제어를 하기 때문에 손쉬운 사용에서 응용 프로그램 화 시킨 스크립트를 추가 해 줘야한다. `시스템 환경설정` → `보안 및 개인정보 보호` → `개인 정보 보호` → `손쉬운 사용` 에서 `+` 버튼을 눌러서 추가해주자.

이렇게 스크립트를 잘 저장해줬다면 Hammer spoon config에서 chrome_tracking을 실행 시킬 수 있도록 추가 해 주면 부팅 시 스크립트가 실행되도록 할 수 있다.

```lua
hs.execute("open /Applications/chrome_tracking.app")
```


> Known Issue : 다른 리전의 로그인 창이 뜬 경우는 안된다. contain을 이용해 해결 할 수 있지만 나는 일본 리전만 사용하고 있기 때문에 그냥 strats with으로 해결했다.

작업에 필요한 소스코드들은 여기에 모아두었다. [https://abr.ge/odw1td](https://abr.ge/odw1td) 

## 후기

간단한 자동화지만 아침에 AWS 자동으로 로그인 될 때 마다 스스로 뿌듯함을 느낀다.
미루지 말고 한 번 시도해보자.

---

## 참고자료

**Generating Authy passwords on other authenticators**

[https://gist.github.com/gboudreau/94bb0c11a6209c82418d01a59d958c93](https://gist.github.com/gboudreau/94bb0c11a6209c82418d01a59d958c93)

**mac-oath-mfa**

[https://github.com/alanplatt/mac-oath-mfa](https://github.com/alanplatt/mac-oath-mfa)

