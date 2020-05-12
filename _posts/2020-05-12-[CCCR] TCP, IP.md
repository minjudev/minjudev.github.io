## 1. TCP/IP
### 1) TCP/IP 계층 모델
- TCP: Transmission Control Protocol
- IP: Internet Protocol
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

### 3) Ethernet Frame 구조
Data link 계층은 상위 계층의 패킷을 전달받아 프레임 형태로 변환하여 Physical 계층에 전달한다.   
이때, 이더넷 헤더에는 상위 계층의 프로토콜 정보와 출발지 주소, 도착지 주소에 대한 정보가 담긴다.  
<img width="600" alt="Ethernet V2 구조" src="https://user-images.githubusercontent.com/53208493/81695716-f33f2700-949d-11ea-8e95-07839995c00a.PNG">
> [이미지 출처: Ethernet Frame의 기능과 구조](https://mr-zero.tistory.com/38)

- Preamble
  - Physical 계층에서 전송된 비트 패턴으로 송신자와 수신자의 동기를 맞추는 데 사용됨
  - Frame 전송의 시작을 나타내는 필드, 10101010이 반복되는 7byte 길이의 필드
  - 수신 측에 Frame이 전송된다는 것을 알리고, 0과 1을 제대로 구분할 수 있게 Synchronization(동기) 신호를 제공하는 역할
  - 패킷 캡쳐 범위 밖
  
- SOF(Start Of Frame)/SFD(Starting Frame Delimiter)
  - 물리 계층이 끝나고 이제부터는 Ethernet 헤더라고 알려주는 역할을 함
  - 1byte 크기로 항상 10101011로 설정되며 Frame의 시작을 알리는 데 사용됨
  - 패킷 캡쳐 범위 밖
  
- 목적지 주소(Destination MAC address)
  - 프레임을 수신할 MAC 주소 정보가 담겨있음
  - 6byte
  
- 출발지 주소(Source MAC address)
  - 프레임을 송신한 호스트의 MAC 주소 정보가 담겨있음
  - 6byte
  
- 타입 or 길이
  - 상위 계층 프로토콜 정보를 담고 있음
  - 데이터 필드의 길이나 MAC 클라이언트 프로토콜의 종류를 표시   
  (1500 이하 - 데이터 필드 길이 표시/ 1500 이상 - MAC 클라이언트 프로토콜의 종류 표시)
  - 2byte

- 데이터
  - 상위 계층으로부터 전달받은 캡슐화된 데이터(packet)가 담김
  - 최소 46byte, 최대 1500byte의 크기를 가짐

- FCS(Frame Check Sequence)
  - 프레임의 오류를 확인(오류 검출용 필드)
  - 4byte
  - 패킷 캡쳐 범위 밖

### 4) TCP/IP 네트워크 접속 계층
- Cable, 송수신, 네트워크 보드, 링크 Protocol, LAN 접속 Protocol에 대한 연결 구성요소들을 나타냄   
  물리적 링크를 통해 실제 데이터를 운반하는 기능 제공   
  물리적 주소, 네트워크 Topology, Flow Control, Error Control 기능 제공   
  사용 가능 장비: L2 Switch, Bridge   

- 2계층 Data-link Layer   
  - LLC(Logical Link Control)   
    - 두 장비 간에 링크를 설정하고, 프레임을 송수신하는 방식과 상위 계층 프로토콜의 종류를 알리는 역할
    - 모든 LAN은 802 표준에 똑같은 LLC 계층을 가짐
    - LLC 계층은 데이터 링크 계층에서 전달하는 전기적인 신호의 흐름을 제어할 목적으로 사용됨
    - 논리적 연결을 사용하여 상위 계층 간을 연결

  - MAC(Media Access Control)
    - 어떻게 프레임을 전송할 것인지를 정의
    - 물리적 매개체를 통해 데이터를 어떻게 보낼 것인지를 책임지고 있는 계층
    - 물리적 주소, 네트워크 Topology, 흐름 제어, 에러 통제 등을 담당

### 5) 주소 결정 Protocol
DNS(Domain Name System)는 FQDN(Fully Qualified Domain Name)을 IP 주소로 변환하기 위해 사용한다.   
또한, Ethernet과 같은 Broadcasting 전송이 가능한 네트워크에서는 실제 데이터 전송을 위해 반드시 수신 시스템의 물리적 주소(MAC)를 알아야 한다.   
그러므로 송신 시스템은 수신 시스템에게 데이터를 전송하기 전에 MAC 주소를 찾아야 한다.   
이때 ARP Protocol은 해당 IP address를 이용해 MAC address를 찾아주는 대표적인 Protocol이다.   

#### (1) ARP(Address Resolution Protocol)
- 논리적인 IP 주소를 기반으로 데이터 링크 계층의 물리적인 MAC 주소로 바꾸어 주는 주소해석 프로토콜
- 변환 단계
  - ARP Request: Broadcast Frame으로 MAC address 찾기
  - ARP Reply: Unicast Frame으로 IP에 해당하는 MAC address 전송
- PC 내에서 ARP로 알아온 IP/MAC 주소 정보는 arp cache table에 저장

#### (2) RARP
- ARP의 반대 개념
- 물리적인 MAC 주소를 논리적인 IP 주소로 변환시켜주는 프로토콜(RAM)

## 2. IP
### 1) IP 주소 체계
- http://www.iana.org
- 사용 가능한 IP 주소 총 개수
  - IP 주소 X.X.X.X 에서 각 X 부분에는 숫자 0 ~ 255까지 올 수 있음
  - 총 IP 주소 개수: 256 * 256 * 256 * 256 = 4,294,967,296(약 43억 개)
  - 많은 수의 IP 주소를 체계 없이 사용한다면 혼란이 올 수 있으므로
  **클래스 주소 체계의 필요성**을 느낌
- IP 주소 클래스
  - 각 클래스는 네트워크 구분자와 호스트 구분자로 구성
  - 네트워크 구분자: 처음 비트를 구분자로 사용
    - 클래스 A: 8비트, 클래스 B: 16비트, 클래스 C: 24비트
  - 호스트 구분자: 네트워크 구분자로 할당되지 않은 나머지
    - 각 네트워크에 연결된 단말기를 구분하는 역할
- 각 네트워크에서 호스트에 할당되지 못하는 예외적인 호스트 주소
  - 처음 IP 주소: 네트워크 자체를 가리키는 네트워크 주소로 사용
  - 마지막 IP 주소: 모든 호스트를 뜻하는 브로드캐스트 주소로 사용
- 네트워크 당 최대 호스트 개수
  - 처음 및 마지막 IP 주소 2개를 제외한 총 개수   
  (ex. 클래스 A -> 2의 24제곱 - 2)
- 대륙별 IP 주소 관리 기구
  - IP 주소는 ICANN(Internet Corporation for Assigned Names and Numbers)이라는 국제기구에서 관리

### 2) Internet 주소 체계
- 인터넷 망에서 컴퓨터 간의 통신을 위한 유일한 논리적 주소를 의미
- 숫자를 이용해 32비트 주소 체계를 사용
- 주소 체계는 Network ID와 Host ID로 나누어짐

### 3) 공인 IP/ 사설 IP
#### (1) 공인 IP
- 전 세계에서 유일하게 쓸 수 있는 IP
- IPv4 주소 체계에서는 모든 기기에 할당하기에 개수가 부족
  - IPv4 총 IP 개수: 약 43억 개
  - 전 세계 인구가 약 72억 명이라고 가정 시, 한 사람당 하나의 IP 주소를 할당하기에는 IP의 총 개수가 부족
  - 컴퓨터, 태블릿 PC, 핸드폰 등 인터넷이 가능한 모든 기기에 사용되므로 문제가 된다.
  
#### (2) 사설 IP
- 의미
  - IP 주소 부족 문제를 해결하기 위해 등장
  - 한 네트워크에 속한 사용자들끼리만 인식할 수 있는 임시 IP
  - 같은 네트워크 내에서 홈페이지를 만들어 서비스 또는 파일 주고 받기 가능
  - but, 네트워크를 벗어나면 사설 IP를 그대로 사용할 수 없음
  
- 이해
  - 사설 IP주소를 사용하는 사용자 A 컴퓨터에서 외부에 있는 X 홈페이지에 접속하려면 A 컴퓨터의 IP 주소(192.168.100.3) 사용 불가능
  - 대신 회사 공인 IP 주소(126.34.78.32) 일련번호를 붙여서 사용
    - ex. 126.34.78.32:34001
    - 34001은 A 컴퓨터에 대한 고유 번호로 회사에 설치된 장비에 기억
  - 반대로 외부 네트워크에서 내부 네트워크로 들어올 땐, 해당 장비를 통해 다시 사설 IP로 변환

- 장점
  - 같은 네트워크를 사용하는 수십, 수백 명의 사람들이 하나의 공인 IP로 인터넷을 쓸 수 있음
  - NAT(Network Address Translation)
    - IP 패킷의 TCP/UDP 포트 숫자와 소스 및 목적지의 IP 주소 등을 재기록하면서 라우터를 통해 네트워크 트래픽을 주고 받는 기술
    - 패킷에 변화가 생기므로 IP나 TCP/UDP의 체크섬(checksum)도 다시 계산되어 재기록해야 함
    - 공인 IP와 사설 IP를 서로 변환하여 IP 부족 문제를 해결하고, 외부 침입에 대한 보안을 높이는 기술

### 4) 서브넷 마스크(Subnet Mask)
#### (1) 서브네팅(Subnetting)이란?
- 네트워크 관리자가 네트워크 성능을 향상시키기 위해, 자원을 효율적으로 분배하는 것
- 자원을 효율적으로 분배: 네트워크 영역과 호스트 영역을 분할
- 필요한 네트워크 주소만 호스트 IP로 할당할 수 있게 만들어 네트워크 낭비를 방지

#### (2) 서브넷 마스크
ex. 210.100.100.1이란 클래스 C의 IP 주소가 있다.
- 1. 디폴트 서브넷 마스크
  - 주어진 클래스 C를 전혀 가공하지 않고, 쪼개지 않고 그대로 사용
  - 주어진 클래스 C를 모두 쓰는 경우에도 서브넷 마스크는 따라다니는데 그게 바로 "디폴트 서브넷 마스크", 즉 "기본 서브넷 마스크"
- 2. 서브넷 마스크 
  - 주어진 네트워크를 가공해서, 쪼개서 사용하는 경우
  - 커다란 네트워크를 잘게 나누기 위해 필요
  - 주어진 IP 주소를 네트워크 환경에 맞게 나누어 주기 위해서 씌워주는 2진수 조합
  - IP 주소를 통해 어디까지가 네트워크 부분이고, 어디까지가 호스트 부분인지를 나타내는 역할    
  (네트워크 부분: 서브넷 마스크가 2진수로 '1'인 부분, 호스트 부분: 2진수로 '0'인 부분)
  
  
  

## 3. TCP/IP
### 1) TCP/IP Internet 계층
### 2) ICMP
### 3) Port number
### 4) TCP Header 구조
### 5) OP Code
### 6) UDP(User Datagram Protocol) Format












