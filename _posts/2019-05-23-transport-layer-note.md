---
layout: post
title: "Transport Layer Note"
description: ""
category: 
tags: []
comments: "true"
thumbnail: https://cdn.pixabay.com/photo/2012/04/02/16/33/modem-24894_960_720.png
---

# Network

# Chapter 1

### Network protocols

define format, order of messages sent, received entities, actions token on transmission 
(token ring같은 경우를 말하는듯)

### Network structure

- Network Edge : hosts client and servers
- On DSL(Digital Subscriber Line) using frequency division multiplexing
  (음성, 데이타 주파수 분리)
- Host sending packet : packet : L bits, link transmission rate(게bandwidth) : R (bit/sec)
<!--break-->
                                        packet transmission delay = L/R
- Network Core : interconnected routers, network of networks
- packet-switching : application layer 메서지를 패킷단위로 쪼개 다음 라우터에게 보내줌
- store and forward : 패킷 전체가 수신되기 전까지는 forward할 수 없음.
- end-end delay : 2L/R
- arrival rate가 transmission rate를 넘어가게 되면 패킷이 큐에 들어가서 기다림. (queueing delay)
   라우터 메모리가 꽉차게 되면 packet dropped. (loss)
- Circuit switching(Alternative core)
- src/dest 간의 경로가 보내기전에 결정되고 모든 리소스를 사용함. 안쓰이면 idle 공유따윈 없음
- FDM(주파수 별로 다른 유저가 사용), TDM(시간을 쪼개서 유저간 공유)
- Packet Switching vs Circuit Switching

