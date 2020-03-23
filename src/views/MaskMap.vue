<template>
  <div class="mask_map">
    <VueDaumMap
      class="daumMap"
      :appKey="appKey"
      :center.sync="center"
      :level.sync="level"
      :libraries="libraries"
      :mapTypeId="mapTypeId"
      :scrollwheel="scrollwheel"
      @click="mouseClick"
      @load="onLoad"
    ></VueDaumMap>
    <SearchLocation v-on:send_location="searchLocation"></SearchLocation>
    <MaskInfo></MaskInfo>
    {{ locationChange }}
  </div>
</template>
//TODO 일반 지도 클릭했을 시 마커 지우기. //TODO 마커 클릭시 중심 위치 이동.
<script>
import SearchLocation from '@/components/SearchLocation.vue';
import MaskInfo from '@/components/MaskInfo.vue';
import VueDaumMap from 'vue-daum-map';
import axios from 'axios';

export default {
  components: {
    VueDaumMap,
    MaskInfo,
    SearchLocation,
  },
  mounted() {
    if ('geolocation' in navigator) {
      //위치 정보를 얻기
      navigator.geolocation.getCurrentPosition(res => {
        this.center.lat = parseFloat(res.coords.latitude);
        this.center.lng = parseFloat(res.coords.longitude);
      });
    }
  },
  computed: {
    locationChange() {
      // 무분별한 호출을 방지하기 위한 위치 이동값 검사 로직
      if (this.map == null) return;
      // console.log(typeof this.center.lat);

      const location = {
        lat: this.center.lat.toFixed(3),
        lng: this.center.lng.toFixed(3),
      };
      return this.copyLocation(location);
    },
  },
  data() {
    return {
      map: null,
      gecoder: null,
      appKey: process.env.VUE_APP_DAUM_APPKEY,
      center: { lat: 37.48435673307468, lng: 127.03496001846625 }, // 지도의 중심 좌표
      copyCenter: { lat: '', lng: '' }, // 지도 과거 중심 좌표 저장 값
      level: 2, // 지도의 레벨(확대, 축소 정도),
      libraries: ['services'], // 추가로 불러올 라이브러리
      mapTypeId: VueDaumMap.MapTypeId.NORMAL,
      scrollwheel: true,
      maskImg: {
        green:
          'https://find-mask.s3.ap-northeast-2.amazonaws.com/img/green.png',
        yellow:
          'https://find-mask.s3.ap-northeast-2.amazonaws.com/img/yellow.png',
        red: 'https://find-mask.s3.ap-northeast-2.amazonaws.com/img/red.png',
        gray: 'https://find-mask.s3.ap-northeast-2.amazonaws.com/img/gray.png',
      },
      markers: [],
      customOverlay: [],
    };
  },
  methods: {
    onLoad(map) {
      this.map = map;
      this.gecoder = new kakao.maps.services.Geocoder();
    },
    mouseClick(mouseMovePoint) {
      // this.mouseClickPoint.lat = mouseMovePoint[0].latLng.Ha;
      // this.mouseClickPoint.lng = mouseMovePoint[0].latLng.Ga;
      if (this.customOverlay.length != 0) {
        for (let selected in this.customOverlay) {
          this.customOverlay[selected].setMap(null);
        }
      }
    },
    copyLocation(value) {
      let renewLat = this.center.lat.toFixed(3);
      let renewLng = this.center.lng.toFixed(3);

      if (this.copyCenter.lat != renewLat || this.copyCenter.lng != renewLng) {
        this.copyCenter.lat = value.lat;
        this.copyCenter.lng = value.lng;
        this.findMask();
        // console.log('조회했음!');
      }
    },
    async findMask() {
      // 현재 중심의 정확한 위도 경도 구하기
      const latlng = this.map.getCenter();

      const url =
        'https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/storesByGeo/json';
      try {
        const { data } = await axios.get(url, {
          params: {
            lat: latlng.getLat(),
            lng: latlng.getLng(),
            m: 1200,
          },
        });
        this.createMark(data.stores);
      } catch (error) {
        // console.log('네트워크 오류로 인한 위치조회 실패.');
      }
    },
    createMark(location) {
      // 마커 초기화 해보기
      if (this.markers.length > 300) {
        for (let a in this.markers) {
          this.markers[a].setMap(null);
        }
        this.markers = [];
      }

      // 마커 커스텀 셋팅
      const imageOption = { offset: new kakao.maps.Point(25, 45) };
      let imageSrc = ''; // 마커이미지의 주소
      let remain_stat = ''; // 재고상태 설명
      const imageSize = new kakao.maps.Size(45, 45); // 마커이미지의 크기

      // 마커 표시
      for (let data in location) {
        // 마스크 재고량에 따른 마커 이미지, 재고상태 설명 변경
        switch (location[data].remain_stat) {
          case 'plenty':
            imageSrc = this.maskImg.green;
            remain_stat = '많음 (100개 이상)';
            break;
          case 'some':
            imageSrc = this.maskImg.yellow;
            remain_stat = '보통 (30 ~ 99개)';
            break;
          case 'few':
            imageSrc = this.maskImg.red;
            remain_stat = '적음 (1 ~ 29개)';
            break;
          case 'empty':
            imageSrc = this.maskImg.gray;
            remain_stat = '매우적음 (0 ~ 1개)';
            break;
          case 'break':
            imageSrc = this.maskImg.gray;
            remain_stat = '판매중지';
            break;
        }

        // 마커 content 생성
        let content = `<div class="markInfo">
                        <div class="pharmacy_name">${location[data].name}</div>
                        <div class="remain_stat">재고 상태 : ${remain_stat}</div>
                        <div class="stock_at">입고 등록시간 : ${location[
                          data
                        ].stock_at.substr(5)}</div>
                        <div class="created_at">업데이트 시간 : ${location[
                          data
                        ].created_at.substr(5)}</div>
                      </div>`;

        // 마커 객체 생성
        const markerImage = new kakao.maps.MarkerImage(
          imageSrc,
          imageSize,
          imageOption,
        );

        // 현재 location 설정
        let markerPosition = new kakao.maps.LatLng(
          location[data].lat,
          location[data].lng,
        );
        // 마커의 이미지정보를 가지고 있는 마커이미지를 생성
        const marker = new kakao.maps.Marker({
          position: markerPosition,
          image: markerImage, // 마커이미지 설정 (커스텀)
          clickable: true,
        });

        // 약국정보 커스텀 오버레이를 지도에 생성
        const customOverlay = new kakao.maps.CustomOverlay({
          position: markerPosition,
          content: content,
          removable: true,
          yAnchor: 1,
        });

        this.markers.push(marker);
        this.customOverlay.push(customOverlay);

        // 마커에 클릭 이벤트 등록
        kakao.maps.event.addListener(marker, 'click', () => {
          // console.log('click!!!', data);
          for (let selected in this.customOverlay) {
            this.customOverlay[selected].setMap(null);
          }

          // this.center.lat = location[data].lat;
          // this.center.lng = location[data].lng;
          let moveLatLon = new kakao.maps.LatLng(
            location[data].lat,
            location[data].lng,
          );
          this.map.panTo(moveLatLon);
          customOverlay.setMap(this.map);
        });

        // 마커를 지도에 생성
        marker.setMap(this.map);
      }
    },
    searchLocation(locationString) {
      this.gecoder.addressSearch(locationString, (result, status) => {
        if (status === kakao.maps.services.Status.OK) {
          // console.log(result);
          this.center.lat = parseFloat(result[0].y);
          this.center.lng = parseFloat(result[0].x);
          this.findMask();
          this.level = 2;
        } else {
          // console.log('검색 실패!');
        }
      });
    },
  },
};
</script>

<style></style>
