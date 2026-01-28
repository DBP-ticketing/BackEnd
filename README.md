# MOL 티케팅

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/00dcedc8-af5b-4e9e-a9a3-c90be6d6c082"
    width="250"
    height="250"
    style="border-radius: 16px;"
  />
</p>



## 목차
- [프로젝트 소개](#프로젝트-소개)
- [개발 기간](#개발-기간)
- [팀 구성](#팀-구성)
- [주요 기능](#주요-기능)
- [기술 스택](#기술-스택)
- [시스템 아키텍처](#시스템-아키텍처)
- [핵심 기술 구현](#핵심-기술-구현)
- [기능 시연](#기능-시연)
- [트러블슈팅](#트러블슈팅)
- [프로젝트 회고](#프로젝트-회고)

## 프로젝트 소개

MOL 티케팅은 공연 티켓 오픈 시 발생하는 폭발적인 트래픽과 동시성 문제를 해결하기 위해 기획된 시스템입니다. 

단순히 예매 기능을 구현하는 것을 넘어, **RDBMS 락의 한계를 Redis 기반의 진입 대기열과 분산 락으로 극복**하여 극한의 부하 상황에서도 안정적인 예매 환경을 제공하는 것을 목표로 합니다.

### 프로젝트 배경

- 대규모 트래픽 상황에서의 티켓 예매 시스템 구현 경험 필요
- 동시성 제어와 분산 처리 기술에 대한 학습 및 실전 적용
- 실제 서비스 수준의 티케팅 플랫폼 아키텍처 설계 경험

## 개발 기간

**2025.09 ~ 2025.12** (약 4개월)

## 팀 구성

**Backend 2명**

## 주요 기능

### 1. 사용자 인증 및 권한 관리

- 일반 사용자(USER), 호스트(HOST), 관리자(ADMIN) 역할 구분
- JWT 기반 Access Token / Refresh Token 발급
- Redis를 활용한 로그아웃 시 토큰 블랙리스트 관리
- 호스트 승인 시스템 (관리자 승인 필요)

### 2. 이벤트 및 장소 관리

#### 다양한 좌석 형태 지원
- **지정좌석 (ASSIGNED)**: 행/열 지정
- **구역별 지정좌석 (SEAT_WITH_SECTION)**: 구역별 등급 및 가격 차등
- **자유좌석 (FREE)**: 구역만 지정
- **스탠딩 (STANDING)**: 좌석 없음

#### 기타 기능
- 장소별 좌석 템플릿 시스템
- 이벤트 상태 관리 (예매 전/예매 중/마감/취소/종료)
- 스케줄러를 통한 자동 예매 오픈

### 3. Redis 기반 대기열 시스템 🔥
<img width="752" height="343" alt="image" src="https://github.com/user-attachments/assets/d3f22955-1136-485a-a500-0b281864f4df" />


#### 대기열 관리
- **진입 시점**: 이벤트 페이지 접속 시 자동으로 대기열 등록
- **순위 확인**: 실시간으로 대기 순번 조회 가능
- **자동 입장**: 스케줄러가 1초마다 Active 목록 관리
  - 최대 100명까지 동시 입장 허용
  - 5분 동안 활동이 없으면 자동 퇴장
  - 빈자리만큼 대기열에서 순차 입장

#### Interceptor를 통한 진입 제어
- 좌석 조회 및 예매 API는 Active 상태만 접근 가능
- 헤더에 `Queue-Token`, `Event-Id` 검증

### 4. Redisson 분산 락을 활용한 동시성 제어 🔥

- Redis 기반 분산 락으로 좌석별 동시성 제어
- tryLock 방식으로 타임아웃 설정 (대기 3초, 유지 10초)
- 지정석은 분산 락 적용, 자유석/스탠딩은 재고 차감 방식

#### 성능 검증
- 1000명이 동일 좌석 예매 시도
- ✅ **성공: 1건** (정확히 1명만 예매 성공)
- ❌ **실패: 999건** (나머지는 "이미 예약된 좌석" 에러)

### 5. 예매 및 결제 시스템

- 좌석 상태 관리: AVAILABLE → RESERVED → SOLD
- 5분 내 미결제 시 자동 취소 (스케줄러)
- 카카오페이 결제 연동
  - 결제 준비 (Ready)
  - 결제 승인 (Approve)
  - 결제 취소/실패 처리
- 예매 내역 조회 및 상태별 필터링

### 6. 히스토리 관리

- 좌석 예매 상태 변경 이력 추적
- 예매 상태 변경 시마다 자동 히스토리 생성

## 기술 스택

### Backend
![Java](https://img.shields.io/badge/Java_17-007396?style=for-the-badge&logo=java&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot_3.4.1-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring_Security-6DB33F?style=for-the-badge&logo=spring-security&logoColor=white)
![Spring Data JPA](https://img.shields.io/badge/Spring_Data_JPA-6DB33F?style=for-the-badge&logo=spring&logoColor=white)

### Database
![MySQL](https://img.shields.io/badge/MySQL_8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)

### Infrastructure
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

### Tools
![Redisson](https://img.shields.io/badge/Redisson-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=json-web-tokens&logoColor=white)
![Swagger](https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)

## 시스템 아키텍처

### 시스템 구조

<img width="3038" height="1342" alt="image" src="https://github.com/user-attachments/assets/396b2bb4-225a-46e2-9d1c-a2157e74e0a5" />


### ERD

<img width="410" alt="ERD" src="https://github.com/user-attachments/assets/c1b055fd-2b33-406f-81a0-fd6bed57823d" />

## 핵심 기술 구현

### 1. Redis Sorted Set을 활용한 대기열 시스템

#### 대기열 등록
- 타임스탬프를 Score로 사용하여 선착순 보장
- 이벤트별로 독립적인 대기열 관리

#### 순위 조회
- Redis의 ZRANK 명령어로 O(log N) 시간 복잡도로 순위 조회
- 실시간으로 내 대기 순번 확인 가능

#### 스케줄러를 통한 자동 입장 관리
- 1초마다 실행되는 스케줄러
- 1시간 넘게 대기한 유저 자동 제거 (Ghost 제거)
- 5분 지난 Active 유저 자동 퇴장 (Time-out)
- 빈자리만큼 대기열에서 자동 입장 처리

### 2. Redisson 분산 락

#### Redisson을 선택한 이유
- **Lettuce의 한계**: Spin Lock 방식으로 Redis에 부하 발생
- **Redisson의 장점**:
  - Pub/Sub 기반으로 효율적인 대기
  - tryLock으로 타임아웃 설정 가능
  - 데드락 방지 기능 내장

#### 분산 락 적용 방식
- 좌석별로 고유한 락 키 생성: `lock:seat:{seatId}`
- 대기 시간 3초, 락 유지 시간 10초 설정
- finally 블록에서 안전하게 락 해제

### 3. JWT 기반 인증 시스템

#### 토큰 구조
- **Access Token**: 1시간 유효, API 인증에 사용
- **Refresh Token**: 7일 유효, Redis에 저장하여 관리

#### 보안 강화
- 로그아웃 시 Access Token을 블랙리스트에 등록
- Refresh Token은 DB에서 즉시 삭제
- JWT 필터에서 블랙리스트 검증

### 4. 카카오페이 결제 연동

#### 결제 흐름
1. **결제 준비**: 카카오페이 결제 페이지 URL 발급
2. **사용자 결제**: 카카오페이 페이지에서 결제 진행
3. **결제 승인**: pg_token을 받아 최종 승인 요청
4. **상태 변경**:
   - Booking: PENDING → CONFIRMED
   - Seat: RESERVED → SOLD
   - Payment: READY → APPROVED

## 기능 시연
![MOL_시연영상+(1) (1)](https://github.com/user-attachments/assets/75dcaa78-064e-4537-a6a6-4251194474c6)



## 트러블슈팅

### 1. 동시성 문제: 같은 좌석에 중복 예매 발생 🔥

#### 문제 상황
- 1000명이 동시에 같은 좌석 예매 시도
- 낙관적 락(@Version) 사용 시: 1명만 성공, 나머지는 재시도 필요
- 비관적 락(PESSIMISTIC_WRITE) 사용 시: DB 부하 증가

#### 해결 과정

**1단계: 비관적 락 시도**
- ✅ 동시성 문제 해결
- ❌ DB 커넥션 부족 및 성능 저하

**2단계: Redisson 분산 락 도입**
- Redis 기반 분산 락으로 전환
- DB 부하를 Redis로 분산
- tryLock으로 타임아웃 제어

#### 결과
- ✅ DB 부하 90% 감소
- ✅ 처리 속도 3배 향상
- ✅ 1000명 동시 예매 시 정확히 1명만 성공

### 2. 대기열 타임아웃 문제

#### 문제 상황
- 사용자가 대기열에 입장 후 예매하지 않고 방치
- Active 슬롯이 계속 점유되어 다음 사용자가 입장하지 못함

#### 해결 방법
- 스케줄러로 5분마다 Active 유저의 활동 시간 체크
- 5분 초과 시 자동 퇴장 처리
- 빈자리만큼 대기열에서 자동 입장

#### 결과
- ✅ 활동이 없는 사용자 자동 퇴장
- ✅ 대기자 자동 입장으로 처리량 증가

### 3. 만료된 예약 처리

#### 문제 상황
- 결제하지 않은 예약이 5분 후에도 좌석을 점유
- 다른 사용자가 해당 좌석 예매 불가

#### 해결 방법
- 스케줄러를 통한 자동 취소 시스템 구현
- 매 분마다 5분 이상 경과한 PENDING 예약 조회
- 좌석 상태 RESERVED → AVAILABLE 변경
- 대기열에서 해당 유저 제거

#### 결과
- ✅ 자동으로 좌석 반환
- ✅ 다른 사용자에게 예매 기회 제공

## 프로젝트 회고

### 배운 점
1. **대규모 트래픽 처리**: Redis를 활용한 대기열 및 분산 락 구현
2. **동시성 제어**: Redisson을 통한 효율적인 락 관리
3. **시스템 설계**: 확장 가능한 아키텍처 설계 경험
4. **문제 해결**: 실전 성능 이슈 해결 경험

### 개선 사항
- 모니터링 시스템 추가 (Prometheus + Grafana)
- 로그 분석 시스템 구축 (ELK Stack)
- CI/CD 파이프라인 고도화
- 부하 테스트 자동화 (JMeter)

## 링크

- **GitHub Repository**: [[프로젝트 링크]](https://github.com/DBP-ticketing/ticketing-backend)
- **프로젝트 발표 자료**: [[PDF 링크]](https://drive.google.com/file/d/1Z0sPf0I_X6R-Zmi6BrMrUTVc0WUEX2Qr/view?usp=drive_link)
