<template>
  <div class="map-container">
    <!-- 검색 입력창 -->
    <input
      ref="searchInput"
      type="text"
      placeholder="장소를 검색하세요..."
      class="search-input"
    />
    <!-- 구글 지도가 렌더링될 컨테이너 -->
    <div ref="mapDiv" class="map-canvas"></div>

    <!-- 요금 안내 영역 -->
    <div class="price-info-box">
      <h3 class="price-title">지역별 요금 안내</h3>
      <div class="price-list">
        <div class="price-item green">
          <div class="color-indicator green-indicator"></div>
          <span class="zone-name">초록 지역</span>
          <span class="price-info">추가요금 없음</span>
        </div>
        <div class="price-item yellow">
          <div class="color-indicator yellow-indicator"></div>
          <span class="zone-name">노랑 지역</span>
          <span class="price-info">추가 100바트</span>
        </div>
        <div class="price-item red">
          <div class="color-indicator red-indicator"></div>
          <span class="zone-name">빨강 지역</span>
          <span class="price-info">추가 200바트</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, shallowRef } from 'vue';
// @ts-ignore
import { Loader } from '@googlemaps/js-api-loader';

// --- 타입 정의 ---
declare var google: any;
interface Position {
  lat: number;
  lng: number;
}

// --- DOM 엘리먼트 참조 ---
const mapDiv = ref<HTMLElement | null>(null);
const searchInput = ref<HTMLInputElement | null>(null);

// --- 구글맵 관련 객체 (shallowRef 사용 권장) ---
// 구글맵 객체는 내부 상태가 복잡하고 반응형으로 추적할 필요가 없으므로
// shallowRef를 사용하여 성능상의 이점을 얻습니다.
const map = shallowRef<any | null>(null);
const marker = shallowRef<any | null>(null);
const fixedPolygons = shallowRef<any[]>([]);

// --- 상태 관리 ---
const selectedPosition = ref<Position | null>(null);

// --- 컴포넌트 마운트 시 실행될 로직 ---
onMounted(async () => {
  try {
    // Google Maps API Loader 설정
    const loader = new Loader({
      apiKey: "AIzaSyAdlN2ToDxAyPUVnaEul0wbn4ZUvC5mK8k",
      version: "weekly",
      libraries: ["places", "drawing", "geometry"]
    });

    // Google Maps API 로드
    await loader.load();
    
    // 지도 초기화
    initMap();
  } catch (error) {
    console.error('Google Maps API 로드에 실패했습니다:', error);
    alert('지도를 불러오는데 실패했습니다. API 키를 확인해주세요.');
  }
});

// --- 지도 초기화 함수 ---
const initMap = () => {
  if (!mapDiv.value) return;

  // 기본 지도 옵션 (파타야 중심)
  const mapOptions: any = {
    center: { lat: 12.9203, lng: 100.9706 },
    zoom: 11,
    streetViewControl: false,
    mapTypeControl: false,
  };

  // 지도 인스턴스 생성
  map.value = new google.maps.Map(mapDiv.value, mapOptions);

  // 검색 기능 초기화 및 고정 폴리곤 생성
  initSearchBox();
  createFixedPolygons();

  // 지도 클릭 이벤트 리스너 추가
  map.value.addListener('click', handleMapClick);
};

// --- 장소 검색 기능 초기화 ---
const initSearchBox = () => {
  if (!searchInput.value || !map.value) return;

  // Autocomplete 기능 활성화
  const autocomplete = new google.maps.places.Autocomplete(searchInput.value, {
    fields: ["geometry", "name"],
    types: ["establishment"],
  });

  // 지도 범위에 따라 검색 결과 우선순위 조정
  autocomplete.bindTo("bounds", map.value);

  // 장소 선택 시 이벤트 처리
  autocomplete.addListener('place_changed', () => {
    const place = autocomplete.getPlace();

    if (place.geometry && place.geometry.location && map.value) {
      // 선택된 장소로 지도 이동 및 확대
      map.value.setCenter(place.geometry.location);
      map.value.setZoom(17);
    } else {
      console.log("선택된 장소에 대한 위치 정보가 없습니다.");
    }
  });
};



