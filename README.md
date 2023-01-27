# Network Programming
네트워크 프로그래밍을 안 듣고 컴퓨터망을 듣는 호구를 위해...  
열혈 TCP/IP 소켓 프로그래밍 책을 보고 정리하는 레포지토리

# 이론 정리
## 소켓의 타입
### 연결지향형 소켓(`SOCK_STREAM`)  
- 중간에 데이터가 소멸되지 않고 목적지로 전송된다.
  - 독립된 연결을 통해 데이터를 전달하기에, 연결 자체가 문제가 있는게 아닌 이상 데이터가 소멸되지 않음을 보장한다.
- 전송 순서대로 데이터가 수신된다.  
- 전송되는 데이터의 경계(Boundary)가 존재하지 않는다.  
  - 데이터를 송수신하는 소켓은 내부적으로 버퍼를 가지고 있다. 이 버퍼를 사용하여 데이터를 나눠서 받을수도 혹은 한번에 받을 수도 있다. 그렇기에 데이터의 경계가 없다고 한다.
- 소켓 대 소켓의 연결은 반드시 1대1이어야 한다.
- 신뢰성 있는 순차적인 바이트 기반의 연결지향 데이터 전송 방식의 소켓
- TCP 프로토콜이 이에 해당한다.

### 비 연결지향형 소켓(`SOCK_DGRAM`)
- 전송된 순서에 상관없이 가장 빠른 전송을 지향한다.
- 전송된 데이터는 손실의 우려가 있고, 파손의 우려가 있다.
- 전송되는 데이터의 경계(Boundary)가 존재한다.
- 한번에 전송할 수 있는 데이터의 크기가 제한된다.
- 신뢰성과 순차적 데이터 전송을 보장하지 않는, 고속의 데이터 전송을 목적으로 하는 소켓
- UDP 프로토콜이 이에 해당한다.

## 인터넷 주소(Internet Address)
IP(Internet Protocol) 주소 체계는 IPv4와 IPv6 두 종류로 나뉜다. 두 체계의 차이는 IP주소 표현에 사용되는 바이트 크기이며, 자세한 설명은 생략.  

IPv4 기준의 4바이트 IP 주소는 네트워크 주소와 호스트(컴퓨터를 의미) 주소로 나뉘며, 주소의 형태에 따라 A, B, C, D, E 클래스로 분류된다.  

- A 클래스 : 앞 1바이트가 네트워크 주소, 뒤 3바이트가 호스트 주소
- B 클래스 : 앞 2바이트가 네트워크 주소, 뒤 2바이트가 호스트 주소
- C 클래스 : 앞 3바이트가 네트워크 주소, 뒤 1바이트가 호스트 주소
- D 클래스 : 멀티캐스트 IP 주소
- E 클래스 : 예약된 IP 주소

네트워크 주소와 호스트 주소로 나뉘어져 있는 이유는 IP 주소로 컴퓨터를 찾을 때, 한번에 바로 접근하는 방식이 아니기 때문이다. 네트워크 주소를 사용하여 특정 네트워크에 접근한 뒤에 호스트 주소를 사용하여 특정 호스트로 접근하는 방식을 사용한다. 예를 들어, SEMI.COM라는 회사의 무대리에게 데이터를 전송한다고 가정해보자. 그러면 우선 네트워크 주소를 사용하여 SEMI.COM의 네트워크로 데이터가 전송되고, 이후 해당 네트워크를 구성하는 라우터가 호스트 주소를 사용하여 무대리에게 데이터를 전송한다.  

> 송신자 ---(네트워크 주소)---> 네트워크(라우터, 스위치) ---(호스트 주소)---> 호스트  

위 흐름에서 알 수 있듯 특정 IP 주소에서 네트워크 주소와 호스트 주소를 아는 것은 중요하다. 이것은 IP 주소의 첫 번째 바이트를 보면 알 수 있다. 왜냐하면 다음과 같이 클래스 별로 IP 주소의 경계를 나눠놓았기 때문이다.

