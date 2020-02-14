## 3-way handshaking (연결을 위한)

1. Client가 Server에게 SYN(c)를 보낸다.

- c은 임의의 난수값

2. Server는 Client에게 ACK(c+1), SYN(s)을 보낸다.

- s는 임의의 난수값 

3. Clinet는 Server에게 ACK(s+1)을 보내면 연결이 성립된다. 





## 4-way handshaking

1. Client가 연결 종료 신호인 FIN을 보낸다.

2. 서버도 알겠다고 ACK를 보낸다.

3. 서버에서 데이터 보낼게 있으면 데이터 전송 

4. 데이터 전송이 끝나면 서버는 FIN을 클라이언트에게 보낸다 .

5. 서버의 FIN 신호를 확인 했다는 ACK를 클라이언트에 보내고 소켓을 종료한다.

* 세션이 끝난후 뒤늦게 도착한 패킷을 위해 일정시간 (default 240초) 세션을 열어둔다 (해당상황을 TIME_WAIT이라고 한다.)







## 공개키 암호화 방식

- 보내야할 데이터를 수신자의 공개키로 암호화를 걸고, 수신자는 자신의 비밀키(10의 308승)로 데이터를 복호화하는 방법 

- 암호화는 강력하지만 컴퓨팅이 높음 





## 대칭키 암호화 방식 

- 하나의 키를 서로 사용하는 방법 

- 컴퓨팅이 낮음 

- 보안의 위험성.



## TLS Handshaking



1. Clinet Hello (C -> S)

- Client가 지원하는 암호화 목록 (Cipher)

- Clinet nonce



2. Server Hello (S -> C)

- Client가 보낸 암호화 목록중 가장 높은 보안 수준을 제공하는 선택.

- Server Certificate(인증서)

- Server Nonce



3. C -> S

- 서버가 보낸 인증서를 CA의 Public key로 해독 하여 server의 public key를 획득 

- 클라이언트는 PMS(Pre master secret)라는 난수값을 생성하여 서버의 public key를 이용해서 암호화 해서 서버에게 전달 (공개키 암호화 방식)



4. 대칭키 생성 

- 이후 C, S는 {Client nonce, Server Nonce, PMS}를 이용해서 Encryption key , MAC key 생성


- Encryption key는 C <-> S간 암호화 통신할 때 사용할 키

- MAC Key는 Message HMAC 값에 계산할때 사용 



5 C <-> S

- 최종적으로 통신 하기전에 이전까지 서로간 통신했던 메시지의 HMAC값을 생성해서 서로에게 보내서 MAC key로 해당 값이 맞는지 확인


- 현재까지 암호화 통신이 아니었으므로 최종적으로 무결성 체크 







## DNS 조회 과정 

1. Local에 저장된 DNS Caching 정보가 있는지 확인 (ipconfig /displaydns)

2. Browser에 URL를 입력하면 hosts에서 설정 된 내역이 있는지 조회한다.

3. 로컬에 설정한 DNS서버로 질의 

4. root DNS에 질의하면, 최상위 DNS서버를 알려줌 (ex) .com dns server )

- root DNS는 세계에 총 13대 (미국 10대, 일본/네덜란드/노르웨이 각 1대)

- 한국에는 mirror 서버 3대 

5. .com 서버에 질의하면 또 다른 dns 정보를 알려줌 (ex) naver.com dns server)

6. nerver.com dns서버에 질의 하면 최종 IP를 알려줌



4~6까지 질의를 recusive query라고 함 







### TCP 

- 신뢰적인 연결 방식



### 흐름제어 

수신자의 데이터 처리하는 속도가 송신자가 보내는 속도보다 빠르면 문제 없음 

처리속도 > 보내는 속도 



처리속도 < 보내는 속도

데이터 손실이 발생 불필요한 데이터 전송이 발생됨..

이런것을 막기 위해서 'Stop and Wait' 과 'Sliding Window' 방식이 존재



Stop and Wait : 송신자가 보낸 내용에 수신자가 잘 받았다고 응답하면 다음 패킷을 전송.

Sliding Window : 수신측에서 설정한 윈도우 크기만큼 송신측에서 확인 응답없이 세그먼트를 전송할 수 있게 하여 데이터 흐름을 동적으로 조절하는 기법




*Window : 수신측에서 받을 수 있는 크기(?), TCP/IP는 2개의 window가 존재 3-way handshaking 과정에서 크기 조율.





### 혼잡제어 

단순히 송신자/수신자 처리에 집중하기 보단 전체적인 네트워크(라우터)까지 생각해서 제어함



### 혼자제어 알고리즘

처음엔 패킷을 하나씩(혹은 적정한 윈도우 크기)를 보내고 문제가 없을 경우 x2 문제가 없으면 x2  계속 보내다가 지연이 있다면 보내는 크기를 줄여서 보냄


-  slow start,  Fast Restransmit, Fast Recovery



