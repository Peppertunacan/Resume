# 문제 해결 포트폴리오

---

## 👋 소개

안녕하세요, **양지웅**입니다.

데이터 파이프라인부터 IoT, 프론트엔드까지 End-to-End 개발 경험을 바탕으로, **문제를 정의하고 해결하는 과정**에서 가치를 만들어냅니다. 아래는 제가 프로젝트에서 마주한 문제들과 그 해결 과정입니다.

---

## 🔧 문제 해결 경험

---

### Agora - 실시간 뉴스 토픽 분석 플랫폼

> Next.js 16 + React 19 기반 프론트엔드 개발 | 2025.10 - 2025.11 | 4인 팀

<table>
  <tr>
    <td><img src="./screenshots/agora_main.png" width="400" /></td>
    <td><img src="./screenshots/agora_topic_detail.png" width="400" /></td>
  </tr>
</table>

---

#### 📌 1. SSR/CSR Hydration Mismatch 해결

**문제**

다크모드 토글 기능을 구현하던 중, 콘솔에 React Hydration 에러가 지속적으로 발생했습니다.

```
Warning: Text content did not match. Server: "light" Client: "dark"
```

SSR에서는 서버가 사용자의 시스템 테마를 알 수 없어 기본값(light)으로 렌더링하고, CSR에서는 `localStorage`나 `prefers-color-scheme`을 읽어 실제 테마(dark)로 렌더링하면서 불일치가 발생했습니다.

**해결**

클라이언트 마운트 전까지는 테마 관련 UI를 렌더링하지 않도록 `mounted` 상태를 도입했습니다.

```tsx
// ThemeToggle.tsx
const ThemeToggle = () => {
  const [mounted, setMounted] = useState(false);
  const { theme, setTheme } = useTheme();

  useEffect(() => {
    setMounted(true);
  }, []);

  // 마운트 전에는 placeholder 렌더링
  if (!mounted) {
    return <div className="w-9 h-9" />; // 레이아웃 시프트 방지
  }

  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      {theme === 'dark' ? <SunIcon /> : <MoonIcon />}
    </button>
  );
};
```

**결과**
- ✅ Hydration 에러 완전 제거
- ✅ 테마 전환 시 깜빡임 현상 해결
- ✅ 레이아웃 시프트 방지를 위한 placeholder 적용

---

#### 📌 2. 페이지별 렌더링 전략 최적화

**문제**

뉴스 플랫폼 특성상 페이지마다 요구사항이 달랐습니다:
- 메인 페이지: 트렌딩 토픽은 자주 바뀌지만, 매 요청마다 서버 렌더링은 비효율적
- 토픽 상세: SEO가 중요하고 초기 로딩 속도가 핵심
- 댓글/필터: 사용자 인터랙션이 많아 빠른 응답 필요

모든 페이지를 동일한 렌더링 방식으로 처리하면 성능 또는 SEO에서 손해를 보게 됩니다.

**해결**

페이지 특성에 따라 SSR, ISR, CSR을 전략적으로 분리했습니다.

```
┌─────────────────────────────────────────────────────────────┐
│                      렌더링 전략 매핑                        │
├─────────────────────────────────────────────────────────────┤
│  SSR (Dynamic)          │  토픽 상세, 기사 상세              │
│  → SEO 최적화           │  → 검색 엔진 인덱싱 중요            │
├─────────────────────────────────────────────────────────────┤
│  ISR (Incremental)      │  메인 페이지(60s), 토픽 목록(300s)  │
│  → 정적 + 주기적 갱신    │  → 트래픽 부하 분산                 │
├─────────────────────────────────────────────────────────────┤
│  CSR (Client)           │  댓글, 필터, 테마 토글              │
│  → 인터랙티브 기능       │  → 즉각적인 사용자 피드백           │
└─────────────────────────────────────────────────────────────┘
```

```typescript
// 메인 페이지 - ISR 적용
export const revalidate = 60; // 60초마다 재생성

// 토픽 상세 - Dynamic SSR
export const dynamic = 'force-dynamic';
```

**결과**

<img src="./screenshots/agora_lighthouse_score.png" width="500" />

