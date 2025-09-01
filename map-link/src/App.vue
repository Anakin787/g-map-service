<template>
  <div class="map-container">
    <!-- 검색 입력창 -->
    <input
      ref="searchInput"
      type="text"
      placeholder="주소를 검색하면 자동으로 요금이 계산됩니다..."
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

    <!-- 요금 계산 버튼 -->
    <button 
      class="fare-calculator-btn"
      @click="showFareDialog = true"
    >
      💰 요금 계산하기
    </button>

    <!-- 요금 계산 다이얼로그 -->
    <div v-if="showFareDialog" class="dialog-overlay" @click="closeFareDialog">
      <div class="fare-dialog" @click.stop>
        <div class="dialog-header">
          <h2>박군투어 요금 계산기</h2>
          <button class="close-btn" @click="closeFareDialog">×</button>
        </div>
        
        <div class="dialog-content">
          <!-- 검색된 주소 정보 표시 -->
          <div v-if="searchedAddress" class="searched-address-info">
            <div class="address-header">
              <span class="address-icon">📍</span>
              <h3>검색된 주소</h3>
            </div>
            <div class="address-card">
              <p class="address-text">{{ searchedAddress }}</p>
              <div class="zone-badge" :class="selectedZone">
                <span class="zone-indicator" :class="selectedZone"></span>
                {{ selectedZone }} 구역 (+{{ zoneExtraFee[selectedZone] }}바트)
              </div>
            </div>
          </div>

          <!-- 출발지 선택 -->
          <div class="form-group">
            <label for="departure">출발지</label>
            <select id="departure" v-model="selectedDeparture">
              <option value="">출발지를 선택하세요</option>
              <option v-for="location in departureLocations" :key="location" :value="location">
                {{ location }}
              </option>
            </select>
          </div>

          <!-- 도착지 선택 -->
          <div class="form-group">
            <label for="destination">도착지</label>
            <select id="destination" v-model="selectedDestination">
              <option value="">도착지를 선택하세요</option>
              <option v-for="location in availableDestinations" :key="location" :value="location">
                {{ location }}
              </option>
            </select>
          </div>

          <!-- 차종 선택 -->
          <div class="form-group">
            <label>차종</label>
            <div class="vehicle-options">
              <label v-for="(name, type) in vehicleTypes" :key="type" class="vehicle-option">
                <input type="radio" :value="type" v-model="selectedVehicle" />
                <span>{{ name }}</span>
              </label>
            </div>
          </div>

          <!-- 요금 계산 결과 -->
          <div v-if="calculatedFare" class="fare-result">
            <h3>요금 정보</h3>
            <div class="fare-details">
              <div class="fare-row">
                <span>기본 요금:</span>
                <span>{{ calculatedFare.baseFare.toLocaleString() }}바트</span>
              </div>
              <div v-if="calculatedFare.zone" class="fare-row">
                <span>추가 요금 ({{ calculatedFare.zone }} 구역):</span>
                <span>{{ calculatedFare.extraFee.toLocaleString() }}바트</span>
              </div>
              <div class="fare-row total">
                <span>총 요금:</span>
                <span>{{ calculatedFare.totalFare.toLocaleString() }}바트</span>
              </div>
            </div>
            
            <div class="route-info">
              <p><strong>{{ selectedDeparture }}</strong> → <strong>{{ selectedDestination }}</strong></p>
              <p>차종: {{ vehicleTypes[selectedVehicle] }}</p>
              <p v-if="calculatedFare.zone">선택된 구역: {{ calculatedFare.zone }}</p>
            </div>

            <div class="notice">
              <p v-if="!calculatedFare.zone">📍 지도에서 정확한 위치를 클릭하면 해당 구역의 추가요금이 자동으로 계산됩니다!</p>
              <p v-else>✅ 지도에서 {{ calculatedFare.zone }} 구역이 선택되었습니다!</p>
            </div>
            
            <div class="dialog-actions">
              <button class="action-btn secondary" @click="resetAll">전체 초기화</button>
              <button class="action-btn primary" @click="closeFareDialog">확인</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 주소 입력 안내 다이얼로그 -->
    <div v-if="showAddressGuideDialog" class="dialog-overlay" @click="closeAddressGuideDialog">
      <div class="address-guide-dialog" @click.stop>
        <div class="dialog-header">
          <h2>🔍 주소 검색 안내</h2>
          <button class="close-btn" @click="closeAddressGuideDialog">×</button>
        </div>
        
        <div class="dialog-content">
          <div class="guide-content">
            <div class="guide-icon">📍</div>
            <h3>정확한 요금 계산을 위해<br/>구역 내부를 클릭하거나 주소를 검색해 주세요!</h3>
            
            <div class="guide-steps">
              <div class="step">
                <span class="step-number">1</span>
                <p>위쪽 검색창에 원하는 주소나 장소명을 입력하세요</p>
              </div>
              <div class="step">
                <span class="step-number">2</span>
                <p>검색 결과에서 정확한 위치를 선택하세요</p>
              </div>
              <div class="step">
                <span class="step-number">3</span>
                <p>구역별 추가요금이 자동으로 계산됩니다</p>
              </div>
            </div>
            
            <div class="guide-note">
              <p>💡 지도를 직접 클릭하는 것보다 주소 검색을 통해<br/>더 정확한 위치와 요금을 확인하실 수 있습니다.</p>
            </div>
            
            <div class="guide-actions">
              <button class="action-btn secondary" @click="closeAddressGuideDialog">닫기</button>
              <button class="action-btn primary" @click="focusOnSearchInput">주소 검색하기</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 주소 확인 다이얼로그 -->
    <div v-if="showAddressConfirmDialog" class="dialog-overlay" @click="closeAddressConfirmDialog">
      <div class="address-confirm-dialog" @click.stop>
        <div class="dialog-header">
          <h2>🔍 주소 확인</h2>
          <button class="close-btn" @click="closeAddressConfirmDialog">×</button>
        </div>
        
        <div class="dialog-content">
          <div class="address-confirm-info">
            <div class="search-result-header">
              <span class="search-icon">📍</span>
              <h3>검색된 주소</h3>
            </div>
            
            <div class="address-result-card">
              <p class="result-address">{{ pendingAddressInfo?.address }}</p>
              <div class="result-zone-badge" :class="pendingAddressInfo?.zone">
                <span class="zone-dot" :class="pendingAddressInfo?.zone"></span>
                {{ pendingAddressInfo?.zone }} 구역 (+{{ zoneExtraFee[pendingAddressInfo?.zone || ''] }}바트)
              </div>
            </div>
            
            <div class="confirm-question">
              <h4>이 주소로 요금을 계산하시겠습니까?</h4>
              <p>확인을 누르시면 출발지와 도착지를 선택하여 정확한 요금을 계산할 수 있습니다.</p>
            </div>
            
            <div class="confirm-actions">
              <button class="action-btn secondary" @click="closeAddressConfirmDialog">취소</button>
              <button class="action-btn primary" @click="confirmFareCalculation">확인</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 구역 외 주소 알림 다이얼로그 -->
    <div v-if="showOutsideZoneDialog" class="dialog-overlay" @click="closeOutsideZoneDialog">
      <div class="outside-zone-dialog" @click.stop>
        <div class="dialog-header">
          <h2>🚗 서비스 구역 안내</h2>
          <button class="close-btn" @click="closeOutsideZoneDialog">×</button>
        </div>
        
        <div class="dialog-content">
          <div class="outside-zone-info">
            <div class="info-icon">📍</div>
            <h3>검색하신 주소</h3>
            <p class="address">{{ outsideZoneInfo?.address }}</p>
            
            <div class="warning-message">
              <div class="warning-icon">⚠️</div>
              <p>죄송합니다. 검색하신 주소는 현재 박군투어 서비스 구역에 포함되지 않습니다.</p>
            </div>
            
            <div class="service-area-info">
              <h4>현재 서비스 구역:</h4>
              <ul>
                <li><span class="zone-color green"></span> 초록 구역 (추가요금 없음)</li>
                <li><span class="zone-color yellow"></span> 노랑 구역 (+100바트)</li>
                <li><span class="zone-color red"></span> 빨강 구역 (+200바트)</li>
              </ul>
            </div>
            
            <div class="contact-info">
              <p>📞 별도 문의가 필요하시면 박군투어로 직접 연락해주세요.</p>
            </div>
            
            <div class="dialog-actions" style="justify-content: flex-end;">
              <button class="action-btn primary" @click="closeOutsideZoneDialog">확인</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, shallowRef, computed, watch } from 'vue';
