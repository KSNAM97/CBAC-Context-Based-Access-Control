# 📗 Chapter 1. CBAC 기본 실습

> 공인 IP 환경의 단일 라우터에서 CBAC 동작을 익히는 실습입니다.

## 예제 1) 내부→외부 TCP만 허용

```
조건:
 # 내부→외부 향하는 모든 TCP 트래픽은 허용
 # 외부→내부 향하는 모든 트래픽은 차단
```

```bash
ip access-list extended ACL
 deny ip any any
!
ip inspect name CBAC tcp
!
interface Serial , FastEthernet [Port number]
 ip access-group ACL in
 ip inspect CBAC out
!
```

```bash
# 정보 확인
show ip inspect session
```

---

## 예제 2) TCP/UDP/ICMP 허용 + TCP 로그

```
조건:
 # 내부→외부 향하는 모든 TCP, UDP, ICMP 트래픽 허용 / 외부→내부 차단
 # 단 TCP 트래픽은 Log-message 출력
```

```bash
ip access-list extended ACL
 deny ip any any
!
ip inspect name CBAC tcp audit-trail on
ip inspect name CBAC udp
ip inspect name CBAC icmp
!
interface Serial , FastEthernet [Port number]
 ip inspect CBAC out
 ip access-group ACL in
!
```

```bash
# 정보 확인
show ip inspect session
```

---

## 예제 3-1) HTTP / HTTPS / DNS 만 허용

```
조건:
 # Rx는 인터넷(http, https)과 DNS만 허용되어야 한다.
```

```bash
ip access-list extended ACL
 deny ip any any
!
ip inspect name CBAC http
ip inspect name CBAC https
ip inspect name CBAC dns
!
interface Serial , FastEthernet [Port number]
 ip access-group ACL in
 ip inspect CBAC out
!
```

---

## 예제 3-2) HTTP / HTTPS / DNS 허용 + DNS 로그

```
조건:
 # Rx는 인터넷(http, https)과 DNS만 허용되어야 한다.
 # DNS에 대해서는 Syslog message를 출력해야 한다.
```

```bash
ip access-list extended ACL
 deny ip any any
!
ip inspect name CBAC http
ip inspect name CBAC https
ip inspect name CBAC dns audit-trail on
!
interface Serial , FastEthernet [Port number]
 ip access-group ACL in
 ip inspect CBAC out
!
```

---

## 예제 4) URL 필터링 (NAVER 허용 / Google 차단)

```
조건:
 # Rx는 내부→외부 http 정보 중 NAVER로의 통신은 허용
 # Rx는 내부→외부 http 정보 중 Google로의 통신은 차단
```

```bash
ip access-list extended ACL
 deny ip any any
!
ip inspect name CBAC http urlfilter
ip urlfilter exclusive-domain permit www.naver.com
ip urlfilter exclusive-domain deny   www.google.co.kr
!
interface Serial , FastEthernet [Port number]
 ip access-group ACL in
 ip inspect CBAC out
!
```

---

# 🧪 토폴로지 기반 실습

## 실습 1) R1 — TCP/UDP/ICMP 검사

```
        13.13.2.2/24                            13.13.4.4/24
                     Serial 1/0            FastEthernet0/0
          R2--------------------------R1--------------------------R4
              (내부)                                    (외부)
```

```
조건:
 # R1은 외부→내부 향하는 모든 트래픽의 접근을 차단
 # R1은 내부→외부 향하는 TCP/UDP/ICMP 트래픽 허용
 # 단 내부에서 생성되어 입력되는 응답 트래픽은 수신되어야 함
```

```bash
# R1
ip access-list extended CBAC_ACL
 permit udp any eq 520 any eq 520
 deny ip any any
!
ip inspect name CBAC tcp
ip inspect name CBAC udp
ip inspect name CBAC icmp
!
interface fastethernet 0/0
 ip access-group CBAC_ACL in
 ip inspect CBAC out
!
```