- 클래스 A의 첫 번째 바이트 범위 : 0이상 127이하 
- 클래스 B의 첫 번째 바이트 범위 : 128이상 191이하 
- 클래스 C의 첫 번째 바이트 범위 : 192이상 223이하 

이는 다음과 같이 표현할 수도 있다.

- 클래스 A의 첫 번째 비트는 항상 `0`으로 시작
- 클래스 B의 첫 번째 비트는 항상 `10`으로 시작
- 클래스 C의 첫 번째 비트는 항상 `110`으로 시작

## PORT 번호
IP 주소가 네트워크 상의 컴퓨터를 구분하기 위한 목적으로 사용된다면, PORT 번호는 컴퓨터 내에서 프로세스를 구분하기 위한 목적으로 사용된다.  

PORT 번호는 16비트로 표현되기에, 할당할 수 있는 PORT 번호의 범위는 0이상 65535이하이다. 하지만 0부터 1023까지는 '잘 알려진 PORT(Well-known PORT)'라서, 사용처가 이미 정해져있다. 추가로 PORT 번호는 기본적으로 중복이 불가능하지만, TCP 소켓과 UDP 소켓은 PORT 번호를 공유하지 않아 중복되어도 상관없다.  

## 바이트 순서(Order)와 네트워크 바이트 순서
CPU가 데이터를 저장하는/해석하는 방식은 두 가지로 나뉜다.  

- 빅 엔디안(Big Endian) : 상위 바이트의 값을 작은 번지수에 저장하는 방식
- 리틀 엔디안(Little Endian) : 상위 바이트의 값을 큰 번지수에 저장하는 방식  

예를 들어, 0x20번지를 시작으로 4바이트 Int형 정수 0x12345678을 저장한다고 가정하면, 다음과 같이 저장된다.  

| | 0x20번지 | 0x21번지 | 0x22번지 | 0x23번지
|-----|-----|-----|-----|-----|
| 빅 엔디안 | 0x12 | 0x34 | 0x56 | 0x78
| 리틀 엔디안 | 0x78 | 0x56 | 0x34 | 0x12

데이터의 저장 방식은 CPU에 따라 달라지며, 이러한 CPU의 데이터 저장방식을 '호스트 바이트 순서(Host Byte Order)'라고 한다. 만약 호스트 바이트 순서가 서로 다른 컴퓨터끼리 데이터를 그냥 주고받는다면, 0x1234라는 데이터를 보냈을 때 상대방은 0x3412라는 값으로 받아들일 것이다. 이런 문제를 해결하기 위해 네트워크를 통해 데이터를 전송할 때는 통일된 기준으로 데이터를 전송하기로 정하게 되었다. 이를 '네트워크 바이트 순서(Network Byte Order)'라고 하며, 빅 엔디안 방식으로 통일하기로 했다. 

즉, 네트워크를 통해 수신된 데이터는 빅 엔디안 방식으로 정렬되어 있음이 보장되고, 리틀 엔디안 시스템에서 데이터를 전송할 때는 빅 엔디안 방식으로 데이터를 재정렬할 필요가 있다.  

C언어의 소켓 프로그래밍에서는 네트워크 바이트 순서로 데이터를 재정렬하는 함수를 제공한다. `htons`, `ntohs`, `htonl`, `ntohl`이 그것이다. 이 함수들에서 `h`는 호스트 바이트 순서를 의미하고, `n`은 네트워크 바이트 순서를 의미한다. `s`는 short, `l`은 long(4바이트)을 의미한다. 즉, `htons`는 "short형 데이터를 호스트 바이트 순서에서 네트워크 바이트 순서로 변환하는 함수"이다. 일반적으로 뒤에 `s`가 붙으면 PORT 번호의 변환을, `l`이 붙으면 IP주소의 변환에 사용한다. (타입이 같으니)  

추가적으로 이러한 변환은 소켓 프로그래밍에서 `sockaddr_in` 변수에 개발자가 직접 값을 채워줄 때만 해주면 된다. 데이터 송수신 과정에서는 함수 내부에서 알아서 데이터 재정렬을 해준다.  