// @ts-ignore
import { Loader } from '@googlemaps/js-api-loader';

// --- 타입 정의 ---
declare var google: any;
interface Position {
  lat: number;
  lng: number;
}

// 요금표 인터페이스
interface FareData {
  sedan: number;
  suv: number;
  van: number;
}

// 출발지-도착지 요금표
interface RouteKey {
  from: string;
  to: string;
}

// 차종 타입
type VehicleType = 'sedan' | 'suv' | 'van';

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

// 요금 계산 관련 상태
const selectedDeparture = ref<string>('');
const selectedDestination = ref<string>('');
const selectedVehicle = ref<VehicleType>('sedan');
const showFareDialog = ref<boolean>(false);
const selectedZone = ref<string>('');

// 출발지 목록 (요금표에서 출발지로 가능한 지역)
const departureLocations = [
  '수완나품 공항', '방콕', '돈므앙 공항', '파타야'
];

// 도착지 목록 (모든 지역)
const destinationLocations = [
  '수완나품 공항', '방콕', '돈므앙 공항', '파타야', '후아힌', '라용[반패]', '아유타야'
];

// 출발지를 제외한 도착지 목록
const availableDestinations = computed(() => {
  let availableList = destinationLocations.filter(location => location !== selectedDeparture.value);
  
  // 각 출발지별로 실제 요금표에 있는 도착지만 표시
  if (selectedDeparture.value === '수완나품 공항') {
    // 수완나품 공항 → 방콕, 파타야, 후아힌만 가능
    availableList = availableList.filter(location => 
      ['방콕', '파타야', '후아힌'].includes(location)
    );
  } else if (selectedDeparture.value === '돈므앙 공항') {
    // 돈므앙 공항 → 방콕, 파타야만 가능
    availableList = availableList.filter(location => 
      ['방콕', '파타야'].includes(location)
    );
  } else if (selectedDeparture.value === '파타야') {
    // 파타야 → 수완나품 공항, 방콕, 돈므앙 공항, 라용[반패], 아유타야만 가능
    availableList = availableList.filter(location => 
      ['수완나품 공항', '방콕', '돈므앙 공항', '라용[반패]', '아유타야'].includes(location)
    );
  } else if (selectedDeparture.value === '방콕') {
    // 방콕 → 모든 곳 가능 (자기 자신 제외)
    // 이미 위에서 자기 자신은 제외되었으므로 그대로 사용
  }
  
  return availableList;
});

