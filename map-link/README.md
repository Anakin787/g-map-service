# 🚗 박군투어 요금 계산기

파타야 지역 박군투어 서비스를 위한 지능형 요금 계산 웹 애플리케이션입니다.

## ✨ 주요 기능

### 🗺️ 인터랙티브 지도 기반 요금 계산
- **Google Maps 통합**: 실시간 지도 인터페이스
- **구역별 요금 시스템**: 초록/노랑/빨강 구역으로 구분된 추가요금 체계
- **주소 검색**: Google Places API를 활용한 자동완성 주소 검색
- **폴리곤 클릭**: 지도에서 직접 위치 선택 가능

### 💰 스마트 요금 계산 시스템
- **다양한 출발지/도착지**: 공항, 방콕, 파타야 등 주요 지역 지원
- **3가지 차종**: 세단, SUV, 밴 선택 가능
- **실시간 요금 계산**: 기본요금 + 구역별 추가요금 자동 계산
- **요금 요약 박스**: 계산 완료 후 한눈에 보는 요금 정보

### 📱 반응형 디자인
- **PC 최적화**: 2열 레이아웃으로 효율적인 화면 구성
- **모바일 친화적**: 터치 인터페이스 완벽 지원
- **4단계 브레이크포인트**: 데스크톱 > 태블릿 > 모바일 > 초소형 대응

## 🛠️ 기술 스택

### Frontend
- **Vue 3** - Composition API 활용
- **TypeScript** - 타입 안전성 보장
- **Vite** - 빠른 개발 환경

### Maps & APIs
- **Google Maps JavaScript API** - 지도 렌더링
- **Google Places API** - 주소 검색 자동완성
- **@googlemaps/js-api-loader** - 모던한 API 로딩

### Styling
- **CSS3** - 모던 CSS 기능 활용
- **Flexbox** - 반응형 레이아웃
- **CSS Grid** - 복잡한 레이아웃 구성
- **Media Queries** - 완벽한 반응형 디자인

## 🚀 빠른 시작

### 사전 요구사항
- Node.js 16.0 이상
- npm 또는 yarn
- Google Maps API 키

### 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 프로덕션 빌드
npm run build

# 타입 체크
npm run type-check
```

### 환경 설정
Google Maps API 키를 `src/App.vue`의 `apiKey` 부분에 설정해주세요:

```typescript
const loader = new Loader({
  apiKey: "YOUR_GOOGLE_MAPS_API_KEY_HERE",
  version: "weekly",
  libraries: ["places", "drawing", "geometry"]
});
```

## 📋 요금표

### 기본 요금 (바트)
| 출발지 | 도착지 | 세단 | SUV | 밴 |
|--------|--------|------|-----|-----|
| 수완나품 공항 | 방콕 | 600 | 700 | 1,000 |
| 수완나품 공항 | 파타야 | 1,000 | 1,200 | 1,700 |
| 방콕 | 파타야 | 1,300 | 1,500 | 1,900 |
| ... | ... | ... | ... | ... |

### 구역별 추가요금
- 🟢 **초록 구역**: 추가요금 없음
- 🟡 **노랑 구역**: +100바트
- 🔴 **빨강 구역**: +200바트

## 🎨 UI/UX 특징

### 사용자 중심 설계
- **직관적인 워크플로우**: 주소 검색 → 확인 → 요금 계산
- **시각적 피드백**: 구역별 색상 코딩 및 애니메이션
- **접근성**: 키보드 네비게이션 및 스크린 리더 지원

### 모던한 디자인
- **그라데이션 효과**: 시각적 깊이감 제공
- **블러 효과**: 반투명 요소로 모던한 느낌
- **마이크로 인터랙션**: 호버 및 클릭 피드백

## 📱 반응형 브레이크포인트

- **데스크톱** (769px+): 2열 레이아웃, 최대 기능 제공
- **태블릿** (768px 이하): 1열 레이아웃, 터치 최적화
- **모바일** (480px 이하): 컴팩트 디자인
- **초소형** (360px 이하): 핵심 기능 중심

## 🔧 개발 환경 설정

### 추천 IDE
- [VSCode](https://code.visualstudio.com/)
- [Volar Extension](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (Vetur 비활성화)

### TypeScript 지원
Vue 3 + TypeScript 환경에서 완벽한 타입 추론을 위해 `vue-tsc`를 사용합니다.

## 📈 성능 최적화

- **shallowRef**: Google Maps 객체 최적화
- **computed**: 요금 계산 결과 캐싱
- **이벤트 최적화**: 불필요한 리렌더링 방지
- **코드 분할**: 필요한 라이브러리만 로드

## 🐛 문제 해결

### 일반적인 문제
1. **지도가 로드되지 않는 경우**: Google Maps API 키 확인
2. **주소 검색이 작동하지 않는 경우**: Places API 활성화 확인
3. **모바일에서 터치가 반응하지 않는 경우**: 브라우저 캐시 삭제

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다.

## 🤝 기여하기

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📞 연락처

박군투어 요금 계산기 프로젝트에 대한 문의사항이 있으시면 언제든 연락주세요.

---

**박군투어와 함께하는 편리한 파타야 여행! 🌴**
