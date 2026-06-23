# 📘 Chapter 0. CBAC 이론

## 1. CBAC 개요

```
- CBAC은 Cisco IOS Stateful 방화벽 기능으로 Traffic filtering을 제공하는 보안 솔루션이다.
- 향상된 기능의 방화벽 엔진으로 네트워크 보안에서 필수적으로 사용된다.
```

---

## 2. 보안 시스템 비교

### Firewall (방화벽)

```
- 네트워크 단계의 인터넷 보안 시스템 중 가장 널리 사용되는 방법
- 일반적으로 IP/Port를 사용하여 제어하는 기능
- 외부로부터 침입을 '불'에 비유하여 '방화벽'이라고 불린다.
```

### 침입 탐지 시스템 (IDS : Intrusion Detection System)

```
- 네트워크 또는 시스템에 유입되는 트래픽을 감시하여 공격이나 비정상 동작을 탐지하는 시스템
- 알려진 공격 패턴, 비정상 트래픽, 정책 위반 여부 등을 분석
- 공격이 탐지되면 로그를 기록하고 관리자에게 경고 또는 보고
- 일반적으로 트래픽 경로 옆에서 패킷을 복사하여 분석하는 방식으로 동작
- 기본적으로 공격을 탐지하고 알리는 기능을 수행하며, 직접 트래픽을 차단하지 않는다.

  공격 트래픽 탐지 ---> 로그 기록 및 경고 ---> 관리자가 확인 후 대응

- NIDS : 네트워크 트래픽을 감시하는 IDS
- HIDS : 서버나 호스트 내부의 파일, 로그, 프로세스를 감시하는 IDS
```

### 침입 방지 시스템 (IPS : Intrusion Prevention System)

```
- 네트워크 트래픽을 실시간 감시하고 공격/비정상 트래픽을 탐지하여 자동으로 차단하는 시스템
- IDS와 같이 공격 패턴과 이상 동작을 분석하지만, 탐지 이후 직접 차단할 수 있다는 차이가 있다.
- 트래픽이 반드시 IPS를 통과하도록 네트워크 경로 중간에 배치된다.

  [공격 탐지 시 수행 동작]
  - 해당 패킷 폐기
  - 공격자의 IP 주소 차단
  - TCP Session 강제 종료
  - 관리자에게 경고 및 로그 전송
  - 방화벽 정책 또는 차단 목록 갱신

  트래픽 수신 ---> 공격 여부 검사 ---> 정상 트래픽 : 통과
                                       공격 트래픽 : 차단 및 관리자 보고

※ IDS = 공격 탐지 + 알림 중심
※ IPS = 공격 탐지 + 직접 차단까지 수행
```

---

## 3. Stateful 방화벽

```
- 외부에서 입력되는 트래픽은 기본적으로 차단되지만
  내부에서 생성된 트래픽에 대한 응답 트래픽은 허용하는 방화벽
```

---

## 4. Traffic Filtering

```
- CBAC은 HTTP, SNMP, FTP 등과 같은 상위 Protocol의 어플리케이션을 기반으로 하는
  TCP, UDP 등을 걸러내는 Dynamic Traffic filtering을 지원하는 방화벽이다.
- CBAC 기능을 활용하려면 보호되어야 할 트래픽과 보호하지 않아야 할 트래픽을 분리해야 한다.
```

---

## 5. CBAC 기본 동작 방식

```
- CBAC으로 설정한 트래픽의 입/출력 시 최상단에 허용하는 기능이 State table에 추가된다.
- State table에 Match되는 트래픽은 허용된다.
- State table에 Match되지 않는 트래픽은 ACL의 적용을 받는다.
```

---

## 6. CBAC 주요 기능

```
- 외부 침입으로부터 내부 네트워크를 보호
- DoS 공격으로부터 내부 네트워크를 보호
- 각 flow와 session의 상태를 추적
  (Network Layer 뿐 아니라 Transport Layer와 상위 Layer까지 정보 검사 가능)
- 네트워크 전반에서 각 어플리케이션에 대한 통제 가능
- 방화벽을 통과하는 모든 연결의 상태 정보를 State table 안에서 유지/관리
  (Packet의 허용/차단 여부 결정을 위해 State table 사용)
- 실시간 이벤트 경고와 감사 실시
- 공격 의심 트래픽 감지 시 Syslog 에러 메시지를 console로 출력
- 응용계층 필터링 기능 제공, NAT/PAT에서 내부주소까지 변환
- FTP, H.323처럼 복수 세션을 사용하는 어플리케이션도 stateful 방화벽 기능 지원
```