// 출발지 변경 감지하여 도착지 자동 초기화
watch(selectedDeparture, (newDeparture) => {
  // 출발지와 도착지가 같으면 도착지를 초기화
  if (newDeparture === selectedDestination.value) {
    selectedDestination.value = '';
  }
  
  // 현재 선택된 도착지가 새로운 가능한 목록에 없으면 초기화
  if (selectedDestination.value && !availableDestinations.value.includes(selectedDestination.value)) {
    selectedDestination.value = '';
  }
});

// 구역 외 알림 다이얼로그 상태
const showOutsideZoneDialog = ref<boolean>(false);
const outsideZoneInfo = ref<{ address: string, position: { lat: number, lng: number } } | null>(null);

// 검색된 주소 정보
const searchedAddress = ref<string>('');

// 주소 확인 다이얼로그 상태
const showAddressConfirmDialog = ref<boolean>(false);
const pendingAddressInfo = ref<{ address: string, zone: string, position: any } | null>(null);

// 주소 입력 안내 다이얼로그 상태
const showAddressGuideDialog = ref<boolean>(false);

// 폴리곤 클릭 정보 윈도우 상태
const infoWindow = shallowRef<any | null>(null);
const clickedLocationInfo = ref<{ position: any, zone: string } | null>(null);

const vehicleTypes = {
  sedan: '세단',
  suv: 'SUV', 
  van: '밴'
};

// 이미지에 나온 요금표 (바트 단위)
const fareTable: Record<string, Record<string, FareData>> = {
  '수완나품 공항': {
    '방콕': { sedan: 600, suv: 700, van: 1000 },
    '파타야': { sedan: 1000, suv: 1200, van: 1700 },
    '후아힌': { sedan: 2000, suv: 2200, van: 2700 },
  },
  '방콕': {
    '수완나품 공항': { sedan: 600, suv: 700, van: 1000 },
    '돈므앙 공항': { sedan: 800, suv: 900, van: 1200 },
    '파타야': { sedan: 1300, suv: 1500, van: 1900 },
    '후아힌': { sedan: 2000, suv: 2200, van: 2700 },
    '라용[반패]': { sedan: 1900, suv: 2100, van: 2500 },
    '아유타야': { sedan: 1500, suv: 1800, van: 2100 }
  },
  '돈므앙 공항': {
    '방콕': { sedan: 800, suv: 900, van: 1200 },
    '파타야': { sedan: 1600, suv: 1700, van: 2200 },
  },
  '파타야': {
    '수완나품 공항': { sedan: 1000, suv: 1200, van: 1700 },
    '방콕': { sedan: 1300, suv: 1500, van: 1900 },
    '돈므앙 공항': { sedan: 1600, suv: 1700, van: 2200 },
    '라용[반패]': { sedan: 1300, suv: 1500, van: 1800 },
    '아유타야': { sedan: 2300, suv: 2600, van: 3300 }
  },
};