## TCP/IP 4계층  
TCP/IP 프로토콜의 과정을 4개의 계층으로 나누어 표현한 것. 이렇게 구체적으로 프로토콜을 계층화함으로써 얻게 되는 장점은 여럿 있다.

간단하게는 프로토콜 설계의 용이성을 얻는 것이고, 책에서 말하는 더 중요한 이유는 표준화 작업을 통한 '개방형 시스템(Open System)'의 설계이다. 개방형 시스템이란 공인된 표준을 사용하여 어떠한 개방형 시스템과도 상호 연동할 수 있는 시스템이다. 개방형 시스템의 장점으로는 공인된 표준에 맞추어 제작되기에 어떠한 구현체를 사용하더라도 문제가 없다는 것이다. 예를들어 IP 계층을 담당하는 라우터는 어떤 회사의 장비를 사용하더라도 어렵지 않게 교체가 가능하다. 모든 라우터 제조사들이 IP 계층의 표준에 맞추어 라우터를 제작하기 때문이다. 즉, 이렇게 표준이 있다면, 이 표준에 맞는 다양한 기술이 빠르게 발전할 수 있고 이를 사용하는데도 문제가 없어진다.  

TCP/IP 4계층은 네트워크 엑세스 계층, 인터넷 계층, 전송 계층, 응용 계층으로 이루어져있다.  

### 네트워크 엑세스 계층
**네트워크 엑세스 계층(Network Access Layer)**은 물리적인 데이터의 전송을 담당하는 계층이다. LAN, WAN, MAN과 같은 네트워크 표준과 관련된 프로토콜을 정의한다. 두 호스트가 인터넷을 통해 데이터를 주고받으려면 물리적인 연결이 존재해야 하는데, 이 부분에 대한 표준을 해당 계층이 담당하고 있다.

OSI 7계층의 물리 계층, 데이터 링크 계층을 포함한다. 책에서는 LINK 계층이라 설명한다.

### 인터넷 계층
**인터넷 계층(Internet Layer)**은 네트워크 상에서 데이터(패킷)의 전송을 담당하는 계층으로, 서로 다른 네트워크 간의 통신을 가능하게 하는 역할을 수행한다. 인터넷을 통해 목적지로 데이터를 전송하기 위해서 어떤 경로를 거쳐갈 것인지를 해결하는 것이 인터넷 계층이다. 이 계층에서 사용하는 프로토콜이 IP(Internet Protocol)이다.  

IP 자체는 비 연결지향적이며 신뢰할 수 없는 프로토콜이다. 데이터를 전송할 때마다 거쳐야 할 경로를 선택해주지만, 그 경로는 일정치 않다. 특히 데이터 전송 도중에 경로상에 문제가 발생하면 다른 경로를 선택하는데, 이 과정에서 데이터가 손실되거나 오류가 발생할 수도 있고 이를 해결해주지 않는다. 즉, IP는 오류발생에 대한 대비가 되어있지 않은 프로토콜이다.

OSI 7계층의 네트워크 계층에 해당한다. 책에서는 IP 계층이라 설명한다.  

### 전송 계층
**전송 계층(Transport Layer)**은 IP 계층에서 알려준 경로 정보를 바탕으로 데이터의 실제 송수신을 담당한다. 해당 계층에는 TCP, UDP와 같은 프로토콜이 있다. TCP에 대해 추가로 설명하자면, TCP는 신뢰성 있는 데이터의 전송을 담당한다. 

TCP가 데이터를 보낼 때 기반이 되는 프로토콜이 IP이다. 그런데 IP는 오직 하나의 데이터 패킷이 전송되는 과정에만 중심을 두고 설계되었다. 따라서 여러 개의 데이터 패킷을 전송한다 하더라도 각각의 패킷이 전송되는 과정은 IP에 의해서 진행되므로 전송의 순서는 물론이고 전송 그 자체를 신뢰할 수 없다. 여기서 TCP가 이러한 데이터 전송 및 흐름에 있어 **신뢰성 보장**을 담당한다. 결론적으로 말하면 IP의 상위 계층에서 호스트 대 호스트의 데이터 송수신에 신뢰성을 부여하는 역할을 한다.  

