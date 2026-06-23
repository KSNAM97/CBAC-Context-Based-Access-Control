# 📙 Chapter 2. CBAC 실습 (사설 IP : NAT + GRE)

## 🗺️ 장비 구성

```
- R1 = GIT-A      - R5 = ISP-3
- R2 = GIT-B      - R6 = ISP-4
- R3 = ISP-1      - R7 = PC1
- R4 = ISP-2      - R8 = PC2
```

---

## EX1) 공중망 구성 (OSPF)

```
조건:
 .OSPF Process = 1, Area = 0
 .Router-ID = XX.XX.XX.XX (X = Router 번호)
 .GIT-A의 사설 네트워크를 제외한 나머지는 OSPF 포함
 .GIT-B, ISP-1~4의 모든 네트워크는 OSPF 포함
 .Interface에 할당된 SubnetMask로 확인 (point-to-point)
 .OSPF 업데이트가 필요한 Interface로만 송신 (passive-interface)
```

```bash
# GIT-A (R1)
router ospf 1
 router-id 1.1.1.1
 passive-interface default
 no passive-interface serial 1/0.10
 network 100.100.1.0  0.0.0.255 area 0
 network 121.160.10.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# GIT-B (R2)
router ospf 1
 router-id 2.2.2.2
 passive-interface default
 no passive-interface serial 1/0.20
 network 100.100.2.0  0.0.0.255 area 0
 network 121.160.20.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# ISP-1 (R3)
router ospf 1
 router-id 11.11.11.11
 passive-interface default
 no passive-interface serial 1/0.10
 no passive-interface serial 1/0.12
 network 100.100.11.0 0.0.0.255 area 0
 network 121.160.10.0 0.0.0.255 area 0
 network 121.160.12.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# ISP-2 (R4)
router ospf 1
 router-id 22.22.22.22
 passive-interface default
 no passive-interface serial 1/0.12
 no passive-interface serial 1/0.23
 network 100.100.12.0 0.0.0.255 area 0
 network 121.160.12.0 0.0.0.255 area 0
 network 121.160.23.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# ISP-3 (R5)
router ospf 1
 router-id 33.33.33.33
 passive-interface default
 no passive-interface serial 1/0.23
 no passive-interface serial 1/0.34
 network 100.100.13.0 0.0.0.255 area 0
 network 121.160.23.0 0.0.0.255 area 0
 network 121.160.34.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# ISP-4 (R6)
router ospf 1
 router-id 44.44.44.44
 passive-interface default
 no passive-interface serial 1/0.20
 no passive-interface serial 1/0.34
 network 100.100.14.0 0.0.0.255 area 0
 network 121.160.20.0 0.0.0.255 area 0
 network 121.160.34.0 0.0.0.255 area 0
!
interface loopback 0
 ip ospf network point-to-point
```

```bash
# 정보 확인
ISP-1# show ip ospf neighbor   [인접성 2개 확인]
ISP-2# show ip ospf neighbor   [인접성 2개 확인]
ISP-3# show ip ospf neighbor   [인접성 2개 확인]
ISP-4# show ip ospf neighbor   [인접성 2개 확인]

GIT-A# ping 100.100.2.2 source 100.100.1.1   # 100% 성공
GIT-B# ping 100.100.1.1 source 100.100.2.2   # 100% 성공
```

---

## EX2) NAT 구성 (PAT / overload)

### EX2-1) GIT-A

```
조건:
 .GIT-A 내부 192.168.1.0/24 → 외부 통신 시
  GIT-A의 물리 Interface IP 주소로 변환되어 통신
```

```bash
# GIT-A (R1)
access-list 1 permit 192.168.1.0 0.0.0.255
!
ip nat pool NAT 121.160.10.1 121.160.10.1 netmask 255.255.255.0
!
ip nat inside source list 1 pool NAT overload
!
interface fastethernet 0/0
 ip nat inside
interface serial 1/0.10
 ip nat outside
```

```bash
# 정보 확인
GIT-A# show ip nat translation
Pro  Inside global      Inside local     Outside local    Outside global
icmp 121.160.10.1:1   192.168.1.1:1    100.100.2.2:1    100.100.2.2:1
icmp 121.160.10.1:2   192.168.1.2:2    100.100.2.2:2    100.100.2.2:2
```

### EX2-2) GIT-B