// 지역별 추가 요금 (바트 단위)
const zoneExtraFee: Record<string, number> = {
  '초록': 0,    // 추가요금 없음
  '노랑': 100,  // 추가 100바트
  '빨강': 200   // 추가 200바트
};

// 계산된 요금 정보
const calculatedFare = computed(() => {
  if (!selectedDeparture.value || !selectedDestination.value) return null;
  
  const baseFare = fareTable[selectedDeparture.value]?.[selectedDestination.value]?.[selectedVehicle.value];
  if (!baseFare) return null;
  
  const extraFee = selectedZone.value ? zoneExtraFee[selectedZone.value] || 0 : 0;
  
  return {
    baseFare,
    zone: selectedZone.value,
    extraFee,
    totalFare: baseFare + extraFee
  };
});

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
      // 새로운 검색 결과 처리 함수 호출
      handleSearchResult(place);
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
    clickable: true
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
    clickable: true
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
    clickable: true
  });
  redPolygon.setMap(map.value);

  // 각 폴리곤에 개별 클릭 이벤트 추가
  greenPolygon.addListener('click', (event: any) => {
    console.log('초록 폴리곤 직접 클릭됨');
    handlePolygonClick(event, '초록');
  });

  yellowPolygon.addListener('click', (event: any) => {
    console.log('노랑 폴리곤 직접 클릭됨');
    handlePolygonClick(event, '노랑');
  });

  redPolygon.addListener('click', (event: any) => {
    console.log('빨강 폴리곤 직접 클릭됨');
    handlePolygonClick(event, '빨강');
  });

  // 폴리곤들을 배열에 저장
  fixedPolygons.value = [greenPolygon, yellowPolygon, redPolygon];
};

// --- 지도 클릭 이벤트 핸들러 ---
const handleMapClick = (event: any) => {
  console.log('지도 클릭됨:', event.latLng?.lat(), event.latLng?.lng());
  console.log('폴리곤 개수:', fixedPolygons.value.length);
  
  if (!event.latLng || fixedPolygons.value.length === 0) {
    console.log('클릭 이벤트 또는 폴리곤이 없음');
    return;
  }

  // 클릭한 위치가 어떤 폴리곤 내부에 있는지 확인
  let clickedPolygon = null;
  let polygonColor = '';
  
  for (const polygon of fixedPolygons.value) {
    const isInside = google.maps.geometry.poly.containsLocation(event.latLng, polygon);
    console.log('폴리곤 내부 체크:', isInside, polygon.get('fillColor'));
    
    if (isInside) {
      clickedPolygon = polygon;
      // 폴리곤 색깔 구분
      const fillColor = polygon.get('fillColor');
      if (fillColor === '#00FF00') polygonColor = '초록';
      else if (fillColor === '#FFFF00') polygonColor = '노랑';
      else if (fillColor === '#FF0000') polygonColor = '빨강';
      console.log('폴리곤 내부 클릭됨:', polygonColor);
      break;
    }
  }

  if (clickedPolygon) {
    // 폴리곤 내부 클릭 - 마커 표시 및 계산 팝업
    console.log('폴리곤 클릭 처리 시작');
    handlePolygonClick(event, polygonColor);
  } else {
    // 기존 정보창이 있다면 닫기
    if (infoWindow.value) {
      infoWindow.value.close();
    }
    
    // 폴리곤 외부 클릭 - 주소 검색 안내
    console.log('폴리곤 외부 클릭');
    showAddressInputGuide();
  }
};



// 다이얼로그 닫기 함수
const closeFareDialog = () => {
  showFareDialog.value = false;
  // 검색된 주소 정보는 유지 (사용자가 다시 열 때 확인 가능)
};

// 전체 초기화 함수
const resetAll = () => {
  selectedDeparture.value = '';
  selectedDestination.value = '';
  selectedVehicle.value = 'sedan';
  selectedZone.value = '';
  searchedAddress.value = '';
  showFareDialog.value = false;
  showAddressConfirmDialog.value = false;
  showAddressGuideDialog.value = false;
  pendingAddressInfo.value = null;
  clickedLocationInfo.value = null;
  
  // 마커 제거
  if (marker.value) {
    marker.value.setMap(null);
    marker.value = null;
  }
  
  // 정보창 제거
  if (infoWindow.value) {
    infoWindow.value.close();
    infoWindow.value = null;
  }
};

// 구역 외 다이얼로그 닫기 함수
const closeOutsideZoneDialog = () => {
  showOutsideZoneDialog.value = false;
  outsideZoneInfo.value = null;
};