---

## 7. CBAC 동작 과정 / 제약사항

```
- 지정된 트래픽에 대해서만 검사를 실시
- Router 자신이 생성하는 트래픽이나 목적지가 Router인 트래픽에는 적용되지 않음
- IPsec과 같이 암호화된 패킷은 검사하지 못함
- 모든 어플리케이션을 지원하지 못하므로 특정 어플리케이션은 CBAC을 중지해야 함
```

---

## 8. CBAC vs Reflexive ACL

### 공통점

```
- 내부→외부 향하는 모든 트래픽 허용, 외부→내부 향하는 모든 트래픽 차단
- 내부에서 생성되어 리턴되는 트래픽은 State Table을 사용하여 허용
- 지정된 Traffic에 대해서만 Filtering
```

### 차이점

```
- Reflexive ACL : Layer 3, Layer 4 계층에 대해서만 Filtering 가능
- CBAC          : Layer 7까지 Filtering 가능
                  감사(Audit) 기능과 경고(Alert) 기능 지원
```

---

## 9. CBAC 경고(Alert) & 감사(Audit)

```
- CBAC은 트래픽 검사에 더해서 State table 안에 유지되는
  모든 세션 정보를 위한 실시간 이벤트 경고(alert) 및 감사(audit)를 만들 수 있다.
- 네트워크의 모든 Transaction, 출발지/목적지 IP 주소, Port 번호, 시간정보가 포함되며
  Console에서 Syslog를 사용하여 이벤트 경고 메시지를 출력하고 감사 리포팅한다.
```

### 경고 (Alert)

```
- 비정상 동작, 공격, 장애 또는 정책 위반이 탐지되었을 때 관리자에게 즉시 알려주는 기능
- 관리자가 빠르게 대응해야 하는 현재의 사건/위험을 알리는 것이 목적
- IDS, IPS, 방화벽, 서버, 네트워크 장비 등에서 발생 가능
- 전달 방식: 화면 메시지, 로그, 이메일, SMS, SNMP Trap, Syslog 등
```

### 감사 (Audit)

```
- 시스템, 네트워크 장비 또는 사용자가 수행한 작업을 기록하고 나중에 확인하는 기능
- 누가, 언제, 어디서, 무엇을 수행했는지 추적하기 위해 사용
- 목적: 보안 사고 분석, 사용자 행위 추적, 규정 준수, 책임 확인
- 즉시 경고보다는 발생한 작업과 이벤트의 기록 및 검토에 중점
```

---

## 10. 프로토콜별 Timeout

### CBAC TCP

```
- TCP는 연결형 Protocol이므로 3way-handshake 과정을 통해 송신측/수신측이 negotiation 실시
- State Table을 생성 및 유지하며, 내부→외부 통신 시 State table에 Session 생성
- 리턴되어 돌아오는 트래픽은 State table에 목록이 존재하므로 허용
- 외부에서 생성되어 내부로 입력되는 트래픽은 State table에 존재하지 않으므로 차단
- 3600초간 트래픽이 없으면 세션 단절
- FIN 수신 후 5초가 지나면 State table에서 해당 세션 제거
```

### CBAC UDP

```
- 30초간 해당 트래픽이 없으면 통신 종료로 간주하여 State table에서 제거
- [DNS Spoofing, DoS 공격 방지를 위해 내부→외부 DNS Query 전송 후
   5초 내 응답 없으면 해당 Session 삭제]
- DNS 서버가 정상 응답하면 즉시 해당 세션을 테이블에서 제거
```

### CBAC ICMP

```
- ICMP 세션은 10초 이내에 응답이 없으면 해당 entry 제거
```

---

## 11. Timeout & 임계치 값

```
- CBAC은 State table 정보를 관리하기 위해 Timeout과 임계치 값을 사용한다.
- 연결이 완벽하게 확립되지 않은 Session의 제거 시점을 결정할 수 있다.
- Session의 출발지/목적지 양쪽에서 연결되지 않은 모든 Session에
  Reset message를 전송하여 불완전한 Session 연결을 해제한다.

[임계치 모니터링 3가지 방법]
- Half-open된 Session의 전체 수
- 시간을 기반으로 한 Half-open Session의 수
- Host별로 Half-open된 TCP Session의 수
```

### Half-open & DoS 방어