OSI 7계층의 전송 계층에 해당한다.

### 어플리케이션 계층
**어플리케이션 계층(Application Layer)**은 응용 계층이라고도 하며, 프로그램의 성격에 따라 클라이언트와 서버간의 데이터 송수신에 대한 프로토콜을 담당한다. 우리가 웹프로그래밍을 하면서 흔히 접하는 여러 서버나 클라이언트 관련 응용 프로그램들이 동작하는 계층이다. 대부분의 네트워크 프로그래밍은 어플리케이션 프로토콜의 설계 및 구현이 상당부분을 차지한다.  

어플리케이션 계층 아래의 있는 다른 계층들은 우리가 네트워크 프로그래밍에서 소켓을 생성하면 데이터 송수신 과정에서 자동으로 처리된다. 데이터의 전송경로를 확인하는 과정이나 데이터 수신에 대한 응답의 과정 등이 소켓에 감추어져 있는 것이다. 즉, 우리는 네트워크 프로그램을 작성할 때 이런 영역에 대해 신경쓰지 않고 어플리케이션 계층의 코드만 작성함으로써 필요에 맞는 네트워크 프로그램을 작성할 수 있다.

## TCP
### 3-Way Handshaking 
TCP는 연결지향 프로토콜이고, 연결을 만들 때 정확한 전송을 보장하기 위해 상대방 컴퓨터와 사전에 세션을 수립하는 과정을 수행한다. 이것을 3-Way Handshaking이라고 한다.  

``` text
[SYN] SEQ: 1000, ACK: -
```
호스트 A가 B에게 연결요청을 한다고 가정하자. A는 B에게 위와 같은 형태의 SYN 메시지를 보낸다. SYN은 Synchronization의 약자로, 데이터 송수신에 앞서 전송되는 '동기화 메시지'라는 의미를 담고 있다. SEQ는 Sequence Number로 메시지의 순서를 나타내는 번호이며, 호스트마다 다른 값을 가질 수 있다. 

``` text
[SYN+ACK] SEQ: 2000, ACK: 1001
```
호스트 B는 A로부터 받은 SYN 메시지의 SEQ 값에 1을 더한 값을 ACK에 담고, 자신의 SEQ 역시 포함한 메시지를 A에게 보내준다. ACK는 Acknowledgment의 약자이며, 이러한 형태의 메시지를 SYN+ACK라고 표현한다.  

``` text
[ACK] SEQ: 1001, ACK: 2001
```
호스트 A는 다시 호스트 B로부터 받은 메시지에서 ACK값을 확인해본다. 처음 보냈던 SYN 메시지의 SEQ 값은 1000이고, 돌려받은 메시지의 ACK 값이 1001이므로 통신이 정상적으로 오고감을 확인할 수 있다. 그러면 이번에는 SYN+ACK의 SEQ 값에 1을 더한 값을 ACK에 담아서 다시 B에게 보내준다. 또한, 이때 SEQ 값은 이전에 보낸 메시지의 SEQ에 1을 더한 값을 보내준다.  

위 예시에서 알 수 있듯이, ACK로 보내는 값은 상대방이 다음번 메시지를 보낼 때의 SEQ 값이다. 이렇게 SEQ와 ACK를 주고받으며, 서로간 메시지를 정상적으로 주고받는지 검증한다. 

### 데이터 송수신
SEQ와 ACK는 실제 데이터를 주고받을 때도 데이터가 정상적으로 송수신되었는지 확인하기 위해 사용된다.  