// 주소 확인 다이얼로그 닫기 함수
const closeAddressConfirmDialog = () => {
  showAddressConfirmDialog.value = false;
  pendingAddressInfo.value = null;
};

// 요금 계산 확인 함수
const confirmFareCalculation = () => {
  if (!pendingAddressInfo.value) return;
  
  // 검색된 주소 정보 저장
  selectedZone.value = pendingAddressInfo.value.zone;
  searchedAddress.value = pendingAddressInfo.value.address;
  
  // 기존 마커가 있다면 삭제
  if (marker.value) {
    marker.value.setMap(null);
  }
  
  // 새 위치에 마커 생성
  marker.value = new google.maps.Marker({
    position: pendingAddressInfo.value.position,
    map: map.value,
    title: `${pendingAddressInfo.value.address} (${pendingAddressInfo.value.zone} 구역)`,
  });
  
  // 확인 다이얼로그 닫기
  closeAddressConfirmDialog();
  
  // 요금 계산 다이얼로그 열기
  showFareDialog.value = true;
};

// 주소 입력 안내 함수
const showAddressInputGuide = () => {
  showAddressGuideDialog.value = true;
};

// 주소 입력 안내 다이얼로그 닫기 함수
const closeAddressGuideDialog = () => {
  showAddressGuideDialog.value = false;
};

// 검색창으로 포커스 이동 함수
const focusOnSearchInput = () => {
  closeAddressGuideDialog();
  if (searchInput.value) {
    searchInput.value.focus();
  }
};

// 폴리곤 클릭 처리 함수
const handlePolygonClick = (event: any, polygonColor: string) => {
  const lat = event.latLng.lat();
  const lng = event.latLng.lng();
  
  // 클릭 위치 정보 저장
  clickedLocationInfo.value = {
    position: event.latLng,
    zone: polygonColor
  };
  
  // 기존 마커가 있다면 삭제
  if (marker.value) {
    marker.value.setMap(null);
  }
  
  // 새 위치에 마커 생성
  marker.value = new google.maps.Marker({
    position: event.latLng,
    map: map.value,
    title: `선택된 위치 (${polygonColor} 구역)`,
  });
  
  // 기존 정보창이 있다면 닫기
  if (infoWindow.value) {
    infoWindow.value.close();
  }
  
  // 정보창 내용 생성
  const contentString = `
    <div style="padding: 10px; font-family: 'Inter', sans-serif; min-width: 140px; max-width: 180px; user-select: none; -webkit-user-select: none; -webkit-touch-callout: none;">
      <div style="display: flex; align-items: center; gap: 6px; margin-bottom: 8px;">
        <div style="width: 8px; height: 8px; border-radius: 50%; background-color: ${getZoneColor(polygonColor)}; border: 1px solid white; box-shadow: 0 1px 2px rgba(0,0,0,0.2);"></div>
        <strong style="color: #333; font-size: 12px;">${polygonColor} 구역</strong>
      </div>
      <div style="color: #666; font-size: 11px; margin-bottom: 8px;">
        +${zoneExtraFee[polygonColor]}바트
      </div>
      <button id="calculateFareBtn" style="
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        border: none;
        padding: 8px 12px;
        border-radius: 15px;
        font-size: 11px;
        font-weight: 600;
        cursor: pointer;
        transition: all 0.3s ease;
        width: 100%;
        touch-action: manipulation;
        -webkit-tap-highlight-color: transparent;
        user-select: none;
        -webkit-user-select: none;
        min-height: 32px;
      ">
        이 위치로 계산하기
      </button>
    </div>
  `;
  
  // 정보창 생성
  infoWindow.value = new google.maps.InfoWindow({
    content: contentString,
    position: event.latLng,
    disableAutoPan: false, // 자동 패닝 활성화
    maxWidth: 200, // 최대 너비 설정
    pixelOffset: new google.maps.Size(0, -10) // 약간 위로 오프셋
  });
  
  // 정보창 열기
  infoWindow.value.open(map.value);
  
  // 정보창이 닫히지 않도록 이벤트 방지
  google.maps.event.addListener(infoWindow.value, 'closeclick', (e: any) => {
    console.log('정보창 닫기 시도');
  });
  
  // 정보창 내 버튼 이벤트 리스너 추가
  google.maps.event.addListenerOnce(infoWindow.value, 'domready', () => {
    const button = document.getElementById('calculateFareBtn');
    if (button) {
      // 터치와 클릭 모두 처리
      button.addEventListener('click', (e) => {
        e.preventDefault();
        e.stopPropagation();
        startFareCalculationFromClick();
      });
      
      button.addEventListener('touchend', (e) => {
        e.preventDefault();
        e.stopPropagation();
        startFareCalculationFromClick();
      });
    }
  });
};