[http://gaia.cs.umass.edu/kurose_ross/interactive/ps_versus_cs.php](http://gaia.cs.umass.edu/kurose_ross/interactive/ps_versus_cs.php)

35명 전체 유저 중 전체 유저가 각각 한번에 시간을 10%씩 점유할 때 10명 이상의 사람이 동시 접속 할 확률

[Probability problem in networking.](https://math.stackexchange.com/questions/918861/probability-problem-in-networking)

[http://icawww1.epfl.ch/sc250_2004/lecture_notes/sc250_exo2.pdf](http://icawww1.epfl.ch/sc250_2004/lecture_notes/sc250_exo2.pdf)

![](http://drive.google.com/uc?export=view&id=16HdP_F4x1C6U-ZJ5rKqSri40C-4DpD5z)

- circuit switching은 큰 데이터(video)같은걸 전송할 때 좋음. packet switching은 대역폭 보장

### Internet Structure

- Network evolution was driven by **economics** and **national policies**
- 소규모의 Network는 **ISP**를 통해 연결됨. ISP는 상호간의 연결이 필요함
- Access ISP는 호텔, 회사, 대학에서 제공하는 네트워크를 의미
1. N개의 **Access ISP**(Node)가 있을 때 Interconnect 하려면 n(n-3)/2 + n O(n^2) scale
2. 그래서 **Global ISP**가 출현해서 ISP간을 연결해주는 Network를 만듬.
3. Global ISP가 여러개 출현하면서 Global ISP간을 이어주는 **IXP**(Internet Exchange Point)가 나옴.
4. Access ISP를 묶어서 Global ISP에 연결해주는 **regional networks**들이 등장
5. **Content Provider**(Google, Akamai)들이 **자체 네트워크**를 구성함

### Delay, Loss, Throughput

- Packet Delay Sources 
- **Transmission Delay** : 라우터로 들어오는 패킷에 대한 딜레이 (L/R_in)
- Nodal Processing Delay : 패킷 헤더 조사, 라우팅 경로 결정 (매우 짧은시간)
- Queueing Delay : 큐에 들어가서 output link로 나가기까지의 딜레이 (congestion 상황에 따라 다름)
- **Propagation Delay** : 매질을 통해 실제 데이터가 전송되는 속도 
   Ex. 라우터간 거리 200km, 전기의 속도 2*10^8m/sec → 2*10^5 / 2*10^8 = 1/1000 sec = 1 ms
- Packet Loss : 라우터에 큐가 꽉 차면서 발생하는 문제.
- Throughput : 1초에 몇 비트를 받고있는가? 전송률 (bits/time unit)
 - instantaeous : 순간 전송률
 - average : 평균 전송률
 - 전송량 Rc, Rs이 있고 전달해주는 Core의 전송량이 R이고 열명이 나눠 쓸 경우
    throughput은 min(Rc, Rs, R/10)

[http://gaia.cs.umass.edu/kurose_ross/interactive/end-end-throughput.php](http://gaia.cs.umass.edu/kurose_ross/interactive/end-end-throughput.php)

---

# Chapter 3 Transport-layer

### Transport-layer services

다른 host에 있는 **Application**이 서로 통신 할 수 있는 **Logical communication**  
Network layer은 다른 **Host** 끼리 서로 통신할 수 있는 것 **구별하기**
break message into segments and reassemble it

### Multiplexing / Demultiplexding at Transport Layer

Transport Layer에서 어떻게 데이터를 처리해줄것인지에 대한 내용
Network Level에서 IP Datagrams를 받음. 여기에는 host, dest ip 정보가 들어있음.
그리고 Datagram한 개에는 Transport Layer segment가 들어있고 여기 헤더에 port정보가 들어있음.

multiplexing : L7 → L4 일 때 여러개의 Application Level socket에서 들어온 데이터들에 transport header를 추가 해줘서 어떤 포트로 갈지에 대한 정보를 넣어 줌
demultiplexing : L4 → L7 헤더에 있는 포트 정보를 보고 해당 프로세스 소켓으로 넘겨줌

**When demuliplexing** 
UDP는 기본적으로 Connectionless 그냥 **dest ip, port**만 알게 되면 보낼 수 있음. src ip,port 와 상관 없이 dest ip, port만 알게되면 같은 소켓으로 통신가능.
TCP는 그와 반대로 Connection orientied  **src ip,port 그리고 dest ip, port** 총 4개를 이용해 소켓을 구분. 저 4개 중 한 개라도 달라지면 다른 소켓을 사용.

---

### UDP

**Connectionless** : No handshaking, independent UDP segment.
순서 없음. 수신 송신에 대한 확인이 없기 때문에 loss발생 가능. → No reliability
(Application Level에서 하려면 할 수 있긴함.)
하지만 이런 특성 때문에 간단해서 많이 쓰임. Congestion Control이 없기 때문에 원하는 만큼 보낼 수 있음.

![](http://drive.google.com/uc?export=view&id=1ihK-svJ6qkG3QNziDPIwhYrglCtwjubo)

([https://tools.ietf.org/html/rfc768](https://tools.ietf.org/html/rfc768))

Source Port is an optional field, when meaningful, it indicates the port
of the sending  process,  and may be assumed  to be the port  to which a
reply should  be addressed  in the absence of any other information.

### Checksum

UDP헤더를 보면 Checksum이 존재하는데 IP header의 일부, UDP header, UDP data를 가지고 한다.
문제는 Transport Layer에서는 IP header에 대한 정보를 안가지고 있기 때문에 정보중 일부인 persudo header를 만들어 계산한다. → TCP도 같은 방법으로
[http://www.netfor2.com/udpsum.htm](http://www.netfor2.com/udpsum.htm) ← checksum calculation code

> 흠,,, L3 switching 에서 DSR(Direct Server Return)을 하게 될 경우 
dst ip addr을 vip로 바꿔주게 될텐데 그러면 다시 checksum을 업데이트..
[http://tech.kakao.com/2014/05/28/l3dsr/](http://tech.kakao.com/2014/05/28/l3dsr/) 그냥 갑자기 난 생각

![](http://drive.google.com/uc?export=view&id=17ymVpuFihSOUKPYurKw-0uRgTFLkSz2f)

이때 one's complement를 사용해서 계산을 한다. left most에 carry가 발생하면 더해주는 방식으로.
checksum에 들어가는거는 마지막의 one's complement를 취해준다(not)
IPv4기준으로 checksum은 optional. 0으로 넣어두면 검사를 안한다는 뜻으로 사용한다.

![](http://drive.google.com/uc?export=view&id=1S7QTkE_I1AlromhK68esoQTHJ5upbAu8)

---

### RDT(Reliable data transfer)- 중요

Reliable data transfer은 좀 일반적인 의미로써의 프로토콜로 어떤 프로토콜이던 spec을 맞추면 
rdt protocol이라 불릴 수 있음. 
The requirements are **retransmission**, **error detection**, and acknowledgments.
PPT상에서는 Unreliable data transfer send함수를 랩핑 해서 하는 식으로

### rdt 1.0

그냥 안전한 채널(no loss, no bit error)에서 전송하면 그 자체로 안-전

![](http://drive.google.com/uc?export=view&id=19HT83cxO8S56K8YTP87o6FMra0FmoT6t)

### rdt 2.0

조건 : bit error 발생하는 상황. no loss. 
→ checksum 추가. 
    feedback 추가 : 체크섬이 이상할경우 NACK (Retransmission), 괜찮으면 ACK

문제점 : ACK/NACK 가 corrupt되는 상황이 발생할수도 있음

![](http://drive.google.com/uc?export=view&id=1J0gw4AXvtwJbjDsC97JHLV0NIUDuaAPf)

### rdt 2.1

해결책
 - ACK/NACK에 Checksum을 추가해서 error검출. corupt가 발생시 데이터 재전송
 - Sequence number를 추가해서 recv측에서 duplicated data를 검출해냄

질문

- 과연 NAK가 필요한가? → 1번 State에서 corrupt가 일어났을 때 NAK대신 0번 ACK를 받는다면?
   (실제로 여러 TCP Implementation에서 쓴 receipt "Triple duplicate ack"   
→ TCP Fast retransmittion) : 2번 중복된 ACK가 오더라도 timeout 되기 전까지는 no retransmission
     But 3번 중복된 ACK가 오게 되면 Fast retransmission. (SR ARQ)
- 이전보다 2배 늘어난 state

![](http://drive.google.com/uc?export=view&id=1pxKseLNwrvLVycgX69qedQVif9B-8SXU)

![](http://drive.google.com/uc?export=view&id=1QqH4i6bDwKA8czgrzcU3U635EvXkPDTB)

### rdt 2.2

NAK-free protocol : ACK만 쓰임. corrupt 발생시 이전 ACK를 보냄. → ACK에 sequence 필수 포함

![](http://drive.google.com/uc?export=view&id=1sfIz3Nnn9_5BcEICz7DPKZiPC39aO3rn)

![](http://drive.google.com/uc?export=view&id=1Zo_UfQ2wrVi34HaMXxYllYzwX8UZFl6R)

### rdt 3.0

조건 : bit error 발생, loss 발생.
해결 : **timeout**을 둬서 시간안에 ACK가 안올경우 retransmission. 

![](http://drive.google.com/uc?export=view&id=11OhF73N281J2R2xlODZOTgN-XbPIXSLi)

Stop&Wait 분석: 1 Gbps Link,  15ms propagation, 8kb packet

→ Transmission Time = 8*10^3bit / 10^9 bit/sec = 8 / 10^6 sec = 8000 ms = 8usec (micro second)
Utilization : 전송에 쓰인 시간 / (총 걸린 시간)
→ Transmission Time / (RTT + Transmission Time) = 0.008 ms / (30.008) ms = 0.00027

---

### Pipelined protocols

문제 : 하나 보내고 time out을 기다리던가 하는건 솔직히 말이 안됨. 여러 개를 보낼 수 있는 방법을 생각.

해결 : **Pipelining**을 사용해서 여러개를 한꺼번에 보내자
→ 여러개를 보내기 위해서 **Sequence number를 확장**해야함. 각 패킷마다 unique한 number가 필요
→ **Buffer**를 확장해야함. 이전에는 한개만 가지고 했지만 이제는 pipeline 할때 필요한 사이즈 만큼 필요

효과 : 3개씩 보낸다고 하면 Utilization이 3배 확장!

이제 밑에서 기술하는 protocol들에서는 **Sliding window**라는 개념이 들어간다. 위에서 얘기한 buffer와 
같은 얘기로 0~10까지 보내야 할 패킷들이 있을 때 buffer size가 3이라면 (0, 2) ~ (8, 10) 이렇게 움직이며 
처리를 한다.

Pipelined protocols를 볼때는 세가지 event point를 중점적으로 봐야한다.

1. Send invocation : send함수가 호출 될 때 sequence number과 sliding window status
2. Receipt of an ACK : ACK를 받을 때 어떻게 처리 하는지
3. Timeout Event :  Timeout을 받을 때 어떻게 처리 하는지

### GBN (Go-back-N)

처음 Send가 호출 됐을 때 window가 꽉 찼다면 거-절. 아니면 윈도우에 넣고 패킷 전송.

N개의 패킷을 받을 때 마지막에 잘 수신된 패킷의 sequence number ack를 보냄. (**cumulative ack)**

→ 0~5 까지의 패킷을 보낸다고 했을때 0, 1, 2 ack를 보낼 수도 있지만 2 ack만 보내게 되면 2 이전의 것까지
다 잘 도착한 상황이라고 가정

Timeout의 기준은 제일 오래 ACK를 못받은 패킷(윈도우 맨 처음) 발생시 timeout된것 부터 전부 다시 전송
→ timer은 가장 마지막 패킷 기준으로 한 개. 어차피 윈도우 전체를 다시 재전송 하는데..

Error가 발생하거나 순서대로 오지 않을 경우 이전 sequence number를 포함한 ACK를 보냄.
**out-of-order discard** : 2번까지 ACK를 보낸 후 3번을 못받고 4번이 먼저옴. 그러면 discard하고 2번 ACK를 보냄

duplicated ack를 받게 될 경우 무시. sender은 해당 ACK가 오지 않을 경우 timeout이 날 때 까지 기다렸다가 retransmission.

→ GBN은 rdt프로토콜들의 명세가 다 지켜지고 스펙이 좋기 때문에 후에 TCP에서 GBN Style을 많이 사용.
그러나 segment가 순서대로 오지 않아도 buffer상에는 순서대로 들어감. + SR(Selective Repeat)

### SR (Selective Repeat)

send가 호출 됐을 때 남아있는 Sequence number가 있는지 확인한다. 남은 sequence number가 현재 sender의 윈도우에 범위 안에 있다면 바로 보내고 아니라면 buffering하거나 다시 상위 레이어로 올린다.

N개의 패킷을 받을 때 각각 패킷에 대해 모두 ACK를 보낸다.. (**individual ACK**)

각각 ACK가 안와서 timeout된 패킷에 대해 다시 재전송.  → unACKed packet 갯수만큼 타이머 필요

문제점 : sender와 receiver간의 **sliding window synchronization** 

윈도우 사이즈가 3, Sequence Number범위가 0~3일 때 아래와 같이 ACK가 drop될 경우 synchronization이 깨지게됨. 슬라이딩 윈도우가 어떻게 움직이는지는 보이지 않고 Sequence number가 
어떻게 오는지만 알 수 있기 때문에 이런 문제가 생김.
이게 지금 윈도우의 못 받은 0인지 아님 이전 윈도우의 0인지를 구분할 수 없는 문제.

최소 Sliding window size *2 ≤ Sequence number size가 돼야 이 문제를 해결 할 수 있다.

![](http://drive.google.com/uc?export=view&id=1nhUksi0iIeoCCsqKJQ0X0iou5A0PpgDA)

---

## TCP(Transmission Control Protocol)

**Connection-oriented** : 맨 처음 서로 연결 할 때 **establish**과정이 필요함. → **handshaking**
establish 함으로써 서로 state variables들을 초기화 한다.

**Full duplex data** : 양방향 통신으로 서로 데이터 통신을 할 수 있다. 

**Point to Point** : 양 끝 단말기기간 둘이서만 통신을 한다. → multicasting은  TCP로 불가능

**Flow controlled** : sender가 receiver의 buffer를 과부화 시키지 않음

### TCP segment structure

![](http://drive.google.com/uc?export=view&id=1GsFmE40p5OSWv5m3wmeatkPCv7Duwlml)

- Src, Dst port each 16bits
- Sequence Number 32bits : 이 세그먼트의 첫번째 바이트의 순서
→ SYN은 ISN(initial sequence number)를 뜻하고 맨 첨 데이터의 바이트는 ISN+1

    If SYN is present the sequence number is the
        initial sequence number (ISN) and the first data octet is ISN+1.

- Acknowledgement Number 32bits : 이 다음 받을 세그먼트의 예상 sequence number
- Data offset 4bits : 4byte 단위로 TCP 헤더의 길이 및 데이터 시작 offset을 알려준다.
- Reserved 6bits : 예약석 0으로 셋팅
- Control Bits 6bits
 - URG : Urgent Pointer 필드가 사용될 때.
- ACK : Acknowledgment 필드가 사용될 때.
- RST, SYN, FIN : connection establishment
- PSH : Push Function 상위 레이어로 데이터를 바로 보냄 
- URG : 긴급한 데이터가 있다고 표현할 때? Urgent Pointer랑 같이 쓰임 (PSH랑 안중요)
- Window 16bits : Flow control에 쓰이는 size. 
그냥 간단하게 receiver가 받고싶어하는 세그먼트 크기라고 생각
- Checksum 16bits : 데이터 체크섬. udp랑 방식이 같다.

밑에는 안중요해서 패스

---

### TCP RTT

RTT < TimeOut 이여야 패킷이 정상적으로 도달 할 수 있는데 RTT가 워낙 많은 경우의 수가 있다.
그러면 대략 측정을 어떻게 할 것 인가? 

→ **Sample RTT :** 데이터를 보낸 이후 ACK가 돌아오기 까지를 걸린시간을 체크. 
근데 이 값은 network congestion에 따라 값이 많이 바뀐다. 그래서 이 값을 그냥 쓰진 않는다.
**Estimate RTT** : (1 - a)*EstimateRTT + a * SampleRTT
각각 SampleRTT를 지수이동평균 (Exponential Moving Average)를 구해준다.
지수 이동 평균은 최근 값에 높은 가중치를 주고 이전 값 또한 영향을 미칠 수 있게 한다.
위 식에서 가중치 a 는 일반적으로 0.125 라는 값을 많이 쓴다.
근데 이 값은 평균이지  RTT보다 크다는 보장이 없다. 실제 그래프를 보게 되면 평균 정도라
정반정도가 timeout이 날 수도 있다.
**DevRTT** : (1-b)*DevRTT + B*|SampleRTT - EstimatedRTT| 
이 값은 safety margin을 주기 위한 값으로 평균과 현재 값의 차를 계속해서 평균을 내 놓는다.
여기서 가중치 b는 일반적으로 0.25 값을 사용한다.
**TimeoutInterval** = EstimatedRTT + 4*DevRTT ( 최종 값 )

---

### TCP RDT

TCP 또한 rdt를 하기 위해 기능들을 추가

1. Pipelined segments
2. Cumulative acks
3. single retransmission timer

Retransmission Condition : timeout, duplicate ack
Timeout : retransmit unacked 패킷중 sequence number가 제일 작은 것 부터

Sender가 패킷을 보낼 때 Seq에 현재 Sequence Number 이랑 data size를 보내준다.
Receiver은 잘 받았을 경우 ACK에 다음 받을껄로 예상대는 Sequence Number를 보낸다.
위에 써놨다 싶이 Cumulative acks가 적용된다.

Receiver은 out-of-order packet을 받았을 때 원래 받아야 할 Sequence number를 포함한 ack를 보내준다.

이 과정 중 Timeout delay가 너무 길 경우 **TCP fast retransmit**을 이용한다.
lost segment에 대한 ACK를 3 번 보낼 경우 sender는 timeout 때와 같은 행동을 한다.

---

### Flow Control

 |OS| TCP Code → TCP socket recv buffer  → |USER| Application Process 

이런 방식으로 구조가 돼 있을 때 unordered segment를 처리 하지 않는다는 가정하에 ordered segment를 받았을 경우 recv buffer에 들어가게 된다. 그러면 Application Process에서 Recv를 통해 버퍼에 있는 데이터들을 가져간다.  그치만,,, Application Process가 처리할 양이 많다면 recv buffer가 꽉 차게될 수도 있다. 이렇게 되면 패킷이 drop되고 다시 전송 받아야한다. 

이러한 이유에서 **flow control**이 만들어졌다.

![](http://drive.google.com/uc?export=view&id=1bkW1_oNCFEgzZIcKUmevWFLy2wdQLExC)

위 그림이 Receiver의 버퍼이다. 
RecvBuffer : 전체 버퍼의 크기 일반적으로 4096bytes
RecvWindow (rwnd) : buffer에 남은 공간으로 이 공간을 Window Size로 정해준다.

Receiver : 자신이 남은 RcvWindow를 rwnd에 담아서 보내준다

Sender : Rwnd 만큼 보내준다 했을 때 **LastByteSent, LastByteAcked** 이 두가지 변수를 고려해야 한다.  `LastByteAcked - LaskyteSent ≤ Rwnd` 여야 하기 때문에 이 값을 세션이 유지되는 동안 유지해야한다. 

이걸 지키면 overflow나지 않는 걸 보장 할 수 있음

---

### Three-way Handshaking (Connection)

![](http://drive.google.com/uc?export=view&id=1LFefF1l7ANSPkA9tbl9sgOTXcip_90Yb)

여러 호스트를 받을 수 있는 UDP랑은 달라달라달라~ 
TCP는 sender, receiver당 한개의 Connection이 만들어진다. 그래서 이 Connection을 맺기 위한 과정을 **handshake**이라고 한다. 

이 과정은 세 번에 걸쳐서 진행된다고 해서 **Three-way handshaking**이라고 불린다.

TCP A <------> TCP B

1.     ———SEQ (ISN(initial Sequence Number) A) CTL<SYN> → 

2.     ←———SEQ(ISN, B), ACK(ISN A + 1), CTL<SYN, ACK> -

3.      ———SEQ(ISN, A+1), ACK(ISN B + 1), CTL<ACK>—>

다음과 같은 과정으로 진행된다. 
이렇게 3번에 걸쳐서 하는 이유는 서로의 **ISN(Initial Sequence Number)**를 동기화 시키기 위해서이다. 또한 2-way로 할 경우 서로의 state를 못 보게 된다. connection initiation 과정에서 네트워크 상황 때문에 오래된 Duplicate connection request는 **half-open** 상황이 일어나게 한다. 즉 한 쪽만 open되고 나머지 한쪽은 이미 timeout되서 커넥션이 일어나지 않을 수도 있는 상황을 뜻 한다.

하지만 three-way hand shaking 은 SYN-ACK, ACK 이 과정에서 상대방에 state를 알 수 있어 **Simultaneous Connection Synchronization**과 **duplicated initialization** 등에 대해 대응할 수 있다.  잘못된 커넥션은 RST condition을 통해 끊어준다.
자세한 내용은 [https://tools.ietf.org/html/rfc793#page-31](https://tools.ietf.org/html/rfc793#page-31) 에서 볼 수 있다. 

Session Closing 과정은 **Four-way handshaking**이라 불린다.

![](http://drive.google.com/uc?export=view&id=1XzYkdo9dp2bibD60L4d8d78s0xeIZfOL)

이 과정은 양쪽 모두 닫는걸 목적으로 하기 때문에 FIN, ACK 를 양쪽에서 한 번 씩 보내준 후 각 요청에 대해 ACK를 한 번 씩 보내준다.

---

## Congestion Control

Flow Control이 Host의 buffer가 overflow될 수 있는 상황이였다면 **Congestion**은 Network 상황이 안좋다는 뜻이다. 여기서 Network의 상황이 안좋다는 것은 Router의 link buffer들이 꽉 찰 수 있다는 것을 뜻 한다.

cwnd( Congestion Window ) : sender-slide limit, ack를 받기 전까지 얼마나 많은 양의 데이터를 보낼 수 있는지. 이 값은 알고리즘에 의해 정해지는 값이다.
rwnd (Receiver's advertised window) : receiver-side limit, 얼마나 많은 데이터를 감당 할 수 있는지
SMSS(Sender Maximum Segment Size) : Sender가 전송할 수 있는 가장 큰 segment size. network의 MTU(Maximum Transmission Unit)에 따라 결정된다.
RMSS(Receiver Maximum Segment Size) : Receiver가 받길 원하는 가장 큰 segment size.  Connection이 맺어질 때 이 정보를 MSS option에 같이 보낸다.

ssthresh(Slow Start Threshold) : slow start를 할지 Congestion avoidance를 사용할지를 결정하는 변수.

### Slow Start & Congestion Avoidance

Slow Start와 Congestion Avoidance는 기본적으로  network에 capacity를 추측하는 알고리즘이다.
이 알고리즘에 따라서 변화하는 두 변수 **cwnd, ssthresh**가 어떤 이벤트에 따라서 값이 변화하는지를 알아보자.

먼저 현재 네트워크 상황을 모르기 때문에 데이터를 천천히 보내는 것 부터 시작한다. (Slow start) 그렇다면 처음에 아무것도 안 보내줄 수는 없기 때문에 cwnd에 초기값을 2*SMSS보다 작거나 같게 설정 해준다. ssthresh는 값이 조금 커야한다. `ssthresh = max (FlightSize / 2, 2*SMSS)` 이정도로 설정한다고 한다.

이렇게 초기값이 잡히고 나면 ACK가 돌아올 때 마다 `cwnd = min(rwnd, 2*cwnd)` 로 설정을 해준다. 이 과정은 `cwnd < ssthresh`일 때 까지 계속 증가한다. 이 과정을 **Slow Start** 라고 부른다.

cwnd > ssthresh인 상황이 오게 되면 **Congestion Avoidance**상황으로 가게 된다. 네트워크가 congestion되지 않도록 천천히 보내는 경우이다. 이 때 부터는 ACK가 올때 마다
 `cwnd = cwnd + SMSS*(SMSS/cwnd)` 즉 ACK가 윈도우 갯수 만큼 올 때 1씩 증가한다. 이렇게 쬐끔씩 올라가는 과정을 **Additive Increase** 라고 부른다. 혼잡해지지 않게 조심 또 조심해주는 것 이다.

그리고 Congestion Avoidance 에서 timeout이 일어나게 되면 congestion이 발생했다는걸 감지하고 `ssthresh = cwnd/2` , `cwnd = 1*SMSS` 다음과 같이 설정하고 slow start로 다시 돌아가게 된다. 즉 cwnd를 최솟값부터 해서 congestion 상태를 최소화하는것이다. 이렇게 window를 확 줄이는것을 **Multiplicaive Decrease** 라고 부른다. 그리고 이렇게 window를 제어하는 알고리즘을 **Addictive Incerease & Mulitplicative Decrease** 라고 부른다. 

![](http://drive.google.com/uc?export=view&id=1qtmanmW7HYGx047ZaNmxOwEF8mbz5JpM)

### Fast Retransmit & Fast Recovery

이제 약간 입이 지겨울 정도로 얘기해서 여기서는 대략 설명한다. ACK가 중복돼서 3번 오게 되면 해당 segment만 loss가 일어났다고 판단을 해서 timeout을 기다리지 않고 바로 retransmit을 해준다. 이 과정을 **Fast Retransmit**이라고 한다. 왜 3 번 오는것만 빠르게 처리 해주냐면 해당 segment만 drop돼서 reordering 때문이라 전체 다 보내는 것 보다는 해당 segment를 빠르게 처리해 주는 것이 좋다.

retransmit을 한 후에는 congestion이 약간 발생했다고 가정해서 일단 `ssthresh = cwnd / 2`, `cwnd = ssthresh + 3` 다음과 같이 값을 설정한다. 그리고 duplicate ACK들이 올때마다 `cwnd = cwnd + SMSS`를 해준다. 이과정을 **Fast Recovery**라고 부른다. Congestion을 피해야 하긴 하지만 다시 Slow Start까지는 아닌 정도라 이렇게 한다.

![](http://drive.google.com/uc?export=view&id=1zSo08vDmOnrYlRnLkBOV8NqT6yeA4E6R)

## Reference

[RFC 793 - Transmission Control Protocol](https://tools.ietf.org/html/rfc793#ref-3)

[Transmission Control Protocol](http://www2.ic.uff.br/~michael/kr1999/3-transport/3_05-segment.html)