```
- CBAC은 DoS 탐지와 방지 기능을 제공한다.
- 과도한 수의 Half-open session 발생은 DoS 공격 가능성을 나타낸다. (예: TCP Syn-flood)
- UDP는 hand shake 메커니즘이 없어 방화벽에서 Half-open을 차단하지 못한다.
- TCP/UDP의 Session 전체 수 또는 연결 시도 비율을 1분 주기로 모니터링하여
  Half-open Session 수를 통제함으로써 DoS 공격을 차단한다.
```

```bash
# 전역 Half-open 임계치
Router(config)# ip inspect max-incomplete high <number>   # Half-open TCP Session 임계치
Router(config)# ip inspect max-incomplete low  <number>   # 설정값 이하인 경우 해당 트래픽 처리

# 분당 Half-open 임계치
Router(config)# ip inspect one-minute high <number>       # 분당 Half-open TCP Session 임계치
Router(config)# ip inspect one-minute low  <number>       # 설정값 이하인 경우 해당 트래픽 처리
```

### Host별 DoS 방어

```
- CBAC은 TCP를 기반으로 Host별 DoS 방어 기능을 지원한다.
- 동일한 Host 주소로 만들어진 Half-open session의 전체 수를 모니터링한다.
- Half-open session 개수가 임계치를 초과하면 지정한 시간 동안 해당 Host의 모든 연결을 차단한다.
```

```bash
Router(config)# ip inspect tcp max-incomplete host <number>
Router(config)# ip inspect tcp max-incomplete host <number> block-time <minute>
```

---

## 12. 설정 방법

### 1) Interface (내부/외부) 선택

```
- CBAC은 방화벽 내부/외부 모두에 설정이 가능하다.
  # Internal : 보호 받아야 하는 네트워크 구간, 트래픽 허용을 위해 Session이 시작되는 구간
  # External : 보호 받지 않는 네트워크 구간, Session을 시작할 수 없는 구간

- Interface마다 단방향 설정을 권장하지만
  Intranet, Extranet, DoS 공격 방어에 따라 양쪽 네트워크를 모두 보호해야 하는 경우
  양방향 설정도 가능하다.
```

### 2) IP ACL 설정

```
- CBAC이 동작하려면 리턴되어 입력되는 트래픽에 대해
  일시적으로 방화벽이 개방되도록 IP ACL을 설정해야 한다.
- ACL 구성에 정답은 정해져 있지 않으며 조직의 보안 정책(또는 가이드라인)에 따라 설정한다.
```

### 3) 검사 규칙 정의

```
- 어떤 어플리케이션이 방화벽 엔진에 의해 검사되어야 하는지 명확하게 정의해야 한다.
- 원하는 어플리케이션 계층뿐 아니라 포괄적인 TCP, UDP도 명시해야 한다.
```

```bash
Router(config)# ip inspect name WORD [application | protocol]
```

### 4) Global Timeout / 임계치 설정

```
- Session 상태 유지/관리를 위해 Timeout과 임계치값을 사용한다.
- 불완전하게 종료된 Session들은 Timeout과 임계치값으로 삭제할 수 있다.
```

```bash
ip inspect tcp synwait-time <sec : default 30sec>    # Syn 수신 후 대기시간, 미응답 시 삭제
ip inspect tcp finwait-time <sec : default 5sec>     # Fin 송신 후 대기시간, 미응답 시 삭제
ip inspect tcp idle-time    <sec : default 3600sec>  # TCP 통신 idle 시 Session 삭제
ip inspect udp idle-time    <sec : default 30sec>    # UDP 통신 idle 시 Session 삭제
ip inspect dns idle-time    <sec : default 30sec>    # DNS 통신 idle 시 Session 삭제
```

### 5) 특정 URL 허용/차단

```bash
ip inspect name WORD http urlfilter
ip urlfilter exclusive-domain permit (허용할 url 설정)
ip urlfilter exclusive-domain deny   (차단할 url 설정)
```

### 6) Interface에 ACL과 검사 규칙 적용 (CBAC 적용)

```
- CBAC을 동작시키려면 ACL과 함께 CBAC을 Interface에 적용해야 한다.
- 내부/외부 모든 Interface에 적용 가능하므로 조직의 보안 정책에 따라 적용한다.
  # 내부→외부 전송 트래픽에 CBAC 적용 시 외부 Interface에 적용
  # 외부→내부 전송 트래픽에 CBAC 적용 시 내부 Interface에 적용
```

```bash
interface serial | fastethernet
 ip access-group x  in/out
 ip inspect WORD    in/out
```

### 검증 및 모니터링

```bash
show ip inspect config
show ip inspect session
show ip inspect all
show ip access-lists
```
