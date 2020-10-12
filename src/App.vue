<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <div id="map" style="width: 70%; height: 70%"></div>

      <template>
        <v-row justify="center">
          <v-dialog
                  v-model="dialog2"
                  persistent
                  max-width="600px"
          >
            <v-card>
              <v-card-title>
                <span class="headline">Write Diary</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field
                              label="제목"
                              required
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12">
                      <v-textarea
                              label="내용"
                              row-height="15"
                              required
                      ></v-textarea>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                        color="blue darken-1"
                        text
                        @click="dialog2 = false"
                >
                  Close
                </v-btn>
                <v-btn
                        color="blue darken-1"
                        text
                        @click="dialog2 = false"
                >
                  Save
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>
      </template>


      <v-dialog
              v-model="dialog"
              max-width="300px"
      >
        <v-card>
          <v-card-title>
          </v-card-title>
          <v-card-text>
            <v-btn
                    color="accent"
                    dark
                    @click="dialog2 = true"
            >
              일기 쓰기
            </v-btn>
            <v-btn
                    color="secondary"
                    dark
            >
              지난 일기 보기
            </v-btn>
          </v-card-text>
          <v-card-actions>
            <v-btn
                    color="primary"
                    text
                    @click="dialog = false"
            >
              닫기
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <span>{{ address }}</span>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  mounted() {
    window.kakao && window.kakao.maps ? this.initMap() : this.addScript();
    },

  components: {
  },

  data: () => ({
    dialog : false,
    dialog2 : false,
    address: ''
  }),

  methods : {
    initMap() {
      var container = document.getElementById('map');
      var that = this;
      var options = {
        center: new kakao.maps.LatLng(33.450701, 126.570667),
        level: 3
      };
      var map = new kakao.maps.Map(container, options); //마커추가하려면 객체를 아래와 같이 하나 만든다.
      var marker = new kakao.maps.Marker({
        position: map.getCenter()
      });

      // HTML5의 geolocation으로 사용할 수 있는지 확인합니다
      if (navigator.geolocation) {

        // GeoLocation을 이용해서 접속 위치를 얻어옵니다
        navigator.geolocation.getCurrentPosition(function(position) {

          var lat = position.coords.latitude, // 위도
                  lon = position.coords.longitude; // 경도

          options.center = new kakao.maps.LatLng(lat, lon);
          map.setCenter(options.center);
          marker.setPosition(options.center);
          marker.setMap(map);

          searchDetailAddrFromCoords(options.center, function (result, status) {
            if (status === kakao.maps.services.Status.OK) {
              var detailAddr = result[0].road_address ? `도로명주소 :  ${result[0].road_address.address_name}` : '';
              detailAddr += `지번 주소 :  ${result[0].address.address_name}` ;

              that.address = detailAddr;
            }
          });

        });
      } else { // HTML5의 GeoLocation을 사용할 수 없을때 마커 표시 위치와 인포윈도우 내용을 설정합니다
                console.log('geolocation을 사용할수 없어요..');
        marker.setMap(map);

      }


      kakao.maps.event.addListener(marker, 'click', function() {
        // 마커에 클릭 이벤트가 발생하면 인포윈도우를 마커위에 표시합니다.
        that.dialog = true;
      });

      // 주소-좌표 변환 객체를 생성합니다
      var geocoder = new kakao.maps.services.Geocoder();

      function searchDetailAddrFromCoords(coords, callback) {
        // 좌표로 법정동 상세 주소 정보를 요청합니다
        geocoder.coord2Address(coords.getLng(), coords.getLat(), callback);
      }

      // 지도에 클릭 이벤트를 등록합니다
      // 지도를 클릭하면 마지막 파라미터로 넘어온 함수를 호출합니다
      kakao.maps.event.addListener(map, 'click', function(mouseEvent) {
        searchDetailAddrFromCoords(mouseEvent.latLng, function (result, status) {
          if (status === kakao.maps.services.Status.OK) {
            var detailAddr = result[0].road_address ? `도로명주소 :  ${result[0].road_address.address_name}` : '';
            detailAddr += `지번 주소 :  ${result[0].address.address_name}` ;

            // 마커를 클릭한 위치에 표시합니다
            marker.setPosition(mouseEvent.latLng);
            marker.setMap(map);

            that.address = detailAddr;
          }
        });

      });

      },
      addScript() {
      const script = document.createElement('script'); /* global kakao */
        script.onload = () => kakao.maps.load(this.initMap);
        script.src = 'http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=2d05efef098285ddc71f867b644dc65a&libraries=services';
        document.head.appendChild(script);
    }
  }

};
</script>
