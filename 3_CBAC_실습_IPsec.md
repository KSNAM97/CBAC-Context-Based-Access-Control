# 📕 Chapter 3. CBAC 실습 (IPsec VPN)

> ⚠️ CBAC는 IPsec으로 암호화된 패킷을 검사할 수 없습니다.
> 따라서 **① NAT 예외 처리** + **② IPsec 프로토콜(ISAKMP/ESP) ACL 허용** 이 핵심입니다.

## 🗺️ 장비 구성

```
- R1 = GIT-A      - R5 = ISP-3
- R2 = GIT-B      - R6 = ISP-4
- R3 = ISP-1      - R7 = PC1
- R4 = ISP-2      - R8 = PC2
```

> ※ 공중망(OSPF), NAT(EX2), CBAC(EX3), Static NAT(EX4)는 Chapter 2와 동일하게 선행 구성합니다.

---

## EX5) IPsec 구성

```
조건:
 # GIT-A 192.168.1.0/24 ↔ GIT-B 192.168.2.0/24 통신 시 정보보호 실시
 # GIT-A IPsec 구성 시 Loopback 1 (100.100.100.1/24) IP 주소 사용
 # GIT-B IPsec 구성 시 Loopback 0 IP 주소 사용

 [Phase 1]                              [Phase 2]
 - 인증방식   : 사전 인증(pre-share)     - IPsec Protocol : ESP
 - 암호화     : AES                      - 암호화         : AES
 - 인증       : MD5                      - 인증           : SHA-HMAC
 - Key교환    : Diffie-Hellman 5
 - Key교환주기 : 10분 (600초)
```

### Phase 1 & Phase 2 설정

```bash
# GIT-A (R1)
interface loopback1
 ip address 100.100.100.1 255.255.255.0
!
router ospf 1
 network 100.100.100.0 0.0.0.255 area 0
!
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
!
crypto isakmp enable
crypto isakmp policy 10
 authentication pre-share
 encryption aes
 hash md5
 group 5
 lifetime 600
!
crypto isakmp key 0 cisco address 100.100.2.2
!
crypto ipsec transform-set IPSEC esp-aes esp-sha-hmac
!
crypto map IPSEC_VPN 10 ipsec-isakmp
 set peer 100.100.2.2
 set transform-set IPSEC
 match address 101
!
crypto map IPSEC_VPN local-address loopback 1
!
interface serial 1/0.10
 crypto map IPSEC_VPN
```

```bash
# GIT-B (R2)
access-list 101 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
!
crypto isakmp enable
crypto isakmp policy 10
 authentication pre-share
 encryption aes
 hash md5
 group 5
 lifetime 600
!
crypto isakmp key 0 cisco address 100.100.100.1
!
crypto ipsec transform-set IPSEC esp-aes esp-sha-hmac
!
crypto map IPSEC_VPN 10 ipsec-isakmp
 set peer 100.100.100.1
 set transform-set IPSEC
 match address 101
!
crypto map IPSEC_VPN local-address loopback 0
!
interface serial 1/0.20
 crypto map IPSEC_VPN
```

```bash
# 이 시점에서는 아직 통신 X (라우팅 정보 없음)
PC1# ping 192.168.2.2 source 192.168.1.1   : 통신 X
PC2# ping 192.168.1.1 source 192.168.2.2   : 통신 X
```

---

### 정적 경로 설정 (상대 사설망 도달)

```
# 라우팅 테이블에 상대 사설 네트워크 정보가 없으므로 Static Route 추가
```

```bash
# GIT-A (R1)
ip route 192.168.2.0 255.255.255.0 121.160.10.11

# GIT-B (R2)
ip route 192.168.1.0 255.255.255.0 121.160.20.4
```

```bash
# 정보 확인
GIT-A# show ip route | include 192.168
C 192.168.1.0/24 is directly connected, FastEthernet0/0
S 192.168.2.0/24 [1/0] via 121.160.10.11

GIT-B# show ip route | include 192.168
S 192.168.1.0/24 [1/0] via 121.160.20.4
C 192.168.2.0/24 is directly connected, FastEthernet0/0

# 라우팅은 됐지만 NAT 때문에 여전히 통신 X
PC1# ping 192.168.2.2 source 192.168.1.1   : 통신 X
```

---

### ① NAT 예외 처리 (핵심)

```
# IPsec 통신 시에는 출발지 IP가 변경되면 안 되므로 NAT 기능을 수정해야 한다.
# IPsec 대상 트래픽(상대 사설망)은 NAT 제외(deny), 나머지만 NAT 적용(permit)
```

```bash
# GIT-A (R1)
no access-list 1
no ip nat inside source list 1 pool NAT overload
!
access-list 110 deny   ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
access-list 110 permit ip 192.168.1.0 0.0.0.255 any
!
ip nat inside source list 110 pool NAT overload
```

```bash
# GIT-B (R2)
no access-list 1
no ip nat inside source list 1 pool NAT overload
!
access-list 110 deny   ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 110 permit ip 192.168.2.0 0.0.0.255 any
!
ip nat inside source list 110 pool NAT overload
```

```bash
# 정보 확인 (사설망 외 일반 통신은 NAT로 정상 동작)
PC1# ping 100.100.12.2   : NAT에 의해 외부 통신 가능
PC1# ping 100.100.13.3   : NAT에 의해 외부 통신 가능
PC2# ping 100.100.12.2   : NAT에 의해 외부 통신 가능
PC2# ping 100.100.13.3   : NAT에 의해 외부 통신 가능
```

---

### ② IPsec 프로토콜 ACL 허용 (핵심)

```
# CBAC 구성 시 Static NAT 외 모든 트래픽이 ACL에 의해 차단되므로
# IPsec 통신을 위한 Protocol(ISAKMP/UDP 500, ESP)을 허용해야 한다.
```

```bash
# GIT-A (R1)
ip access-list extended ACL_IN
 permit ospf host 121.160.10.11 host 224.0.0.5
 12 permit udp host 100.100.2.2 eq 500 host 100.100.100.1 eq 500   # ISAKMP 허용
 13 permit esp host 100.100.2.2        host 100.100.100.1          # ESP 허용
 permit ip host 121.160.20.2 host 100.100.1.1
 deny ip any any log-input
```

---

### 최종 정보 확인

```bash
# 최초 SA 협상 시 첫 패킷 손실 가능 (80%)
PC1# ping 192.168.2.2 source 192.168.1.2
.!!!!
Success rate is 80 percent (4/5)

# ISAKMP Peer 확인
GIT-A# show crypto isakmp peer
Peer: 100.100.2.2 Port: 500 Local: 100.100.100.1
 Phase1 id: 100.100.2.2

# Crypto Engine 연결 확인
GIT-A# show crypto engine connections active
 ID  Interface  Type   Algorithm   Encrypt Decrypt IP-Address
  1  Se1/0.10   IPsec  AES+SHA        0       9    100.100.100.1
  2  Se1/0.10   IPsec  AES+SHA        9       0    100.100.100.1
1001 Se1/0.10   IKE    MD5+AES        0       0    100.100.100.1

# SA 수립 후 정상 통신 (100%)
PC1# ping 192.168.2.2 source 192.168.1.1
!!!!!
Success rate is 100 percent (5/5)

PC2# ping 192.168.1.1 source 192.168.2.2
!!!!!
Success rate is 100 percent (5/5)
```
