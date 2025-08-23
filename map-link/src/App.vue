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
    <!-- 안내 및 결과 표시 영역 -->
    <div class="info-box">
      <p v-if="!polygon">1. 지도 좌측 상단의 그리기 도구를 사용하여 폴리곤을 그려주세요.</p>
      <p v-else-if="!selectedPosition">2. 생성된 폴리곤 내부를 클릭하여 위치를 선택하세요.</p>
      <div v-else>
        <h3 class="result-title">산출 결과</h3>
        <p><strong>선택된 위치:</strong></p>
        <p>위도: {{ selectedPosition.lat.toFixed(6) }}</p>
        <p>경도: {{ selectedPosition.lng.toFixed(6) }}</p>
        <p class="result-value"><strong>계산된 값:</strong> {{ calculatedValue }}</p>
        <button @click="reset" class="reset-button">초기화</button>
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
const drawingManager = shallowRef<any | null>(null);
const polygon = shallowRef<any | null>(null);
const marker = shallowRef<any | null>(null);

// --- 상태 관리 ---
const selectedPosition = ref<Position | null>(null);
const calculatedValue = ref<string>('');

// --- 컴포넌트 마운트 시 실행될 로직 ---
onMounted(async () => {
  try {
    // Google Maps API Loader 설정
    const loader = new Loader({
      apiKey: "AIzaSyB_XSMccZRe5dWeqTBzBhJrxmaM05Z-42M",
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

  // 기본 지도 옵션 (서울 중심)
  const mapOptions: any = {
    center: { lat: 37.5665, lng: 126.9780 },
    zoom: 12,
    streetViewControl: false,
    mapTypeControl: false,
  };

  // 지도 인스턴스 생성
  map.value = new google.maps.Map(mapDiv.value, mapOptions);

  // 검색 및 그리기 기능 초기화
  initSearchBox();
  initDrawingManager();

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

// --- 그리기 관리자 초기화 ---
const initDrawingManager = () => {
  if (!map.value) return;

  // DrawingManager 인스턴스 생성
  drawingManager.value = new google.maps.drawing.DrawingManager({
    drawingMode: google.maps.drawing.OverlayType.POLYGON,
    drawingControl: true,
    drawingControlOptions: {
      position: google.maps.ControlPosition.TOP_LEFT,
      drawingModes: [
        google.maps.drawing.OverlayType.POLYGON,
      ],
    },
    polygonOptions: {
      fillColor: '#4285F4',
      fillOpacity: 0.3,
      strokeWeight: 2,
      strokeColor: '#4285F4',
      clickable: false,
      editable: true,
      zIndex: 1,
    },
  });

  // 지도에 DrawingManager 연결
  drawingManager.value.setMap(map.value);

  // 폴리곤 그리기 완료 시 이벤트 처리
  google.maps.event.addListener(drawingManager.value, 'polygoncomplete', (newPolygon: any) => {
    // 기존 폴리곤이 있다면 삭제
    if (polygon.value) {
      polygon.value.setMap(null);
    }
    polygon.value = newPolygon;

    // 그리기 모드 비활성화
    drawingManager.value?.setDrawingMode(null);
  });
};

// --- 지도 클릭 이벤트 핸들러 ---
const handleMapClick = (event: any) => {
  if (!event.latLng || !polygon.value) return;

  // 클릭한 위치가 폴리곤 내부에 있는지 확인
  const isInside = google.maps.geometry.poly.containsLocation(event.latLng, polygon.value);

  if (isInside) {
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
      title: '선택된 위치',
    });

    // 핵심 로직: 특정 값 산출 함수 호출
    calculateSpecificValue(selectedPosition.value);

  } else {
    alert('폴리곤 내부를 클릭해주세요.');
  }
};

/**
 * @description 선택된 위치를 기준으로 특정 값을 계산하는 함수
 * (이곳에 실제 비즈니스 로직을 구현합니다)
 * @param position 위도, 경도 정보
 */
const calculateSpecificValue = (position: Position) => {
  // --- 예시 로직 ---
  // 실제 구현 시에는 이 부분에 API 호출, 복잡한 연산 등
  // 비즈니스 요구사항에 맞는 코드를 작성합니다.
  // 여기서는 위도와 경도의 합을 예시로 보여줍니다.
  const result = position.lat + position.lng;
  calculatedValue.value = `계산 결과: ${result.toFixed(4)} (예시 값)`;
  // --- 예시 로직 끝 ---
};

// --- 초기화 함수 ---
const reset = () => {
  if (polygon.value) {
    polygon.value.setMap(null);
    polygon.value = null;
  }
  if (marker.value) {
    marker.value.setMap(null);
    marker.value = null;
  }
  selectedPosition.value = null;
  calculatedValue.value = '';
  drawingManager.value?.setDrawingMode(google.maps.drawing.OverlayType.POLYGON);
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
</style>
