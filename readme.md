# 이력서

---

## 인적사항

| 항목       | 내용                             |
| ---------- | -------------------------------- |
| **이름**   | 양지웅                           |
| **이메일** | didwldnd0722@gmail.com           |
| **GitHub** | https://github.com/Peppertunacan |

---

## 자기소개

데이터 파이프라인부터 IoT 임베디드, 프론트엔드까지 **End-to-End 개발 경험**을 보유한 개발자입니다.
Airflow/Kafka/Spark 기반 ETL 파이프라인 구축, Raspberry Pi와 RFID를 활용한 IoT 시스템 개발, Next.js/React Native 기반 사용자 인터페이스 구현까지 — 데이터가 수집되어 사용자에게 전달되기까지의 **전체 흐름을 이해하고 구현**할 수 있습니다.

---

## 기술 스택

| 분류            | 기술                                                        |
| --------------- | ----------------------------------------------------------- |
| **Frontend**    | React, Next.js, Vue.js, TypeScript, Tailwind CSS            |
| **Mobile**      | React Native, Expo                                          |
| **Backend**     | Django, Spring Boot, Python, Java                           |
| **IoT**         | Raspberry Pi, MQTT, RFID                                    |
| **Data**        | Apache Airflow, Kafka, Spark, Flink, HDFS                   |
| **Database**    | PostgreSQL, Redis, Elasticsearch                            |
| **DevOps**      | Docker, Nginx                                               |

---

## 프로젝트 경험

### 1. Agora - 실시간 뉴스 토픽 분석 플랫폼

> 정치·사회 이슈에 대한 찬성/반대/중립 입장을 시각화하여 보여주는 한국형 공론장 플랫폼

| 항목 | 내용 |
| --- | --- |
| **기간** | 2025.10.27 ~ 2025.11.20 |
| **인원** | 4명 |
| **역할** | Frontend Developer |
| **기술** | Next.js 16 (App Router), React 19, TypeScript 5, Tailwind CSS 4 |
| **GitHub** | https://github.com/Peppertunacan/agora |

**담당 업무**
- SSR / ISR / CSR 렌더링 전략 설계 및 적용 (토픽 상세 SSR, 메인 ISR 60초, 댓글 CSR)
- Custom Fetch Client 구현 (타임아웃, 로깅, 캐싱, 에러 핸들링) 및 Service Layer Pattern 적용
- ThemeToggle, TrendingBanner, LoadingSkeleton, ErrorBoundary 등 공통 컴포넌트 시스템 구축
- 입장별(찬/반/중립) 스타일 시스템 설계 (`stanceStyles.ts` 유틸리티)

**성과**
- Dynamic Import 코드 스플리팅 + ISR 캐싱으로 Lighthouse 성능 점수 개선
- Hydration Mismatch 해결 (`mounted` 상태 체크로 SSR/CSR 불일치 해결)
- Next.js 16 Breaking Change (`params` Promise 변경) 마이그레이션 대응

---

### 2. MSKT (만사경통) - 건설현장 근로자 관리 플랫폼

> 건설 현장, 인력사무소, 원청을 위한 통합 근로자 관리 모바일 애플리케이션

| 항목 | 내용 |
| --- | --- |
| **기간** | 2025.09.01 ~ 2025.09.29 |
| **인원** | 5명 |
| **역할** | Mobile Developer, PM |
| **기술** | React Native 0.81, Expo SDK 54, TypeScript 5.9, React Native Reanimated 4.1 |
| **GitHub** | https://github.com/Peppertunacan/mskt |

**담당 업무**
- `WorkStatusContext` 기반 6단계 실시간 근무 상태 관리 시스템 구현 (IDLE → WAITING_ASSIGNMENT → WORK_ASSIGNED → WAITING_CONTRACT → WORKING → WAITING_CHECKOUT)
- `expo-camera` 활용 QR 코드 출퇴근 시스템 개발
- `react-native-signature-canvas` 기반 전자 서명 시스템 구현
- FCM 푸시 알림 서비스 개발 (토큰 관리, 채널 설정, 포그라운드/백그라운드 핸들링)
- Android 생체인증 서비스 구현 (지문인식 + 비밀번호 폴백)
- Contract, StatusCard, WorkHistory 등 모듈화된 컴포넌트 시스템 설계
- **PM**: Jira 스프린트 관리, 스토리 포인트 기반 작업량 산정, Jira+Git+Mattermost 자동화 연동, 기획 및 발표