// 구역 색상 반환 함수
const getZoneColor = (zone: string): string => {
  switch (zone) {
    case '초록': return '#00FF00';
    case '노랑': return '#FFFF00';
    case '빨강': return '#FF0000';
    default: return '#888888';
  }
};

// 클릭한 위치로 요금 계산 시작
const startFareCalculationFromClick = () => {
  if (!clickedLocationInfo.value) return;
  
  // 선택된 구역 정보 저장
  selectedZone.value = clickedLocationInfo.value.zone;
  searchedAddress.value = `선택된 위치 (${clickedLocationInfo.value.zone} 구역)`;
  
  // 정보창 닫기
  if (infoWindow.value) {
    infoWindow.value.close();
  }
  
  // 요금 계산 다이얼로그 열기
  showFareDialog.value = true;
};

// 좌표가 어떤 구역에 속하는지 판단하는 함수
const getZoneFromCoordinates = (lat: number, lng: number): string | null => {
  if (!map.value || fixedPolygons.value.length === 0) return null;
  
  const position = new google.maps.LatLng(lat, lng);
  
  for (const polygon of fixedPolygons.value) {
    const isInside = google.maps.geometry.poly.containsLocation(position, polygon);
    if (isInside) {
      const fillColor = polygon.get('fillColor');
      if (fillColor === '#00FF00') return '초록';
      else if (fillColor === '#FFFF00') return '노랑';
      else if (fillColor === '#FF0000') return '빨강';
    }
  }
  
  return null; // 구역 밖
};

// 주소 검색 결과 처리 함수
const handleSearchResult = (place: any) => {
  if (!place.geometry || !place.geometry.location) return;
  
  const lat = place.geometry.location.lat();
  const lng = place.geometry.location.lng();
  const address = place.name || place.formatted_address;
  
  // 구역 판단
  const zone = getZoneFromCoordinates(lat, lng);
  
  if (zone) {
    // 구역 안에 있는 경우 - 확인 다이얼로그 표시
    pendingAddressInfo.value = {
      address: address,
      zone: zone,
      position: place.geometry.location
    };
    
    // 지도 이동 (미리 보여주기)
    map.value.setCenter(place.geometry.location);
    map.value.setZoom(17);
    
    // 확인 다이얼로그 열기
    showAddressConfirmDialog.value = true;
    
  } else {
    // 구역 밖에 있는 경우
    outsideZoneInfo.value = {
      address: address,
      position: { lat, lng }
    };
    showOutsideZoneDialog.value = true;
  }
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

/* 요금 계산 버튼 */
.fare-calculator-btn {
  position: absolute;
  bottom: 25px;
  left: 50%;
  transform: translateX(-50%);
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 15px 25px;
  border-radius: 25px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  z-index: 10;
}

.fare-calculator-btn:hover {
  transform: translateX(-50%) translateY(-2px);
  box-shadow: 0 15px 20px rgba(0, 0, 0, 0.2);
}

/* 다이얼로그 오버레이 */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

/* 요금 계산 다이얼로그 */
.fare-dialog {
  background: white;
  border-radius: 15px;
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 25px;
  border-bottom: 1px solid #eee;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 15px 15px 0 0;
}

.dialog-header h2 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
}

.close-btn {
  background: none;
  border: none;
  color: white;
  font-size: 28px;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s ease;
}

.close-btn:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.dialog-content {
  padding: 25px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #333;
  font-size: 14px;
}

.form-group select {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  font-size: 16px;
  background-color: white;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
}

.form-group select:focus {
  outline: none;
  border-color: #667eea;
}

.vehicle-options {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.vehicle-option {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 10px 15px;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  transition: all 0.3s ease;
  flex: 1;
  min-width: 80px;
  justify-content: center;
}

.vehicle-option:hover {
  border-color: #667eea;
  background-color: #f8f9ff;
}

.vehicle-option input[type="radio"] {
  margin-right: 8px;
  accent-color: #667eea;
}

.vehicle-option input[type="radio"]:checked + span {
  font-weight: 600;
  color: #667eea;
}

.fare-result {
  margin-top: 25px;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 12px;
  border-left: 4px solid #667eea;
}

.fare-result h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 18px;
}

.fare-details {
  margin-bottom: 15px;
}

.fare-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.3);
}

.fare-row:last-child {
  border-bottom: none;
}

.fare-row.total {
  font-weight: 700;
  font-size: 18px;
  color: #667eea;
  margin-top: 10px;
  padding-top: 15px;
  border-top: 2px solid #667eea;
  border-bottom: none;
}