- ✅ Lighthouse Performance 점수 개선
- ✅ ISR 캐싱으로 TTFB(Time To First Byte) 단축
- ✅ 서버 부하 분산

---

### MSKT - 건설현장 근로자 관리 플랫폼

> React Native + Expo 기반 Android 앱 개발 / **PM** | 2025.09 | 5인 팀

<table>
  <tr>
    <td><img src="./screenshots/mskt_main.png" width="250" /></td>
    <td><img src="./screenshots/mskt_contract.png" width="250" /></td>
    <td><img src="./screenshots/mskt_signature.png" width="250" /></td>
  </tr>
</table>

---

#### 📌 1. FCM 푸시 토큰 동시성 제어

**문제**

앱 시작 시 FCM 토큰을 서버에 등록하는 로직에서, 다음과 같은 상황이 발생했습니다:

1. 사용자가 앱을 빠르게 껐다 켬
2. 이전 토큰 삭제 요청과 새 토큰 등록 요청이 동시에 발생
3. Race condition으로 인해 토큰이 중복 등록되거나 유실

```
Timeline:
[앱 종료] → deleteToken() 시작
                    [앱 시작] → registerToken() 시작
[deleteToken 완료]
                    [registerToken 완료] → 충돌!
```

**해결**

플래그 기반 동시성 제어를 구현했습니다.

```typescript
// NotificationService.ts
class NotificationService {
  private isRegisteringToken = false;
  private isDeletingToken = false;

  async registerPushToken(): Promise<void> {
    // 이미 등록 중이거나 삭제 중이면 스킵
    if (this.isRegisteringToken || this.isDeletingToken) {
      console.log('Token operation in progress, skipping...');
      return;
    }

    try {
      this.isRegisteringToken = true;
      
      const token = await Notifications.getExpoPushTokenAsync();
      const storedToken = await SecureStore.getItemAsync('pushToken');
      
      // 동일 토큰이면 재등록 불필요
      if (storedToken === token.data) return;
      
      await api.post('/notifications/register', { token: token.data });
      await SecureStore.setItemAsync('pushToken', token.data);
      
    } finally {
      this.isRegisteringToken = false;
    }
  }

  async deletePushToken(): Promise<void> {
    if (this.isRegisteringToken || this.isDeletingToken) return;
    
    try {
      this.isDeletingToken = true;
      // ... 삭제 로직
    } finally {
      this.isDeletingToken = false;
    }
  }
}
```

**결과**
- ✅ 토큰 중복 등록/유실 문제 해결
- ✅ 불필요한 API 호출 방지
- ✅ SecureStore를 활용한 토큰 영속화로 앱 재시작 시에도 일관성 유지

---

#### 📌 2. [PM] Jira + Git + Mattermost 연동 자동화

**문제**

5인 팀에서 다음과 같은 비효율이 발생했습니다:
- 개발자가 코드 커밋 후 Jira 티켓 상태를 수동으로 변경
- PR 생성/머지 시 팀원들에게 수동으로 알림
- 스프린트 진행 상황 파악을 위해 매번 Jira 보드 확인 필요

**해결**

Jira Smart Commit + Webhook을 활용해 자동화 파이프라인을 구축했습니다.

```
┌──────────────────┐     ┌──────────────────┐      ┌──────────────────┐
│     Git 커밋     │────▶│       Jira       │────▶│    Mattermost    │
│ MSKT-123 #done   │     │   상태: Done     │      │    알림 발송      │
└──────────────────┘     └──────────────────┘      └──────────────────┘
```

**커밋 컨벤션 도입:**
```
feat: QR 스캔 기능 구현 MSKT-42
fix: 토큰 중복 등록 버그 수정 MSKT-45
```

**결과**
- ✅ 티켓 상태 업데이트 자동화 → 개발자 컨텍스트 스위칭 감소
- ✅ 스프린트 번다운 차트 실시간 반영
- ✅ PR 알림 자동화로 리뷰 대기 시간 단축

---

### SOBI - AIoT 스마트 바스켓

> Raspberry Pi + RFID + MQTT 기반 IoT 시스템 개발 / **PM** | 2025.07 - 2025.08 | 6인 팀