// --- 고정 폴리곤 생성 함수 ---
const createFixedPolygons = () => {
  if (!map.value) return;

  // 초록 폴리곤 좌표
  const greenCoordinates = [
    { lat: 12.884707, lng: 100.878598 },
    { lat: 12.910811, lng: 100.855596 },
    { lat: 12.930554, lng: 100.862119 },
    { lat: 12.926539, lng: 100.871045 },
    { lat: 12.940258, lng: 100.883062 },
    { lat: 12.971374, lng: 100.882032 },
    { lat: 12.977730, lng: 100.891645 },
    { lat: 12.973047, lng: 100.899198 },
    { lat: 13.003490, lng: 100.924604 },
    { lat: 13.042960, lng: 100.915677 },
    { lat: 13.052994, lng: 100.995328 },
    { lat: 13.011184, lng: 100.996358 },
    { lat: 12.965017, lng: 100.979707 },
    { lat: 12.952805, lng: 100.971124 },
    { lat: 12.950128, lng: 100.957734 },
    { lat: 12.907464, lng: 100.952413 },
    { lat: 12.907464, lng: 100.952413 },
    { lat: 12.903447, lng: 100.954227 },
    { lat: 12.886635, lng: 100.968878 },
  ];

  // 노랑 폴리곤 좌표
  const yellowCoordinates = [
    { lat: 12.885417, lng: 100.879598 },
    { lat: 12.822490, lng: 100.910154 },
    { lat: 12.826340, lng: 101.002508 },
    { lat: 12.861319, lng: 101.024309 },
    { lat: 12.882405, lng: 101.023794 },
    { lat: 12.908342, lng: 101.028943 },
    { lat: 12.928420, lng: 101.003023 },
    { lat: 12.965560, lng: 100.980020 },
    { lat: 12.953348, lng: 100.972639 },
    { lat: 12.946154, lng: 100.974355 },
    { lat: 12.942975, lng: 100.972467 },
    { lat: 12.938793, lng: 100.972639 },
    { lat: 12.936952, lng: 100.967660 },
    { lat: 12.928754, lng: 100.965600 },
    { lat: 12.923902, lng: 100.971780 },
    { lat: 12.919385, lng: 100.970235 },
    { lat: 12.921058, lng: 100.965944 },
    { lat: 12.919217, lng: 100.953412 },
    { lat: 12.908844, lng: 100.953069 },
    { lat: 12.903824, lng: 100.954957 },
    { lat: 12.886421, lng: 100.969549 }
  ];

  // 빨강 폴리곤 좌표
  const redCoordinates = [
    { lat: 12.821953, lng: 100.910719 },
    { lat: 12.806463, lng: 100.911902 },
    { lat: 12.769969, lng: 100.898169 },
    { lat: 12.758920, lng: 100.991553 },
    { lat: 12.777001, lng: 100.992240 },
    { lat: 12.775996, lng: 101.012496 },
    { lat: 12.789054, lng: 101.011809 },
    { lat: 12.798763, lng: 101.017302 },
    { lat: 12.803450, lng: 101.027602 },
    { lat: 12.809141, lng: 101.031722 },
    { lat: 12.813494, lng: 101.037215 },
    { lat: 12.812489, lng: 101.044768 },
    { lat: 12.815251, lng: 101.049489 },
    { lat: 12.818348, lng: 101.048545 },
    { lat: 12.824541, lng: 101.054038 },
    { lat: 12.835839, lng: 101.063479 },
    { lat: 12.841278, lng: 101.061848 },
    { lat: 12.852157, lng: 101.064252 },
    { lat: 12.853496, lng: 101.061505 },
    { lat: 12.858684, lng: 101.060904 },
    { lat: 12.865462, lng: 101.065882 },
    { lat: 12.892320, lng: 101.072749 },
    { lat: 12.895834, lng: 101.078585 },
    { lat: 12.900352, lng: 101.079100 },
    { lat: 12.903699, lng: 101.075152 },
    { lat: 12.907714, lng: 101.078242 },
    { lat: 12.915913, lng: 101.088027 },
    { lat: 12.931139, lng: 101.085624 },
    { lat: 12.959245, lng: 101.090087 },
    { lat: 12.970620, lng: 101.101073 },
    { lat: 12.976475, lng: 101.093177 },
    { lat: 12.982162, lng: 101.091975 },
    { lat: 12.989522, lng: 101.093348 },
    { lat: 13.025926, lng: 101.072736 },
    { lat: 13.019236, lng: 101.067414 },
    { lat: 13.006358, lng: 101.066728 },
    { lat: 12.920707, lng: 101.032910 },
    { lat: 12.882640, lng: 101.023726 },
    { lat: 12.862475, lng: 101.024413 },
    { lat: 12.823900, lng: 101.001271 },
  ];

  // 초록 폴리곤 생성
  const greenPolygon = new google.maps.Polygon({
    paths: greenCoordinates,
    strokeColor: '#00FF00',
    strokeOpacity: 0.8,
    strokeWeight: 2,
    fillColor: '#00FF00',
    fillOpacity: 0.35,
  });
  greenPolygon.setMap(map.value);

  // 노랑 폴리곤 생성
  const yellowPolygon = new google.maps.Polygon({
    paths: yellowCoordinates,
    strokeColor: '#FFFF00',
    strokeOpacity: 0.8,
    strokeWeight: 2,
    fillColor: '#FFFF00',
    fillOpacity: 0.35,
  });
  yellowPolygon.setMap(map.value);

  // 빨강 폴리곤 생성
  const redPolygon = new google.maps.Polygon({
    paths: redCoordinates,
    strokeColor: '#FF0000',
    strokeOpacity: 0.8,
    strokeWeight: 2,
    fillColor: '#FF0000',
    fillOpacity: 0.35,
  });
  redPolygon.setMap(map.value);

  // 폴리곤들을 배열에 저장
  fixedPolygons.value = [greenPolygon, yellowPolygon, redPolygon];
};

