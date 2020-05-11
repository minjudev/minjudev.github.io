## 1. TCP/IP
### 1) TCP/IP 계층 모델
- 우리가 범용적으로 사용하는 TCP 프로토콜과 IP 프로토콜을 OSI 7계층 형식에 맞추어 더 추상화시킨 모델
- 기존의 OSI 참조 모델을 4가지 계층으로 줄인 프로토콜

#### (1) Network Layer
- 시스템 간 물리적 연결과 상태를 관리
- OSI 참조 모델의 1 ~ 2계층에 해당
- PDU(Protocol Data Unit): Frame

#### (2) Internet Layer
- 주소를 이용한 경로 설정 및 전송 담당
- OSI 참조 모델의 3계층에 해당
- PDU(Protocol Data Unit): IP Datagram

#### (3) Transport Layer
- 통신 간 신뢰성 있는 연결을 보장
- OSI 참조 모델의 4계층에 해당
- PDU(Protocol Data Unit): Segment

#### (4) Application Layer
- 응용 프로그램에서 HTTP, FTP, SMTP 등의 프로토콜을 담당
- OSI 참조 모델의 5 ~ 7계층에 해당
- PDU(Protocol Data Unit): Data

### 2) Ethernet
#### (1) Ethernet의 정의
- 컴퓨터 네트워크 기술의 하나로, 일반적으로 LAN, MAN, WAN에서 가장 많이 활용되는 기술 규격
- OSI 모델의 물리 계층에서 신호와 배선, 데이터 링크 계층에서 MAC 패킷과 프로토콜의 형식 정의
- 이더넷 기술은 대부분 IEEE 802.3 규약으로 표준화됨
- 네트워크 방식에 맞춰서 네트워크 장비들을 구입해야 함
- 현재 가장 널리 사용되고 있으며 Token ring, FDDI(Fiber Distributed Data Interface) 등의 다른 표준을 대부분 대체함

#### (2) CSMA/CD(Carrier Sense Multiple Access with Collision Detection) 
- Ethernet이 Frame을 전송하는 방식은 full-duplex와 half-duplex에 따라 다름
- full duplex로 동작하는 링크는 Frame의 송수신이 서로 다른 채널을 통해 이루어지므로 충돌 발생 X, 그러므로 충돌 감지도 X)
- CSMA/CD는 half-duplex로 동작하는 링크에서, Ethernet이 Frame을 전송하는 방식
  - 1. Carrier Sense(네트워크 신호 감지): 호스트가 Frame을 전송하기 전에 네트워크 상에 다른 Frame이 전송되는지 확인
  - 2. Multiple Access(다중 접근): Ethernet에 연결된 장비들은 네트워크 상에 Frame의 흐름이 없을 때, 서로 동시에 Frame 전송 가능
  - 3. Collision Detection(충돌 감지): Ethernet은 복수의 장비가 동시에 Frame을 전송하기 때문에 충돌 가능성이 있으므로, 전송 후 충돌 발생 여부를 확인

### 3) Ethernet 프로토콜 구조

### 4) TCP/IP 네트워크 접속 계층





