<table>
  <tr>
    <td><img src="screenshots/sobi_HW_info.png" width="400" /></td>
    <td><img src="screenshots/sobi_Front_UI.png" width="400" /></td>
  </tr>
</table>

<p align="center">
  <img src="screenshots/sobi_architecture_flow.png" width="700" />
</p>

---

#### 📌 1. RFID 인식 시간 6초 → 2초로 단축 (+ 정확도 유지)

**문제**

스마트 장바구니의 핵심 UX는 "상품을 넣으면 바로 화면에 반영"되는 것입니다. 
초기 구현에서는 **6초 이상**의 지연이 발생했습니다:

```
초기 상태:
RFID 태그 인식 → (4초: 2사이클 연속 감지 대기) → MQTT 발행 
→ 백엔드 처리 → DB 저장 → 프론트 폴링 → 화면 업데이트
= 총 6초 이상
```

왜 2사이클 연속 감지가 필요했나?
- 장바구니 **외부** 상품이 인식되는 것을 막기 위해 RFID 신호를 약하게 설정
- 약한 신호로 인한 불안정한 감지 → 2사이클 연속 감지해야 "확실히 있다"고 판단

사용자 입장에서 6초는 "고장났나?" 수준이었습니다.

**해결**

속도와 정확도, 두 마리 토끼를 잡기 위해 **트레이드오프를 재설계**했습니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                    속도 vs 정확도 트레이드오프                    │
├─────────────────────────────────────────────────────────────────┤
│  기존: 약한 신호 + 2사이클 연속 감지 → 느리지만 정확                │
│  변경: 강한 신호 + 1사이클 감지 + 추가 필터링 → 빠르고 정확         │
└─────────────────────────────────────────────────────────────────┘
```

**1) 감지 로직 변경: 2사이클 → 1사이클**
```python
# 기존: 2사이클 연속 감지 필요
THRESHOLD_ADD = 2  # 느림

# 변경: 1사이클만 감지되면 즉시 인식
THRESHOLD_ADD = 1  # 빠름
```

**2) 신호 강도 상향 + RSSI 임계값 필터링**
```python
# 신호를 강하게 올리되, RSSI로 외부 태그 필터링
RSSI_THRESHOLD = -50  # dBm

def should_add_to_cart(self, tag: RFIDTag) -> bool:
    # 강한 신호(가까운 거리)만 인식
    if tag.rssi < RSSI_THRESHOLD:
        return False  # 장바구니 외부로 판단
    return True
```

**3) 물리적 차폐: 은박 호일 적용**
```
┌─────────────────────────────┐
│  ░░░░░ 은박 호일 차폐 ░░░░░  │  ← 외부 신호 차단
│  ┌─────────────────────┐    │
│  │   장바구니 내부      │    │
│  │   상품 A (인식 O)    │    │
│  └─────────────────────┘    │
│  ░░░░░░░░░░░░░░░░░░░░░░░░░  │
└─────────────────────────────┘
        
    상품 B (인식 X) ← 호일 + RSSI 필터링으로 차단
```

**4) 프론트엔드: 폴링 → SSE 푸시**
```python
# mqtt_controller.py - 최적화된 RFID 처리
def on_tag_detected(self, tag: RFIDTag):
    # RSSI 필터링
    if tag.rssi < RSSI_THRESHOLD:
        return
    
    # 중복 감지 필터링 (debounce)
    if self.cart_manager.is_recently_detected(tag.epc):
        return
    
    # 즉시 MQTT 발행 (QoS 1)
    self.mqtt_client.publish(
        topic=f"basket/{self.basket_id}/update",
        payload=json.dumps({"action": "add", "epc": tag.epc}),
        qos=1  # At least once delivery
    )