```bash
# GIT-B (R2)
access-list 1 permit 192.168.2.0 0.0.0.255
!
ip nat pool NAT 121.160.20.2 121.160.20.2 netmask 255.255.255.0
!
ip nat inside source list 1 pool NAT overload
!
interface fastethernet 0/0
 ip nat inside
interface serial 1/0.20
 ip nat outside
```

```bash
# 정보 확인
GIT-B# show ip nat translation
Pro  Inside global      Inside local     Outside local    Outside global
icmp 121.160.20.2:1   192.168.2.1:1    100.100.1.1:1    100.100.1.1:1
icmp 121.160.20.2:2   192.168.2.2:2    100.100.1.1:2    100.100.1.1:2
```

---

## EX3) CBAC 구성

### EX3) GIT-A

```
조건:
 .TCP/UDP/ICMP 내부→외부 허용 (외부→내부 차단)
 .내부→외부 TCP, ICMP 트래픽은 Log-message 출력
 .외부→내부 차단되는 모든 트래픽은 Log-message 출력
```

```bash
# GIT-A
ip access-list extended CBAC_ACL_IN
 permit ospf host 121.160.10.11 host 224.0.0.5
 deny ip any any log-input
!
ip inspect name GIT_A_IPS tcp  audit-trail on
ip inspect name GIT_A_IPS udp
ip inspect name GIT_A_IPS icmp audit-trail on
!
interface serial 1/0.10
 ip access-group CBAC_ACL_IN in
 ip inspect GIT_A_IPS out
```

```bash
# 정보 확인
GIT-A# show ip inspect sessions
Established Sessions
 Session 65CDE5AC (192.168.1.2:8)=>(100.100.2.2:0) icmp SIS_OPEN
 Session 65CDE82C (192.168.1.1:8)=>(100.100.2.2:0) icmp SIS_OPEN
 Session 65CDE5AC (192.168.1.1:58046)=>(100.100.2.2:23) tcp SIS_OPEN

# 감사 로그
*Mar 1 00:44:31.991: %FW-6-SESS_AUDIT_TRAIL: Stop icmp session: initiator (192.168.1.1:8) sent 360 bytes -- responder (100.100.2.2:0) sent 360 bytes
*Mar 1 00:44:22.827: %FW-6-SESS_AUDIT_TRAIL: Stop tcp session: initiator (192.168.1.1:58046) sent 48 bytes -- responder (100.100.2.2:23) sent 99 bytes
```

### EX3-2) GIT-B (UDP도 audit-trail 추가)

```
조건:
 .TCP/UDP/ICMP 내부→외부 허용 (외부→내부 차단)
 .내부→외부 TCP, UDP, ICMP 트래픽 모두 Log-message 출력
 .외부→내부 차단되는 모든 트래픽 Log-message 출력
```

```bash
# GIT-B
ip access-list extended CBAC_ACL_IN
 permit ospf host 121.160.20.4 host 224.0.0.5
 deny ip any any log-input
!
ip inspect name GIT_B_IPS tcp  audit-trail on
ip inspect name GIT_B_IPS udp  audit-trail on
ip inspect name GIT_B_IPS icmp audit-trail on
!
interface serial 1/0.20
 ip access-group CBAC_ACL_IN in
 ip inspect GIT_B_IPS out
```

```bash
# 정보 확인
GIT-B# show ip inspect session
Established Sessions
 Session 66F61A28 (192.168.2.2:8)=>(100.100.1.1:0) icmp SIS_OPEN
 Session 66F61FA8 (192.168.2.1:45899)=>(100.100.1.1:23) tcp SIS_OPEN
 Session 66F61CE8 (192.168.2.1:8)=>(100.100.1.1:0) icmp SIS_OPEN

# 차단 로그 (외부 PC1 → 내부)
*Mar 1 00:29:02.479: %SEC-6-IPACCESSLOGDP: list CBAC_ACL_IN denied icmp 121.160.10.1 (Serial1/0.20) -> 100.100.2.2 (8/0), 1 packet
*Mar 1 00:29:07.391: %SEC-6-IPACCESSLOGP:  list CBAC_ACL_IN denied tcp  121.160.10.1(54244) (Serial1/0.20) -> 100.100.2.2(23), 1 packet
```

---

## EX4) Static NAT (외부→내부 서버 접근 허용)