// --- 지도 클릭 이벤트 핸들러 ---
const handleMapClick = (event: any) => {
  if (!event.latLng || fixedPolygons.value.length === 0) return;

  // 클릭한 위치가 어떤 폴리곤 내부에 있는지 확인
  let clickedPolygon = null;
  let polygonColor = '';
  
  for (const polygon of fixedPolygons.value) {
    const isInside = google.maps.geometry.poly.containsLocation(event.latLng, polygon);
    if (isInside) {
      clickedPolygon = polygon;
      // 폴리곤 색깔 구분
      const fillColor = polygon.get('fillColor');
      if (fillColor === '#00FF00') polygonColor = '초록';
      else if (fillColor === '#FFFF00') polygonColor = '노랑';
      else if (fillColor === '#FF0000') polygonColor = '빨강';
      break;
    }
  }

  if (clickedPolygon) {
    const lat = event.latLng.lat();
    const lng = event.latLng.lng();
    selectedPosition.value = { lat, lng };

    // 기존 마커가 있다면 삭제
    if (marker.value) {
      marker.value.setMap(null);
    }

    // 새 위치에 마커 생성
    marker.value = new google.maps.Marker({
      position: event.latLng,
      map: map.value,
      title: `선택된 위치 (${polygonColor} 폴리곤)`,
    });

    // 핵심 로직: 특정 값 산출 함수 호출
    calculateSpecificValue(selectedPosition.value, polygonColor);

  } else {
    alert('폴리곤 내부를 클릭해주세요.');
  }
};

/**
 * @description 선택된 위치를 기준으로 특정 값을 계산하는 함수
 * (이곳에 실제 비즈니스 로직을 구현합니다)
 * @param position 위도, 경도 정보
 * @param polygonColor 클릭된 폴리곤 색깔
 */
const calculateSpecificValue = (position: Position, polygonColor: string) => {
  // --- 예시 로직 ---
  // 실제 구현 시에는 이 부분에 API 호출, 복잡한 연산 등
  // 비즈니스 요구사항에 맞는 코드를 작성합니다.
  // 여기서는 위도와 경도의 합을 예시로 보여줍니다.
  // --- 예시 로직 끝 ---
};

// --- 초기화 함수 ---
const reset = () => {
  // 마커만 제거 (폴리곤은 고정이므로 유지)
  if (marker.value) {
    marker.value.setMap(null);
    marker.value = null;
  }
  selectedPosition.value = null;
};

</script>

<style scoped>
.map-container {
  position: relative;
  width: 100%;
  height: 100vh; /* 전체 화면 높이 */
  font-family: 'Inter', sans-serif;
}

.map-canvas {
  width: 100%;
  height: 100%;
}