예시를 통해 설명해보자. 호스트 A의 현재 SEQ가 1200이고 보내야 할 데이터가 200바이트일때, 데이터를 100바이트씩 나누어 보내는 상황이라고 가정하자.  
우선 호스트 A는 상대방에게 `SEQ 1200, 100바이트의 데이터`가 담긴 메시지를 보낸다. 상대방은 이 메시지를 정상적으로 받았다면, `ACK 1301`이 담긴 메시지를 보낸다. 이 값이 나오게 된 이유는 `ACK = SEQ + 전송된 바이트 크기 + 1`이라는 공식에 의해서이다. 메시지를 온전하게 전부 받았는지 확인하기 위해서 전송된 바이트 크기를 더해주는 것이고, 1을 더해주는 이유는 Handshaking의 경우처럼 다음 번에 전달된 SEQ 번호를 알리기 위함이다.  
다음으로 넘어가 상대방이 보낸 `ACK 1301` 메시지를 받은 호스트 A는 `SEQ 1301, 100바이트의 데이터`가 담긴 메시지를 다시 상대방에게 보낸다. 이후, 상대방은 `ACK 1402` 메시지를 보낼 것이고, 호스트 A가 이를 받으면 정상적인 데이터 송수신이 끝이 난다.

그렇다면 데이터가 손실된 경우 어떻게 동작할까? 우선 호스트 A가 메시지를 보냈는데 중간에 문제가 발생하여 상대방이 메시지를 받지 못했다고 가정하자. 그러면 호스트 A는 일정시간이 지나도 보낸 메시지에 대한 ACK 메시지를 받지 못할 것이고, 호스트 A는 재전송을 진행한다. 이렇듯 데이터의 손실에 대한 재전송을 위해, TCP 소켓은 ACK 응답을 요구하는 패킷 전송 시에 타이머를 동작시킨다. 그리고 이 타이머가 Timeout되었을 때 패킷을 재전송한다.  

### 4-Way Handshaking
TCP 소켓은 연결지향 프로토콜이고, 또한 데이터가 모두 전송되어야하는 신뢰성이 중요하다. 그렇기에 소켓을 끊을 때도 별도의 과정이 필요하다. 그냥 연결을 바로 끊어버릴 경우, 상대방이 전송할 데이터가 남아있을 때 문제가 되기 때문이다. 이 연결을 끊는 과정에는 4번의 통신이 필요하기에 4-Way Handshaking이라고 한다.  

4-Way Handshaking의 진행과정은 우선 연결 종료를 하고 싶은 호스트 A가 먼저 `FIN`이 포함된 메시지를 호스트 B에게 보낸다. 이 메시지를 받은 호스트 B는 다시 ACK 메시지를 보내준다. 이후 호스트 B가 전송해야할 메시지를 모두 전송한 경우, 호스트 A에게 `FIN` 메시지를 보내준다. 이 메시지를 받은 호스트 A는 호스트 B에게 ACK 메시지를 보내주고 연결이 종료된다.  

호스트 B가 ACK 메시지를 보내고 FIN 메시지를 다시 보내는 이유는 위 설명에서 유추가능하다. 먼저 보낸 ACK 메시지는 호스트 A에게 FIN 메시지를 잘 받았다는 의미로 보내주는 것이다. 그리고 만약 보내야할 메시지가 아직 남아있는 경우, 이 메시지들을 먼저 전송하기 위해 FIN 메시지를 추후에 보내는 것이다.  

마지막 추가로 알아두면 좋은 것이 호스트 B가 FIN 메시지를 보내기 전에 전송한 패킷이 어떠한 이유로 인해 FIN 메시지보다 늦게 도착하는 상황이 발생한다면 어떻게 될까? 호스트 A가 FIN을 받자마자 세션을 종료시킨다면, 뒤늦게 도착하는 메시지가 소멸되고 데이터는 유실된다. TCP는 이러한 현상을 막기 위해 호스트 A는 FIN을 수신하더라도 일정시간동안 세션을 남겨놓고 잉여 메시지를 기다리는 과정을 거치게 되는데 이 과정을 "TIME_WAIT"라고 한다. 일정시간이 지나면, 세션을 만료하고 연결을 종료시키며, "CLOSE" 상태로 바뀌며 완전히 연결이 끝이 난다.

