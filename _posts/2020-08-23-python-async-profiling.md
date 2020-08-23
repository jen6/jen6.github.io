---
layout: post
title: "Profiling을 통해 python async web application 병목지점 찾기 ⛑"
description: ""
category: 
tags: ["profiling", "yappy", "python", "성능개선", "프로파일링", "performance"]
comments: true
thumbnail: https://drive.google.com/uc?export=view&id=13QI6d3R7xxsK011j35Lj0fJ26wlUJbup
---
## Prologue
이번에 회사내에서 병목 지점이라고 여겨지던 db문제를 개선하기 위해 async db 라이브러리 사용하는 작업을 했었다. 하지만 async db를 사용하더라도 큰 성능 향상이 없어서 이게 내가 짠 코드의 문제인지 아니면 라이브러리가 병목인건지를 밝혀내고 싶어서 프로파일링을 해봤다.

회사에서는 APM으로 newrelic을 사용하는데 flask 기반의 WAS를 async로 동작하는 sanic으로 바꾼 후 함수가 안나온다거나 정확히 profiling 되지 않는 문제들이 존재했다. 왜인지는 모르겠지만 python code profiling을 할 때 가장 많이 쓰는 cProfiler에서도 같은 문제가 발생하고 있었고 다른 대안이 필요했다. 

1. Asyncio profiling을잘 지원해야한다.
2. 디버거를 붙이더라도 속도가 빨라야한다.
3. 결과를 보기 쉬워야한다.

내가 찾아본것 중 위 세개의 조건을 만족하는 프로파일러는 “yappi”라는 프로파일러가 유일했다. C로 짜여져 있어 속도가 빠를 뿐더러 asyncio를 profiling도 잘 했고 여러 file-format으로 저장 가능해서 시각화 툴에서 결과를 보기도 좋았다.