```bash
# 정보 확인
R1# show ip inspect interfaces
Interface Configuration
 Interface FastEthernet0/0
  Inbound inspection rule is not set
  Outgoing inspection rule is CBAC
    tcp  alert is on audit-trail is off timeout 3600
    udp  alert is on audit-trail is off timeout 30
    icmp alert is on audit-trail is off timeout 10
  Inbound access list is CBAC_ACL
  Outgoing access list is not set
```

```bash
# 외부(R4) → 내부(R2) : 차단 확인
R4# ping 13.13.2.2 source 13.13.4.4              [내부 <---- 외부 : 통신 X]
U.U.U
Success rate is 0 percent (0/5)

R4# telnet 13.13.2.2 /source-interface loopback 0  [내부 <---- 외부 : 접속 X]
% Destination unreachable; gateway or host down

# 내부(R2) → 외부(R4) : 허용 확인
R2# ping 13.13.4.4 source 13.13.2.2              [통신 O]
!!!!!
Success rate is 100 percent (5/5)

R2# telnet 13.13.4.4 /source-interface loopback 0  [접속 O]
... Open

# 세션 확인
R1# show ip inspect session
Established Sessions
 Session 66F96EB0 (13.13.2.2:8)=>(13.13.4.4:0) icmp SIS_OPEN
 Session 66F96EB0 (13.13.2.2:19719)=>(13.13.4.4:23) tcp SIS_OPEN
```

---

## 실습 2) R3 — 감사(audit-trail) 추가

```
        13.13.2.2/24                            13.13.5.5/24
                     Serial 1/0            FastEthernet0/0
          R2--------------------------R3--------------------------R5
              (내부)                                    (외부)
```

```
조건:
 # R3은 외부→내부 향하는 모든 트래픽 차단
 # R3은 내부→외부 향하는 TCP/UDP/ICMP 트래픽 허용 (각각 Log-message 출력)
 # 단 내부에서 생성되어 입력되는 응답 트래픽은 수신되어야 함
```

```bash
# R3
ip access-list extended IN<---OUT
 permit udp any eq 520 any eq 520
 deny ip any any
!
ip inspect name IN--->OUT tcp  audit-trail on
ip inspect name IN--->OUT udp  audit-trail on
ip inspect name IN--->OUT icmp audit-trail on
!
interface fastethernet 0/0
 ip access-group IN<---OUT in
 ip inspect IN--->OUT out
!
```

```bash
# 정보 확인
R3# show ip inspect interfaces
Interface Configuration
 Interface FastEthernet0/0
  Inbound inspection rule is not set
  Outgoing inspection rule is IN--->OUT
    tcp  alert is on audit-trail is on timeout 3600
    udp  alert is on audit-trail is on timeout 30
    icmp alert is on audit-trail is on timeout 10
  Inbound access list is IN<---OUT
  Outgoing access list is not set
```

```bash
# 내부(R2) → 외부(R5)
R2# ping   13.13.5.5 source 13.13.2.2
R2# telnet 13.13.5.5 /source-interface loopback 0
... Open

# 감사 로그 (Audit-trail)
*Jun 22 07:57:48.935: %FW-6-SESS_AUDIT_TRAIL_START: Start icmp session: initiator (13.13.2.2:8) -- responder (13.13.5.5:0)
*Jun 22 07:57:59.555: %FW-6-SESS_AUDIT_TRAIL: Stop icmp session: initiator (13.13.2.2:8) sent 360 bytes -- responder (13.13.5.5:0) sent 360 bytes
*Jun 22 07:59:22.735: %FW-6-SESS_AUDIT_TRAIL_START: Start tcp session: initiator (13.13.2.2:39349) -- responder (13.13.5.5:23)
*Jun 22 08:01:00.823: %FW-6-SESS_AUDIT_TRAIL: Stop tcp session: initiator (13.13.2.2:39349) sent 24 bytes -- responder (13.13.5.5:23) sent 208 bytes

# 세션 확인
R3# show ip inspect session
Established Sessions
 Session 66F96EB0 (13.13.2.2:8)=>(13.13.5.5:0) icmp SIS_OPEN
 Session 66F96EB0 (13.13.2.2:39349)=>(13.13.5.5:23) tcp SIS_OPEN
```