## UDP
UDP는 전송 계층의 프로토콜 중 하나이다. 신뢰성 있는 데이터의 송수신을 위해 '흐름 제어(Flow Control)'을 하는 TCP와 달리 UDP는 흐름 제어를 하지 않는다. 그렇기 때문에 일반적으로 UDP가 TCP보다 빠르다. 그 이유는 TCP가 데이터 송수신 이전, 이후에 거치는 핸드쉐이킹 과정과 데이터 송수신 과정에서 거치는 흐름제어 때문이다. 이 말은 다르게 말하면 이 과정을 최대한 줄일 수 있으면, TCP도 UDP 못지않은 속도를 낼 수 있다는 뜻이다. 한번에 보내는 데이터의 크기를 크게해서 보낸다거나 하는 식으로 말이다.  

UDP는 흐름 제어를 하지 않는데, 그렇다면 UDP의 역할은 어디까지일까? 우선 호스트 A에서 전송된 UDP 패킷이 호스트 B에게 전달되도록 하는 것은 IP(인터넷 계층)의 역할이다. 이렇게 전달된 UDP 패킷을 호스트 B에 존재하는 UDP 소켓 중 하나에게 최종 전달하는 것이 UDP의 역할이다. 즉, UDP의 역할 중 가장 중요한 것은 호스트로 전달된 패킷을 PORT 정보를 참조하여 최종 목적지인 UDP 소켓에 전달하는 것이다.  

UDP는 흐름 제어를 하지 않기에 데이터의 신뢰성은 보장할 수 없지만, 그 성능은 빠르다. (물론 최근 네트워크 기술들의 발전으로 UDP도 데이터가 손실되는 경우가 거의 없다.) 즉, 압축파일을 전송하는 것과 같이 데이터의 일부만 손상되도 문제가 발생하는 경우, 반드시 TCP를 기반으로 송수신해야한다. 그러나 인터넷으로 실시간 영상 및 음성을 스트리밍하는 경우 데이터의 일부가 손상되더라도 큰 문제가 되지 않는다. 하지만 속도는 중요한 요소인데, 실시간으로 멀티미디어 데이터를 받아야하기에 속도가 빨라야한다. 즉, 이러한 경우가 UDP를 기반으로 통신하기에 좋은 상황일 수 있다. 일반적으로 스트리밍 데이터는 조금씩 나누어 작게 자주 보내기 때문에, TCP로 이를 구현할 경우 UDP에 비해 느릴 가능성이 크다.  

# C 네트워크 프로그래밍  
## TCP 소켓
TCP 소켓을 통해 `write` 함수를 호출할 경우 데이터를 출력버퍼로 넣고, `read` 함수를 호출할 경우 입력버퍼로부터 저장된 데이터를 읽어들인다. 즉, 실제 데이터가 전송되거나 수신되는 시점은 `write`, `read` 함수를 호출할 때가 아니다. `write` 함수를 호출하면 출력버퍼에 데이터가 전달되고 상황에 맞게 적절히 데이터를 상대방의 입력버퍼로 전송한다. 그러면 상대방은 `read` 함수를 호출해서 입력버퍼에 저장된 데이터를 읽게되는 것이다.  

입출력 버퍼의 특성은 다음과 같다.  

- 입출력 버퍼는 TCP 소켓 각각에 대해 별도로 존재한다.
- 입출력 버퍼는 소켓 생성시 자동으로 생성된다.
- 소켓을 닫아도 출력버퍼에 남아있는 데이터는 계속해서 전송이 이루어진다.
- 소켓을 닫으면 입력버퍼에 남아있는 데이터는 소멸된다.

데이터가 입출력 버퍼를 통해 전달된다는 사실을 알았다. 그렇다면 클라이언트의 입력버퍼 크기보다 큰 데이터를 전송할 경우 어떻게 될까? TCP는 데이터의 흐름을 컨트롤하고, 입력버퍼의 크기를 초과하는 분량의 데이터 전송은 하지 않는다. TCP는 '슬라이딩 윈도우(Sliding Window)'라는 프로토콜을 사용하여 클라이언트의 입력 버퍼에 수용할 수 있을 정도로만 데이터를 전송해간다. 예를들어 클라이언트의 입력 버퍼가 50바이트이고, 보내야할 데이터가 100바이트라고 가정하자. 그렇다면 가장 처음 50바이트의 데이터가 전송되고, 이후 입력 버퍼가 20바이트만큼 비었다면 그 다음 20바이트를 보내주는 방식으로 동작한다.  