**성과**
- 앱 번들 크기 약 15MB (압축 후)
- useEffect cleanup 기반 메모리 누수 방지
- 플래그 기반 토큰 등록/삭제 동시성 제어

---

### 3. SOBI - AIoT 스마트 바스켓

> RFID 기술과 AI를 결합한 차세대 스마트 쇼핑 솔루션의 IoT/Embedded 시스템

| 항목 | 내용 |
| --- | --- |
| **기간** | 2025.07.15 ~ 2025.08.17 |
| **인원** | 6명 |
| **역할** | Project Manager, IoT·Embedded Developer |
| **기술** | Python 3.9+, Raspberry Pi, YRM1001 RFID Reader, MQTT (Paho), I2C LCD |
| **GitHub** | https://github.com/Peppertunacan/smart-online-basket-interface |

**담당 업무**
- MQTT 메인 컨트롤러 구현 (백엔드 명령 구독, RFID 시스템 생명주기 관리)
- RFID 센서 모듈 개발 (멀티폴링 태그 스캔, RSSI 기반 감지 정확도 향상, CartManager 상태 추적)
- I2C LCD 디스플레이 제어 (실시간 금액 표시, 백라이트 자동 관리)
- MQTT Pub/Sub 통신 모듈 (QoS Level 1, 브로커 재연결 로직)
- **PM**: Jira 백로그 관리, HW/SW 팀 간 스탠드업 조율, 기획 및 발표

**성과**
- 상품 인식 후 프론트엔드/LCD 업데이트 **2초 이내** 달성
- QoS Level 1 적용으로 MQTT 메시지 유실 방지
- RSSI 기반 필터링으로 오인식률 감소
- LCD 미사용 시 백라이트 자동 off로 전력 절약

---

### 4. AI News Curation - AI 뉴스 큐레이션 서비스

> AI 기반 뉴스 큐레이션 서비스 - 수집부터 추천까지 전체 파이프라인 구현

| 항목 | 내용 |
| --- | --- |
| **기간** | 2025.04.11 ~ 2025.05.27 |
| **인원** | 2명 |
| **역할** | Full Stack Developer (Backend, Frontend, Data Engineering) |
| **기술** | Django 5.1, DRF, PostgreSQL, pgvector, Elasticsearch 8.x, LangChain, Vue 3, Pinia, Airflow, Kafka, Flink, Spark, HDFS |
| **GitHub** | [front-pjt](https://github.com/Peppertunacan/front-pjt) · [backend-pjt](https://github.com/Peppertunacan/backend-pjt) · [data-pjt](https://github.com/Peppertunacan/data-pjt) |

**담당 업무**
- **Backend**: JWT 인증 시스템, RESTful API 설계, Elasticsearch 연동 검색 API, pgvector 벡터 유사도 검색, LangChain RAG 챗봇 구현
- **Frontend**: Vue 3 Composition API 기반 SPA 개발, Pinia 상태관리, Chart.js 대시보드 시각화
- **Data ETL**: Airflow DAG 설계/스케줄링, Kafka Producer/Consumer, Flink 실시간 스트림 처리, Spark 배치 처리, GPT-4o-mini 자동 분류, text-embedding-3-small 벡터화

**성과**
- Flink 실시간 + Spark 배치 하이브리드 아키텍처 구현
- 한국어 형태소 분석기(Nori) 적용 Elasticsearch 검색 최적화
- LangChain + 벡터 DB 기반 RAG 챗봇으로 기사 질의응답 구현
- Airflow로 수집/분류/임베딩 파이프라인 자동화

---

## 이력

| 기간              | 기관                  | 세부 사항                     | 비고      |
| ----------------- | --------------------- | ------------------------- | --------- |
| 2019.06 ~ 2025.02 | 경북대학교            | 컴퓨터학부 심화컴퓨터학과 | 졸업      |
| 2025.01 ~ 2025.12 | 삼성청년AI·SW아카데미 | 데이터트랙                | 수료     |
| 2025.12 ~ 2026.06 | 한국수자원공사(K-water) | 전산직렬 체험형 인턴 |  예정 |
---

## 자격증

| 취득일     | 자격증              | 발급기관             |
| ---------- | ------------------- | -------------------- |
| 2024.06.18 | 정보처리기사        | 한국산업인력공단     |
| 2025.03.21 | 데이터분석 준전문가 | 한국데이터산업진흥원 |
| 2025.07.11 | 빅데이터분석기사    | 한국데이터산업진흥원 |

---

## Contact

- Email: didwldnd0722@gmail.com
- GitHub: https://github.com/Peppertunacan
