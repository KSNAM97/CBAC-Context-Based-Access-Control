# 🔒 CBAC (Context Based Access Control)

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=flat&logo=cisco&logoColor=white)
![Firewall](https://img.shields.io/badge/Firewall-Stateful-FF6B6B?style=flat)
![Protocol](https://img.shields.io/badge/Protocol-TCP%2FUDP%2FICMP-4CAF50?style=flat)
![VPN](https://img.shields.io/badge/VPN-IPsec%20%7C%20GRE-2196F3?style=flat)
![Routing](https://img.shields.io/badge/Routing-OSPF%20%7C%20EIGRP-9C27B0?style=flat)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat)

> **Cisco IOS Stateful 방화벽 기능을 활용한 Traffic Filtering 보안 솔루션 학습 자료**

---

## 📖 소개

**CBAC(Context Based Access Control)** 는 Cisco IOS의 Stateful 방화벽 기능으로,
상위 프로토콜(HTTP, FTP, SNMP 등) 기반의 애플리케이션을 검사하여
**Dynamic Traffic Filtering** 을 제공하는 보안 솔루션입니다.

기존 ACL이 Layer 3~4 계층만 필터링하는 것과 달리,
CBAC는 **Layer 7(응용 계층)까지 검사**할 수 있으며, 감사(Audit) 및 경고(Alert) 기능을 지원합니다.

---

## 📚 목차

| 챕터 | 제목 | 내용 |
|------|------|------|
| [0](./0_CBAC_이론.md) | **CBAC 이론** | 방화벽 개념, IDS/IPS, Stateful 동작, Timeout, DoS 방어 |
| [1](./1_CBAC_기본실습.md) | **CBAC 기본 실습** | 단일 라우터 환경 TCP/UDP/ICMP 검사 구성 |
| [2](./2_CBAC_실습_NAT_GRE.md) | **CBAC 실습 (NAT + GRE)** | 사설 IP, NAT, Static NAT, GRE Tunnel, EIGRP |
| [3](./3_CBAC_실습_IPsec.md) | **CBAC 실습 (IPsec)** | NAT 환경에서 IPsec VPN 연동 구성 |

---

## 🧩 핵심 개념 요약

- **Stateful 방화벽**: 내부에서 생성된 트래픽의 응답만 허용, 외부 발신 트래픽은 차단
- **State Table**: 모든 연결의 상태 정보를 유지하여 패킷 허용/차단 결정
- **Alert & Audit**: 실시간 이벤트 경고와 트랜잭션 감사 리포팅
- **DoS 방어**: Half-open 세션 임계치 모니터링으로 SYN-flood 등 차단

---

## ⚙️ 실습 환경

```
- GNS3 / Dynamips 기반 Cisco IOS 라우터
- Frame-Relay 백본
- OSPF / EIGRP / RIP 라우팅
- IPsec, GRE Tunnel VPN
```

---

## 🛠️ 주요 명령어 한눈에 보기

```bash
# 검사 규칙 정의
ip inspect name WORD [protocol] [audit-trail on]

# 인터페이스 적용
ip access-group ACL in
ip inspect WORD out

# 검증 및 모니터링
show ip inspect config
show ip inspect session
show ip inspect all
show ip inspect interfaces
```

---

## 📝 License

This project is licensed under the MIT License.