### Half-close
`close` 함수의 호출을 완전 종료를 의미한다. 완전 종료라는 것은 데이터를 전송하는 것 뿐만 아니라 수신하는 것 조차 더 이상 불가능한 상황을 의미한다. 때문에 한쪽에서의 일방적인 `close` 호출은 경우에 따라 위험할 수 있다. 상대방이 보낸 데이터가 전송되고 있을 때 `close` 함수가 호출되면, 해당 호스트는 이 데이터를 받을 수 없다. 이런 문제를 해결하기 위해 데이터의 송수신에 사용되는 스트림의 일부만 종료(Half-close)하는 방법이 제공된다.  

소켓을 통해 두 호스트가 연결되면, 상호간 데이터의 송수신이 가능한 '스트림이 형성된 상태'가 된다. 이 스트림은 한 방향으로만 데이터를 전송할 수 있기에, 양방향 통신을 위해서는 두 개의 스트림이 필요하다. 즉, 각 호스트 별로 입력 스트림과 출력 스트림이 형성되는 것이다. 기존 `close` 함수 호출은 한번에 이 두 스트림을 모두 끊어버리지만, Half-close 방식은 이 중 하나의 스트림만 끊게 된다.

## UDP 소켓  
UDP 서버, 클라이언트는 비 연결지향형이기에 TCP와 달리 연결된 상태로 데이터를 송수신하지 않는다. 때문에 TCP와 달리 연결 설정 과정이 필요없고, UDP 소켓의 생성과 데이터 송수신 과정만 존재한다.  

또한 TCP에서는 소켓과 소켓의 관계가 1대1이었던 것과 달리 UDP에서는 서버건 클라이언트건 하나의 소켓만 있으면 된다. 또한 UDP 소켓 하나만 있다면, 여러 호스트를 대상으로 데이터의 송수신이 가능하다.  

TCP 소켓은 목적지에 해당하는 소켓과 연결된 상태이기 때문에, 주소 정보를 따로 추가할 필요가 없었다. 그러나 UDP는 연결상태를 유지하지 않으므로, 데이터를 전송할 때마다 반드시 목적지의 주소정보를 별도로 추가해야 한다.  

또한, UDP는 데이터의 경계가 있다. 이 말은 상대방이 데이터를 3번 보내면, 나도 3번의 거쳐 받아야하는 것이다. TCP의 경우 입력 버퍼에 쌓아놨다가 한번에 받을 수 있는 것과는 차이가 있다. 비슷한 예시로 상대방이 데이터를 한번에 보내면, 이를 한번에 받아야하지 여러 번 나누어 받을 수 없다.

### Connected UDP 소켓
기본적으로 UDP 소켓은 목적지 정보가 등록되어 있지 않은 unconnected 소켓이다. 매번 메시지 전송을 할 때마다 UDP 소켓에 목적지의 IP, PORT를 등록하고, 데이터를 전송한 다음, 소켓에 등록된 목적지 정보를 삭제하는 과정을 거친다. 그러나 같은 호스트와 반복해서 메시지를 주고받아야 하는 경우, 매번 목적지 정보를 등록하고 삭제하는 과정이 불필요하다. 이럴 때 UDP 소켓을 목적지 정보가 등록된 connected 소켓으로 만드는 것이 효율적일 것이다.  

C 시스템 프로그래밍에서 `connect` 함수를 호출해서 Connected UDP 소켓을 만들 수 있다. TCP 소켓에서는 해당 함수가 연결을 만들었지만, UDP 소켓에서는 단순히 목적지를 등록하는 것에서 끝난다.