.route-info {
  background: white;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.route-info p {
  margin: 5px 0;
  color: #333;
}

.notice {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  color: #856404;
  padding: 12px;
  border-radius: 8px;
  font-size: 14px;
}

.notice p {
  margin: 0;
}

/* 검색된 주소 정보 스타일 */
.searched-address-info {
  margin-bottom: 25px;
  padding: 20px;
  background: linear-gradient(135deg, #e8f5e8 0%, #f0f8ff 100%);
  border-radius: 12px;
  border: 2px solid #4caf50;
}

.address-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 15px;
}

.address-icon {
  font-size: 24px;
}

.address-header h3 {
  margin: 0;
  color: #2e7d32;
  font-size: 18px;
  font-weight: 600;
}

.address-card {
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.address-text {
  margin: 0 0 12px 0;
  font-size: 16px;
  font-weight: 600;
  color: #333;
  line-height: 1.4;
}

.zone-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
  color: white;
}

.zone-badge.초록 {
  background: linear-gradient(135deg, #4caf50 0%, #66bb6a 100%);
}

.zone-badge.노랑 {
  background: linear-gradient(135deg, #ff9800 0%, #ffb74d 100%);
}

.zone-badge.빨강 {
  background: linear-gradient(135deg, #f44336 0%, #ef5350 100%);
}

.zone-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.zone-indicator.초록 {
  background-color: #00FF00;
}

.zone-indicator.노랑 {
  background-color: #FFFF00;
}

.zone-indicator.빨강 {
  background-color: #FF0000;
}

/* 주소 입력 안내 다이얼로그 */
.address-guide-dialog {
  background: white;
  border-radius: 15px;
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.guide-content {
  text-align: center;
}

.guide-icon {
  font-size: 64px;
  margin-bottom: 20px;
}

.guide-content h3 {
  margin: 0 0 30px 0;
  color: #333;
  font-size: 22px;
  font-weight: 600;
  line-height: 1.4;
}

.guide-steps {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 25px;
  margin: 25px 0;
  text-align: left;
}

.step {
  display: flex;
  align-items: flex-start;
  gap: 15px;
  margin-bottom: 20px;
}

.step:last-child {
  margin-bottom: 0;
}

.step-number {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 14px;
  flex-shrink: 0;
}

.step p {
  margin: 0;
  color: #495057;
  line-height: 1.5;
  font-size: 15px;
}

.guide-note {
  background: linear-gradient(135deg, #e3f2fd 0%, #f3e5f5 100%);
  border: 1px solid #90caf9;
  border-radius: 12px;
  padding: 20px;
  margin: 25px 0;
}

.guide-note p {
  margin: 0;
  color: #1565c0;
  line-height: 1.5;
  font-size: 14px;
}

.guide-actions {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 30px;
}

.guide-actions .action-btn {
  flex: 1;
  max-width: 150px;
}

/* 주소 입력 안내 다이얼로그 반응형 */
@media (max-width: 768px) {
  .address-guide-dialog {
    width: 95%;
    margin: 10px;
  }
  
  .guide-icon {
    font-size: 48px;
  }
  
  .guide-content h3 {
    font-size: 20px;
  }
  
  .guide-steps {
    padding: 20px;
  }
  
  .step-number {
    width: 24px;
    height: 24px;
    font-size: 12px;
  }
  
  .step p {
    font-size: 14px;
  }
}

@media (max-width: 480px) {
  .guide-actions {
    flex-direction: column;
  }
  
  .guide-actions .action-btn {
    max-width: none;
  }
  
  .guide-content h3 {
    font-size: 18px;
  }
  
  .guide-note p {
    font-size: 13px;
  }
}

/* 주소 확인 다이얼로그 */
.address-confirm-dialog {
  background: white;
  border-radius: 15px;
  width: 90%;
  max-width: 450px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.address-confirm-info {
  text-align: center;
}

.search-result-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.search-icon {
  font-size: 28px;
}

.search-result-header h3 {
  margin: 0;
  color: #333;
  font-size: 20px;
  font-weight: 600;
}

.address-result-card {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border: 2px solid #007bff;
  border-radius: 12px;
  padding: 20px;
  margin: 20px 0;
}

.result-address {
  margin: 0 0 15px 0;
  font-size: 18px;
  font-weight: 700;
  color: #212529;
  line-height: 1.4;
}

.result-zone-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  border-radius: 25px;
  font-size: 14px;
  font-weight: 600;
  color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.result-zone-badge.초록 {
  background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
}

.result-zone-badge.노랑 {
  background: linear-gradient(135deg, #ffc107 0%, #fd7e14 100%);
}

.result-zone-badge.빨강 {
  background: linear-gradient(135deg, #dc3545 0%, #e83e8c 100%);
}

.zone-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 2px solid white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

.zone-dot.초록 {
  background-color: #00FF00;
}

.zone-dot.노랑 {
  background-color: #FFFF00;
}

.zone-dot.빨강 {
  background-color: #FF0000;
}

.confirm-question {
  background: #e3f2fd;
  border-radius: 12px;
  padding: 20px;
  margin: 25px 0;
  text-align: center;
}

.confirm-question h4 {
  margin: 0 0 10px 0;
  color: #1565c0;
  font-size: 18px;
  font-weight: 600;
}

.confirm-question p {
  margin: 0;
  color: #424242;
  line-height: 1.5;
  font-size: 14px;
}

.confirm-actions {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 25px;
}

.confirm-actions .action-btn {
  flex: 1;
  max-width: 120px;
}

/* 주소 확인 다이얼로그 반응형 */
@media (max-width: 768px) {
  .address-confirm-dialog {
    width: 95%;
    margin: 10px;
  }
  
  .search-icon {
    font-size: 24px;
  }
  
  .search-result-header h3 {
    font-size: 18px;
  }
  
  .result-address {
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .confirm-actions {
    flex-direction: column;
  }
  
  .confirm-actions .action-btn {
    max-width: none;
  }
  
  .address-result-card {
    padding: 15px;
  }
  
  .confirm-question {
    padding: 15px;
  }
  
  .confirm-question h4 {
    font-size: 16px;
  }
}

/* 반응형 디자인 */
@media (max-width: 768px) {
  .fare-dialog {
    width: 95%;
    margin: 10px;
  }
  
  .dialog-header {
    padding: 15px 20px;
  }
  
  .dialog-header h2 {
    font-size: 18px;
  }
  
  .dialog-content {
    padding: 20px;
  }
  
  .vehicle-options {
    flex-direction: column;
  }
  
  .vehicle-option {
    min-width: auto;
  }
}

@media (max-width: 480px) {
  .fare-calculator-btn {
    bottom: 20px;
    padding: 12px 20px;
    font-size: 15px;
  }
  
  .dialog-header {
    padding: 12px 15px;
  }
  
  .dialog-header h2 {
    font-size: 16px;
  }
  
  .dialog-content {
    padding: 15px;
  }
}

/* 구역 외 주소 다이얼로그 */
.outside-zone-dialog {
  background: white;
  border-radius: 15px;
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.outside-zone-info {
  text-align: center;
}

.info-icon {
  font-size: 48px;
  margin-bottom: 15px;
}

.outside-zone-info h3 {
  margin: 10px 0;
  color: #333;
  font-size: 18px;
}

.address {
  background: #f8f9fa;
  padding: 12px;
  border-radius: 8px;
  margin: 15px 0;
  font-weight: 600;
  color: #495057;
  border-left: 4px solid #007bff;
}

.warning-message {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  color: #856404;
  padding: 15px;
  border-radius: 8px;
  margin: 20px 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.warning-icon {
  font-size: 24px;
  flex-shrink: 0;
}

.warning-message p {
  margin: 0;
  line-height: 1.4;
}

.service-area-info {
  background: #e3f2fd;
  padding: 15px;
  border-radius: 8px;
  margin: 20px 0;
  text-align: left;
}

.service-area-info h4 {
  margin: 0 0 10px 0;
  color: #1976d2;
  font-size: 16px;
}

.service-area-info ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.service-area-info li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 5px 0;
  color: #333;
}

.zone-color {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 2px solid #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.zone-color.green {
  background-color: #00FF00;
}

.zone-color.yellow {
  background-color: #FFFF00;
}

.zone-color.red {
  background-color: #FF0000;
}

.contact-info {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin: 20px 0;
  text-align: left;
}

.contact-info p {
  margin: 5px 0;
  color: #666;
  line-height: 1.4;
}

.dialog-actions {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.action-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.action-btn.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.action-btn.primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.action-btn.secondary {
  background: #6c757d;
  color: white;
}

.action-btn.secondary:hover {
  background: #5a6268;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(108, 117, 125, 0.4);
}

/* 구역 외 다이얼로그 반응형 */
@media (max-width: 768px) {
  .outside-zone-dialog {
    width: 95%;
    margin: 10px;
  }
  
  .info-icon {
    font-size: 36px;
  }
  
  .outside-zone-info h3 {
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .warning-message {
    flex-direction: column;
    text-align: center;
  }
  
  .warning-icon {
    margin-bottom: 5px;
  }
}
</style>