[https://github.com/sumerc/yappi](https://github.com/sumerc/yappi)

이 글에서는 profiling을 yappi로 어떻게 하는지 병목지점을 찾고 개선하는 프로세스를 다룬다. 

## 💿설치

```bash
pip install yappi pyprof2calltree
brew install qcachegrind
```

yappi는 위에서 말했다 싶이 프로파일러 이고 qcachegrind와 pyprof2calltree는 python profiling 결과를 시각화 해서 좀 더 보기 좋게 만들어 주는 툴 이다. 프로파일링 결과를 터미널에서 텍스트로 봐도 되긴 하지만  단순히 숫자로 보는 것 보다는 qcachegrind에서 지원하는 call graph를 보면서 하면 좀 더 분석하기 수월하다.

## 📊yappi를 이용해 프로파일링 하기

yappi에서 다양한 프로파일링 옵션을 지원하고 해당 옵션들은 깃헙에 있는 example에 잘 설명 되어있으니 
이 글에서는 다양한 옵션들 보다는 어떻게 프로파일링을 하는 과정에 대해 설명할거다.

아래는 web server를 local에서 돌리는 코드에 yappi profiling을 하는 코드를 넣어봤다. 이렇게 하고 실행을 끝내게 되면 실행시킨 폴더에 callgrind.profiled라는 파일 명으로 분석결과가 나온다.

```python
from os import environ
from app import create_app
import yappi

if __name__ == "__main__":
    sanic_app = create_app()

    yappi.start()

    try:
        sanic_app.run(
            host="0.0.0.0",
            port=8000,
            workers=1,
            debug=False,
            access_log=False,
            backlog=30000,
        )
    except Exception:
				pass
		finally:
        yappi.stop()
        yappi.get_func_stats().save("callgrind.profiled", "pstat")
        print("saved!")
```

 테스트를 할 때 병목 지점이 좀 더 명확하게 보이게 하기 위해 로컬에서 ngrinder를 써서 스트레스 테스트 진행했다. 

## 🔍프로파일링 결과 보기

이제 qcachegrind를 통해 시각화된 결과를 보자!

```bash
pyprof2calltree -k -i ./callgrind.profile
```

pyprof2calltree는 python profiling 결과 형태인 pstat을 qcachegrind가 읽을 수 있는 형태인 callgrind 형태로 변환시켜주는 역할을 한다. yappi가 callgrind 형태로 저장을 할 수 있긴 한데 그래프가 이 방법이 더 보기 좋게 나왔던걸로 기억한다.

qcachegrind 창이 켜지면 가장 왼쪽에 보이는 창이 프로파일링 된 함수들 리스트다. 항목중에서 설명이 필요한 것만 설명을 하자면 

- Self : Callee들의 실행시간을 포함하지 않은 현재 메소드의 실행시간

- Incl : Callee들의 실행시간을 포함한 실행 시간 

- Called : 이 메소드의 호출 횟수

![https://drive.google.com/uc?export=view&id=1muPOdDh_rcrbzNohWlujTTmizYpu9Qgo](https://drive.google.com/uc?export=view&id=1muPOdDh_rcrbzNohWlujTTmizYpu9Qgo)
이 항목에서 분석 원하는 함수를 누르게 되면 오른쪽 화면에서 call graph 혹은 callee들이 어떤게 있는지를 볼 수 있다

![https://drive.google.com/uc?export=view&id=1VPEJHVkaPu1XuRyNgz6DYPCy44pK2AVM](https://drive.google.com/uc?export=view&id=1VPEJHVkaPu1XuRyNgz6DYPCy44pK2AVM)
상단에 리스트에도 어떤 함수가 몇번 호출 됐는지 등 텍스트로 여러 정보를 볼 수 있지만 분석할 때 보기 편하다고 생각하는건 하단에 그래프로 돼있는 'Call Graph'였다.

## 🍾병목 지점 찾아내기

성능 개선을 해볼 타겟은 TrackingLinkRepository이고 Database에 접근하는 역할을 한다. sanic으로 바꾼 후 request는 concurrent하게 처리 할 수 있게 됐지만 mysql client는 sync client였기 때문에 다른 request들을 blocking 할 수 있을 것 같아 async db client로 바꾸는 시도를 했었다. 하지만 예상 외로 성능은 sync db client 보다 더 한참 안나왔다.

![https://drive.google.com/uc?export=view&id=17tb0dhJho0Ctl5VHW-FrX2dYmF8ToZCN](https://drive.google.com/uc?export=view&id=17tb0dhJho0Ctl5VHW-FrX2dYmF8ToZCN)

db에 접근하는 부분인 `get_first_from_cache`가 크게 두가지 갈래가 있는데 connection을 맺는 
`FlaskDB.aio_engine`이란 함수는 크게 오버헤드가 없어보이고 커넥션에서 실제로 쿼리를 보내 `SAConnection._execute` 부분에 오버헤드가 있어보였다.

![https://drive.google.com/uc?export=view&id=1J5kXY0ir6wYRG-w9JqPLikuwwJdMKKWw](https://drive.google.com/uc?export=view&id=1J5kXY0ir6wYRG-w9JqPLikuwwJdMKKWw)

![https://drive.google.com/uc?export=view&id=1boAVsRGXf5ypul8SUIM0Cn37wt8H3BkG](https://drive.google.com/uc?export=view&id=1boAVsRGXf5ypul8SUIM0Cn37wt8H3BkG)

코드를 다시 한 번 보니 쿼리를 빌드 할 때 캐싱을 안하고 매번 빌드하고 있었다. 찾아보니 mysql client에서 쿼리 빌드에 대한 캐시를 남길 수 있는 옵션이 있어서 적용했다. 이거 외에도 statement를 빌드 할 때 where절 조건 중 bind parameter를 이용해서 캐싱할 수 있는 statement를 만들어서 쿼리 빌드를 다시 하지않도록 코드를 수정했다.

## 🧰1 차 성능 개선 결과

![https://drive.google.com/uc?export=view&id=1ZRwc-MMsFXv-mKoLfs3Sm8y1-UzLKa_l](https://drive.google.com/uc?export=view&id=1ZRwc-MMsFXv-mKoLfs3Sm8y1-UzLKa_l)

쿼리를 빌드하는 부분은 개선 돼서 이제 calltree에서 안보인다!  하지만 여전히 성능은 안나왔고 자세히 보니 call count가 다른 db치는 함수들에 비해 높았다. 

![https://drive.google.com/uc?export=view&id=1qzhUABa7tJnG0V5vkX76yGRx791Oz31g](https://drive.google.com/uc?export=view&id=1qzhUABa7tJnG0V5vkX76yGRx791Oz31g)

코드를 읽어보니 함수 결과를 캐싱하는 부분에 문제가 있어 매번 db를 치고있었다. 기존에는 캐싱할 때 key에 session객체를 같이 넣어서 매번 다른 key 값이 나왔었고 where절에 넣을 조건들로만 캐싱을 하니 정상적으로 됐다.

## 📈최종 결과

![https://drive.google.com/uc?export=view&id=177XawCsVUSBCrAS-aRoTf2cytB5gycBO](https://drive.google.com/uc?export=view&id=177XawCsVUSBCrAS-aRoTf2cytB5gycBO)

CallGraph는 일정 시간 이상 소비되지 않은 노드들은 제외되기 때문에 노드가 몇 안남은걸 보면 실행시간이 많이 개선된걸 알 수 있다. 결과적으로는 이번에 개선한 async함수의 성능이 sync 함수랑 비슷한 수준까지 올라왔지만 약간 더 나쁜상태라 적용할 수가 없었다.. 

나중에 asyncio 관련된 글을 적을 때 좀 더 자세히 적을 생각이지만 짧은 실행시간을 가진 함수를 비동기로 할 때 오히려 context switching에 대한 overhead가 더 크기때문에 오히려 성능 하락이로 이어진것 같다.

그래도 프로파일링을 통해 알 수 있는점은 내 잘못인가? 라이브러리 탓인가? 이 부분이 더 개선 가능한가?  와 같은 어려운 질문들에 대한 대답을 어느정도는 구할 수 있다는 것 이다. 그냥 성능이 안 나왔을 때 라이브러리 탓이겠거니 했던걸 callgraph나 profiling상에서는 이전에 있던 함수랑 거의 유사해진걸 봐서 더 이상 내가 손댈 수 있는 부분이 없겠구나 하고 스스로 결론을 내릴 수 있었다. 물론 더 개선하면 좋아질 수 있겠지만... 아직 나의 한계는 여기까지...