.search-input {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: 90%;
  max-width: 400px;
  padding: 12px 18px;
  border: 1px solid #ccc;
  border-radius: 25px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  font-size: 16px;
  z-index: 10;
  outline: none;
  transition: box-shadow 0.3s ease;
}
.search-input:focus {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

/* 검색창 반응형 */
@media (max-width: 768px) {
  .search-input {
    top: 15px;
    width: 85%;
    max-width: 350px;
    padding: 10px 16px;
    font-size: 15px;
  }
}

@media (max-width: 480px) {
  .search-input {
    top: 12px;
    width: 82%;
    max-width: 300px;
    padding: 8px 14px;
    font-size: 14px;
  }
}

@media (max-width: 360px) {
  .search-input {
    top: 10px;
    width: 80%;
    max-width: 250px;
    padding: 6px 12px;
    font-size: 13px;
  }
}

.info-box {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 10;
  width: 90%;
  max-width: 400px;
  text-align: center;
}

.result-title {
  margin-top: 0;
  color: #333;
}

.result-value {
  background-color: #f0f4ff;
  color: #4285F4;
  padding: 10px;
  border-radius: 8px;
  margin-top: 10px;
  font-weight: bold;
}

.reset-button {
  margin-top: 15px;
  padding: 10px 20px;
  background-color: #4285F4;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
}
.reset-button:hover {
  background-color: #357ae8;
}

.price-info-box {
  position: absolute;
  top: 80px;
  right: 20px;
  background-color: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 10;
  width: 280px;
}

.price-title {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
  font-size: 16px;
  text-align: center;
  font-weight: 600;
}

.price-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.price-item {
  display: flex;
  align-items: center;
  padding: 12px;
  border-radius: 8px;
  background-color: #f8f9fa;
  border-left: 4px solid transparent;
}

.price-item.green {
  border-left-color: #00FF00;
}

.price-item.yellow {
  border-left-color: #FFFF00;
}

.price-item.red {
  border-left-color: #FF0000;
}

.color-indicator {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  margin-right: 12px;
  border: 2px solid #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.green-indicator {
  background-color: #00FF00;
}

.yellow-indicator {
  background-color: #FFFF00;
}

.red-indicator {
  background-color: #FF0000;
}

.zone-name {
  font-weight: 600;
  color: #333;
  margin-right: auto;
  font-size: 14px;
}

.price-info {
  font-weight: 500;
  color: #666;
  font-size: 13px;
}

/* 태블릿 환경 (768px 이하) */
@media (max-width: 768px) {
  .price-info-box {
    top: 60px;
    right: 15px;
    width: 240px;
    padding: 16px;
  }

  .price-title {
    font-size: 14px;
    margin-bottom: 12px;
  }

  .price-list {
    gap: 10px;
  }

  .price-item {
    padding: 10px;
    border-left-width: 3px;
  }

  .color-indicator {
    width: 14px;
    height: 14px;
    margin-right: 10px;
  }

  .zone-name {
    font-size: 13px;
  }

  .price-info {
    font-size: 12px;
  }
}

/* 모바일 환경 (480px 이하) */
@media (max-width: 480px) {
  .price-info-box {
    top: 50px;
    right: 10px;
    width: 200px;
    padding: 12px;
    border-radius: 8px;
  }

  .price-title {
    font-size: 13px;
    margin-bottom: 10px;
  }

  .price-list {
    gap: 8px;
  }

  .price-item {
    padding: 8px;
    border-radius: 6px;
    border-left-width: 3px;
  }

  .color-indicator {
    width: 12px;
    height: 12px;
    margin-right: 8px;
    border-width: 1px;
  }

  .zone-name {
    font-size: 12px;
  }

  .price-info {
    font-size: 11px;
  }
}

/* 초소형 모바일 환경 (360px 이하) */
@media (max-width: 360px) {
  .price-info-box {
    top: 45px;
    right: 8px;
    width: 180px;
    padding: 10px;
  }

  .price-title {
    font-size: 12px;
    margin-bottom: 8px;
  }

  .price-list {
    gap: 6px;
  }

  .price-item {
    padding: 6px;
    border-left-width: 2px;
  }

  .color-indicator {
    width: 10px;
    height: 10px;
    margin-right: 6px;
  }

  .zone-name {
    font-size: 11px;
  }

  .price-info {
    font-size: 10px;
  }
}
</style>