```

```
최적화 후:
RFID 인식 → (1.5초: 1사이클) → RSSI 필터 → MQTT 발행 
→ (0.2초) → 백엔드 처리 → (0.2초) → SSE 푸시 → 화면 업데이트
= 총 2초 이내
```

**결과**
- ✅ 응답 시간 **6초 → 1~2초 이내** (약 70% 단축)
- ✅ RSSI 필터링 + 물리적 차폐로 **정확도 유지**
- ✅ 사용자 테스트에서 "즉각적인 반응" 피드백

---

#### 📌 2. [PM] HW/SW 팀 간 협업 프로세스 구축

**문제**

6인 팀이 HW(Raspberry Pi, RFID)와 SW(Backend, Frontend, AI)로 나뉘어 개발하면서:
- HW팀: "백엔드 API가 아직 안 돼서 테스트를 못 해요"
- SW팀: "MQTT 메시지 포맷이 자꾸 바뀌어요"
- 일정 지연 및 통합 테스트에서 대규모 버그 발생

**해결**

**1) 인터페이스 문서 선행 작성**
```
Notion 문서:
├── MQTT 토픽 명세 (topic, payload 스키마)
├── 하드웨어 스펙 (RFID 리더기, LCD, GPIO 핀맵)
├── API 명세 (백엔드 ↔ 프론트엔드)
└── 트러블슈팅 가이드
```

**2) Mock 서버/클라이언트 제공**
- HW팀: MQTT Mock Publisher 제공 → 백엔드 없이 테스트 가능
- SW팀: MQTT Simulator 제공 → 실제 하드웨어 없이 개발 가능

**3) 일일 스탠드업에서 "블로커" 중심 논의**
- 각 팀의 블로커를 먼저 공유
- 블로커 해결이 당일 최우선 과제

**결과**
- ✅ 통합 테스트 첫 시도에서 주요 기능 정상 동작
- ✅ 일정 내 프로젝트 완료
- ✅ 팀원 피드백: "역할 분담과 소통이 명확했다"

---

### AI News Curation - 뉴스 큐레이션 서비스

> Full Stack (Django + Vue.js) + Data Engineering | 2025.04 - 2025.05 | 2인 팀

<table>
  <tr>
    <td><img src="screenshots/ai_news_curation_Home.png" width="400" /></td>
    <td><img src="screenshots/ai_news_curation_Dashboard.png" width="400" /></td>
  </tr>
</table>

---

#### 📌 1. Elasticsearch 한글 검색 + 자동완성 최적화

**문제**

뉴스 검색에서 한글 특성으로 인한 여러 문제가 발생했습니다:
- "경제" 검색 시 "경제적", "경제학" 등 파생어 미검색
- 오타 입력 시 ("겅제") 검색 결과 없음
- 자동완성이 너무 느리거나 관련 없는 결과 반환

**해결**

**1) Nori 형태소 분석기 + Edge N-gram 조합**
```python
# 검색 쿼리 - 다중 필드 + 가중치 부여
body = {
    "query": {
        "bool": {
            "should": [
                {  # 제목 edge_ngram (접두어 매칭)
                    "match": {
                        "title.edge_ngram": {
                            "query": query,
                            "boost": 2.0
                        }
                    }
                },
                {  # 본문 edge_ngram
                    "match": {
                        "content.edge_ngram": query
                    }
                },
                {  # 제목 기본 매칭 (형태소 분석)
                    "match": {
                        "title": {
                            "query": query,
                            "boost": 1.0
                        }
                    }
                }
            ],
            "minimum_should_match": 1
        }
    },
    "highlight": {
        "fields": {
            "title": {},
            "content": {}
        }
    }
}
```

**2) 검색어 길이별 동적 Fuzziness 적용**
```python
# autocomplete_articles 함수
def get_fuzziness(query):
    if len(query) <= 1:
        return 0  # 1글자: 정확 매칭만
    elif len(query) == 2:
        return 1  # 2글자: 1글자 오차 허용
    else:
        return min(2, len(query) // 3)  # 3글자+: 최대 2

# 자동완성 쿼리 - 4단계 우선순위
body = {
    "query": {
        "bool": {
            "should": [
                {"match_phrase_prefix": {"title.nori": {"query": q, "boost": 4.0}}},  # 정확한 구문
                {"match": {"title.nori": {"query": q, "boost": 3.0}}},               # 형태소 매칭
                {"match": {"title.edge_ngram": {"query": q, "boost": 2.5}}},         # 접두어 매칭
                {"fuzzy": {"title.nori": {"value": q, "fuzziness": fuzziness_value, "boost": 2.0}}}  # 오타 교정
            ]
        }
    },
    "size": 10
}
```

```
┌─────────────────────────────────────────────────────────────┐
│                  자동완성 우선순위 전략                       │
├─────────────────────────────────────────────────────────────┤
│  1순위 (boost 4.0)  │  match_phrase_prefix  │  "경제 성장"   │
│  2순위 (boost 3.0)  │  nori 형태소 매칭      │  "경제적"      │
│  3순위 (boost 2.5)  │  edge_ngram           │  "경제..."     │
│  4순위 (boost 2.0)  │  fuzzy (오타 교정)     │  "겅제" → 경제 │
└─────────────────────────────────────────────────────────────┘
```

**결과**
- ✅ 한글 조사/어미 처리로 검색 정확도 향상
- ✅ 동적 fuzziness로 오타 교정 + 과도한 매칭 방지
- ✅ 실시간 자동완성 응답 속도 < 100ms

---

#### 📌 2. pgvector 기반 개인화 뉴스 추천

**문제**

단순 조회 기록 기반 추천의 한계:
- 실수로 클릭한 기사도 관심사로 반영
- 좋아하는 기사와 그냥 본 기사의 구분 없음
- 추천 결과가 사용자 취향을 제대로 반영하지 못함

**해결**

**좋아요 가중치 + 벡터 유사도 기반 추천 시스템**을 구현했습니다.

```python
# recommend 함수 - 가중치 기반 사용자 벡터 생성
@api_view(['GET'])
@permission_classes([IsAuthenticated])
def recommend(request):
    user = request.user
    interactions = UserArticleInteraction.objects.filter(user=user)
    
    # 1. 상호작용별 가중치 계산
    article_embeddings = []
    weights = []
    
    for interaction in interactions:
        article_embeddings.append(interaction.article.embedding)
        # 핵심: 좋아요는 가중치 2, 조회만은 가중치 1
        weight = 2 if interaction.liked else 1
        weights.append(weight)
    
    # 2. 가중치 적용된 임베딩 합산
    weighted_embeddings = article_embeddings * weights[:, np.newaxis]
    user_embedding = np.sum(weighted_embeddings, axis=0)
    
    # 3. 정규화 (단위 벡터로 변환)
    norm = np.linalg.norm(user_embedding)
    if norm > 0:
        user_embedding /= norm
    
    # 4. 코사인 거리로 유사 기사 검색 (pgvector)
    similar_articles = Article.objects.annotate(
        similarity=CosineDistance('embedding', user_embedding)
    ).filter(
        ~Exists(interaction_exists)  # 안 읽은 기사만
    ).order_by('similarity')[:5]
    
    return Response({
        'read_articles': ...,    # 읽은 기사 중 유사한 것
        'unread_articles': ...,  # 안 읽은 기사 중 유사한 것 (신규 추천)
        'liked_articles': ...    # 좋아요 기사 중 유사한 것
    })
```

```
┌─────────────────────────────────────────────────────────────┐
│              사용자 벡터 생성 과정                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   기사 A (조회) ──→ embedding × 1 ─┐                         │
│   기사 B (좋아요) ──→ embedding × 2 ─┼─→ 합산 → 정규화        │
│   기사 C (좋아요) ──→ embedding × 2 ─┘      ↓                │
│                                        사용자 벡터           │
│                                            ↓                │
│                              pgvector CosineDistance        │
│                                            ↓                │
│                                    유사 기사 Top 5           │
└─────────────────────────────────────────────────────────────┘
```

**결과**
- ✅ 좋아요 기사에 2배 가중치 → 명확한 관심사 반영
- ✅ pgvector의 벡터 인덱스로 밀리초 단위 유사도 검색
- ✅ 읽은/안 읽은/좋아요 기사 분리 제공으로 다양한 추천

---

## 📞 Contact

- 📧 Email: didwldnd0722@gmail.com
- 🐙 GitHub: https://github.com/Peppertunacan

---

<div align="center">

*Thank you for reading!*

</div>
