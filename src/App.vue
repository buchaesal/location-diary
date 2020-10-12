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
    items : [
      {title:'일기쓰기'},
      {title:'이전 일기 보기'}
    ]
  }),

  methods : {
    initMap() {
      var container = document.getElementById('map');
      var options = {
        center: new kakao.maps.LatLng(33.450701, 126.570667),
        level: 3
      };
      var map = new kakao.maps.Map(container, options); //마커추가하려면 객체를 아래와 같이 하나 만든다.
      var marker = new kakao.maps.Marker({
        position: map.getCenter()
      });
      marker.setMap(map);


      var that = this;

      kakao.maps.event.addListener(marker, 'click', function() {
        // 마커에 클릭 이벤트가 발생하면 인포윈도우를 마커위에 표시합니다.
        that.dialog = true;
      });

      // 지도에 클릭 이벤트를 등록합니다
      // 지도를 클릭하면 마지막 파라미터로 넘어온 함수를 호출합니다
      kakao.maps.event.addListener(map, 'click', function(mouseEvent) {

        // 클릭한 위도, 경도 정보를 가져옵니다
        var latlng = mouseEvent.latLng;

        // 마커 위치를 클릭한 위치로 옮깁니다
        marker.setPosition(latlng);
        //
        // var message = '클릭한 위치의 위도는 ' + latlng.getLat() + ' 이고, ';
        // message += '경도는 ' + latlng.getLng() + ' 입니다';

      });

      },
      addScript() {
      const script = document.createElement('script'); /* global kakao */
        script.onload = () => kakao.maps.load(this.initMap);
        script.src = 'http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=2d05efef098285ddc71f867b644dc65a';
        document.head.appendChild(script);
    }
  }

};
</script>