```
조건:
 # GIT-B 사설망 → GIT-A 내부 사설 서버 192.168.1.1 통신 가능 (static nat: 사설→외부)
 # GIT-A의 Loopback 0 IP 주소를 사용하여 통신
 # 나머지 모든 네트워크는 192.168.1.1로 통신 불가
```

```bash
# GIT-A
ip nat inside source static 192.168.1.1 100.100.1.1
!
ip access-list extended CBAC_ACL_IN
 permit ospf host 121.160.10.11 host 224.0.0.5
 15 permit ip host 121.160.20.2 host 100.100.1.1   # GIT-B IP 허용
 deny ip any any log-input
```

```bash
# 정보 확인 (GIT-B 사설망 PC2 → 내부 서버)
PC2# ping   100.100.1.1   # 통신 O
PC2# telnet 100.100.1.1   # 접속 O

# PC1에서 echo reply 확인
PC1# debug ip icmp
*Mar 1 00:31:06.315: ICMP: echo reply sent, src 192.168.1.1, dst 121.160.20.2

# 나머지 네트워크는 차단
ISP-1~4# ping   100.100.1.1   # 통신 X (UUUUU)
ISP-1~4# telnet 100.100.1.1   # 접속 X (% Destination unreachable)
```

---

## EX5-1) GRE Tunnel 구성

```
조건:
 # GIT-A ~ GIT-B Tunnel IP = 192.168.12.0/24 사용
 # Tunnel 공인 Source/Destination IP = ISP에 직접 연결된 물리 Interface IP 사용
```

```bash
# GIT-A
interface tunnel 12
 ip address 192.168.12.1 255.255.255.0
 tunnel source 121.160.10.1
 tunnel destination 121.160.20.2
```

```bash
# GIT-B
interface tunnel 12
 ip address 192.168.12.2 255.255.255.0
 tunnel source 121.160.20.2
 tunnel destination 121.160.10.1
```

```bash
# GRE 통신 허용 ACL 추가
# GIT-A
ip access-list extended CBAC_ACL_IN
 permit ospf host 121.160.10.11 host 224.0.0.5
 15 permit ip  host 121.160.20.2 host 100.100.1.1
 17 permit gre host 121.160.20.2 host 121.160.10.1
 deny ip any any log-input

# GIT-B
ip access-list extended CBAC_ACL_IN
 permit ospf host 121.160.10.11 host 224.0.0.5
 15 permit gre host 121.160.10.1 host 121.160.20.2
 deny ip any any log-input
```

```bash
# 정보 확인
GIT-A# show ip route connected
C 192.168.12.0/24 is directly connected, Tunnel12

GIT-A# ping 192.168.12.2   # 100% 성공
GIT-B# ping 192.168.12.1   # 100% 성공
```

---

## EX6) EIGRP 사설망 구성 (Tunnel 활용)

### EX6-1) Loopback 1 생성

```bash
# GIT-A
int loop 1
 ip add 222.222.1.1 255.255.255.0

# GIT-B
int loop 1
 ip add 222.222.2.2 255.255.255.0
```

### EX6-2) EIGRP 구성

```
조건:
 # EIGRP AS = 100, 자동 요약(no auto-summary) 사용 안 함
 # GIT-A/GIT-B의 사설 네트워크와 Loopback 1 네트워크는 EIGRP 포함
 # EIGRP 업데이트가 필요한 Interface(Tunnel)로만 송신
```

```bash
# GIT-A
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface tunnel 12
 network 192.168.1.0
 network 192.168.12.0
 network 222.222.1.0

# GIT-B
router eigrp 100
 no auto-summary
 passive-interface default
 no passive-interface tunnel 12
 network 192.168.2.0
 network 192.168.12.0
 network 222.222.2.0
```

```bash
# 정보 확인
GIT-A# show ip route eigrp
D 222.222.2.0/24 [90/297372416] via 192.168.12.2, Tunnel12
D 192.168.2.0/24 [90/297270016] via 192.168.12.2, Tunnel12

GIT-B# show ip route eigrp
D 222.222.1.0/24 [90/297372416] via 192.168.12.1, Tunnel12
D 192.168.1.0/24 [90/297270016] via 192.168.12.1, Tunnel12

PC1# ping 192.168.2.1   # 성공
PC2# ping 192.168.1.1   # 성공
```
