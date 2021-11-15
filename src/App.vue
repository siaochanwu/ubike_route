<template>
	<div class="container mx-auto">
		<h1>www</h1>
		<div id="map" class="h-80"></div>
	</div>
</template>

<script lang="ts">
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import jsSHA from "jssha";
import { ref, onMounted } from "vue";

export default {
	setup() {
		const stationData = ref([])
		let mymap = ref(null)

		//取得當下位置
		function getLocation() {
			mymap = L.map('map').setView([0, 0], 13)
			if (navigator.geolocation) {
				navigator.geolocation.getCurrentPosition(
					function (position) {
						const lng = position.coords.longitude; //經度
						const lat = position.coords.latitude; //緯度
						mymap.setView([lat, lng], 13); //注意順序!!

						L.tileLayer(
							"https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
							{
								attribution:
									'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
								maxZoom: 18,
								id: "mapbox/streets-v11",
								tileSize: 512,
								zoomOffset: -1,
								accessToken: import.meta.env.VITE_APP_ACCESSTOKEN,
							}
						).addTo(mymap);
						let marker = L.marker([lat, lng]).addTo(mymap);
						marker.bindPopup('<b>You are here!</b>')
						getStation(lat, lng)
					},
					function (e) {
						const msg = e.code
						const dd = e.message
						console.error(msg)
						console.error(dd)
					}
				)
			}
		}

		//憑證
        function getAuthorizationHeader() {
            let AppID = import.meta.env.VITE_APP_ID;
            let AppKey = import.meta.env.VITE_APP_KEY;

            let GMTString = new Date().toGMTString();
            let ShaObj = new jsSHA('SHA-1', 'TEXT');
            ShaObj.setHMACKey(AppKey, 'TEXT');
            ShaObj.update('x-date: ' + GMTString);
            let HMAC = ShaObj.getHMAC('B64');
            let Authorization = 'hmac username=\"' + AppID + '\", algorithm=\"hmac-sha1\", headers=\"x-date\", signature=\"' + HMAC + '\"';
            return { 'Authorization': Authorization, 'X-Date': GMTString }; 
        }

		//get station data
		function getStation(lat:number, lng:number) {
			fetch(`https://ptx.transportdata.tw/MOTC/v2/Bike/Station/NearBy?&format=JSON&$spatialFilter=nearby(${lat},${lng}, 300)`,{
				headers: getAuthorizationHeader()
			})
			.then(res => res.json())
			.then(data => {
				console.log(data)
			})
		}


		onMounted(() => {
			getLocation();
		});

		return {
			stationData
		};
	},
};
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	width: 100%;
	/* margin-top: 60px; */
}
</style>
