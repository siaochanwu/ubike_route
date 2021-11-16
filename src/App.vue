<template>
	<div class="container mx-auto relative">
		<h1>www</h1>
		<div id="map" class="h-96 z-0">
		</div>
		<div class="bg-red-200 z-10 h-40 w-40 absolute top-0 left-0 transition-all duration-300" :class="{ '-ml-40': !sideOpen }">
			<div class="absolute top-1/2 -right-5 transform -translate-y-1/2 animate-pulse" @click="sideOpen = !sideOpen">>>></div>
			<div>
				<h1>地區選擇</h1>
				<select name="" id="" v-model="selectCountry">
					<option :value="item.value"  v-for="item in country" :key="item.value">{{ item.name }}</option>
				</select>
				<h1>路線選擇</h1>
				<select name="" id="" class="w-10/12" v-model="selectRoute">
					<option :value="item.RouteName" v-for="item in route" :key="item.AuthorityName">{{ item.RouteName }}</option>
				</select>
			</div>
		</div>
	</div>
</template>

<script lang="ts">
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import jsSHA from "jssha";
import Wkt from 'wicket'
import { ref, onMounted, watch } from "vue";

export default {
	setup() {
		interface data {
			StationUID: string,
			StationName: {
				Zh_tw: string
			},
			StationAddress: {
				Zh_tw: string
			},
			StationPosition: {
				PositionLat: number,
				PositionLon: number
			},
			AvailableRentBikes: number,
			AvailableReturnBikes: number
		}
		interface routeData {
			AuthorityName: string,
			RouteName: string,
			Geometry: string
		}

		const stationData = ref<data[]>([])
		const availableBike = ref<data[]>([])
		const filterData = ref<data[]>([])
		const sideOpen = ref(true)
		const country = ref([
			{
                name: "臺中市",
                value: "Taichung"
            },
			{
                name: "基隆市",
                value: "Keelung"
            },
			{
                name: "新竹縣",
                value: "HsinchuCounty"
            },
			{
                name: "苗栗縣",
                value: "MiaoliCounty"
            },
			{
                name: "彰化縣",
                value: "ChanghuaCounty"
            },
			{
                name: "新北市",
                value: "NewTaipei"
            },
			{
                name: "南投縣",
                value: "NantouCounty"
            },
			{
                name: "雲林縣",
                value: "YunlinCounty"
            },
			{
                name: "嘉義縣",
                value: "ChiayiCounty"
            },
			{
                name: "嘉義市",
                value: "Chiayi"
            },
			{
                name: "屏東縣",
                value: "PingtungCounty"
            },
			{
                name: "宜蘭縣",
                value: "YilanCounty"
            },
			{
                name: "花蓮縣",
                value: "HualienCounty"
            },
			{
                name: "臺東縣",
                value: "TaitungCounty"
            },
			{
                name: "金門縣",
                value: "KinmenCounty"
            },
			{
                name: "澎湖縣",
                value: "PenghuCounty"
            },
			{
                name: "桃園市",
                value: "Taoyuan"
            },
			{
                name: "臺北市",
                value: "Taipei"
            },
            {
                name: "臺南市",
                value: "Tainan"
            },
            {
                name: "高雄市",
                value: "Kaohsiung"
            },
		])
		const selectCountry = ref('')
		const route = ref<routeData[]>([])
		const selectRoute = ref('')
		let mymap = ref<any>('')
		let myLayer = ref<any>(null)

		//取得當下位置
		function getLocation() {
			mymap = L.map('map').setView([0, 0], 15)
			if (navigator.geolocation) {
				navigator.geolocation.getCurrentPosition(
					function (position) {
						const lng = position.coords.longitude; //經度
						const lat = position.coords.latitude; //緯度
						mymap.setView([lat, lng], 15); //注意順序!!

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
						getAvailableBike(lat, lng)
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
            let AppKey: any = import.meta.env.VITE_APP_KEY;

            let GMTString = new Date().toUTCString();
            let ShaObj = new jsSHA('SHA-1', 'TEXT');
            ShaObj.setHMACKey(AppKey, 'TEXT');
            ShaObj.update('x-date: ' + GMTString);
            let HMAC = ShaObj.getHMAC('B64');
            let Authorization = 'hmac username=\"' + AppID + '\", algorithm=\"hmac-sha1\", headers=\"x-date\", signature=\"' + HMAC + '\"';
            return { 'Authorization': Authorization, 'X-Date': GMTString }; 
        }

		//get station data
		function getStation(lat:number, lng:number) {
			fetch(`https://ptx.transportdata.tw/MOTC/v2/Bike/Station/NearBy?&format=JSON&$spatialFilter=nearby(${lat},${lng}, 500)`,{
				headers: getAuthorizationHeader()
			})
			.then(res => res.json())
			.then(data => {
				// console.log(data)
				stationData.value = data
			})
		}

		//get available data
		function getAvailableBike(lat:number, lng:number) {
			fetch(`https://ptx.transportdata.tw/MOTC/v2/Bike/Availability/NearBy?&format=JSON&$spatialFilter=nearby(${lat},${lng}, 500)`,{
				headers: getAuthorizationHeader()
			})
			.then(res => res.json())
			.then(data => {
				// console.log(data)
				availableBike.value = data
				for (let i = 0; i < availableBike.value.length; i++) {
					for (let j = 0; j < stationData.value.length; j++) {
						if (availableBike.value[i].StationUID == stationData.value[j].StationUID) {
							availableBike.value[i].StationName = stationData.value[j].StationName;
							availableBike.value[i].StationAddress = stationData.value[j].StationAddress;
							availableBike.value[i].StationPosition = stationData.value[j].StationPosition;
							filterData.value.push(availableBike.value[i])
						}
					}
				}
				console.log('filter:', filterData.value)
				setMarker(filterData.value)
			})
		}

		//pin marker
		function setMarker(data:data[]) {
			console.log(data)
			var myIcon = L.icon({ //修改marker樣式
				iconUrl: 'https://yama.tw/wp-content/uploads/20190917100615_25.png',
				iconSize: [38, 95],
				iconAnchor: [22, 94],
				popupAnchor: [-3, -76],
				// shadowUrl: 'my-icon-shadow.png',
				shadowSize: [68, 95],
				shadowAnchor: [22, 94]
			});
			data.forEach(item => {
				let marker = L.marker([item.StationPosition.PositionLat, item.StationPosition.PositionLon], {icon: myIcon}).addTo(mymap);
				marker.bindPopup(`
					<b class="text-xl">${item.StationName.Zh_tw}</b><br>
					<b>地點:${item.StationAddress.Zh_tw}</b><br>
					<b class="text-red-600">可租借數量:${item.AvailableRentBikes}</b><br>
					<b class="text-red-600">可租借數量:${item.AvailableReturnBikes}</b>
				`)
			})
		}

		//get route data
		function getRoute(selectCountry:string) {
			fetch(`https://ptx.transportdata.tw/MOTC/v2/Cycling/Shape/${selectCountry}?&format=JSON`,{
				headers: getAuthorizationHeader()
			})
			.then(res => res.json())
			.then(data => {
				console.log(data)
				route.value = data
			})
		}

		//showRoute
		function showRoute(selectRoute:string) {
			route.value.forEach(item => {
				if(item.RouteName === selectRoute) {
					var geo = item.Geometry
					polygon(geo)
				}
			})
		}

		//draw polygon
		function polygon(geo:string) {
			mymap.removeLayer(myLayer)
			console.log(typeof myLayer)
			const wicket = new Wkt.Wkt();
			const geojsonFeature = wicket.read(geo).toJson()
			const myStyle = {
				"color": "#ff0000",
				"weight": 5,
				"opacity": 0.65
			};
			myLayer = L.geoJSON(geojsonFeature, {
				style: myStyle
			}).addTo(mymap);

			myLayer.addData(geojsonFeature)
			mymap.fitBounds(myLayer.getBounds());
		}

		watch(selectCountry, (newVal, oldVal) => {
            if (newVal !== '') {
				getRoute(selectCountry.value)
			}
        })

		watch(selectRoute, (newVal, oldVal) => {
            if (newVal !== '') {
				showRoute(selectRoute.value)
			}
        })

		onMounted(() => {
			getLocation();
		});

		return {
			stationData,
			availableBike,
			filterData,
			sideOpen,
			country,
			selectCountry,
			route,
			selectRoute,
			showRoute,
			polygon,
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
