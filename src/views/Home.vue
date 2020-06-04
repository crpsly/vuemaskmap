<template>
  <div>
    <div class="row no-gutters">
    <div class="col-lg-5">
    <div class="toolbox">
    <div class="sticky-top">
    <div class="form-group d-flex">
    <select id="cityName" class="form-control mr-1" @change="changecounty" 
    v-model="selectcounty">
      <option value disabled selected hidden>請選擇縣市</option>
      <option :value="item" v-for="item in showcounty" :key="item">{{item}}</option>
    </select>
    <select id="area" class="form-control" v-model="selectown" @change="changetown">
    <option value disabled selected hidden>請選擇區域</option>
    <option :value="item" v-for="item in townfilter" :key="item">{{item}}</option>
    </select>
    </div>
    <div class="form-group d-flex search-area">
      <input type="text" placeholder="輸入地址" class="form-control" @keyup.enter="gosearch" @click="searchword = ''" v-model.trim="searchword">
      <button class='btn' @click='gosearch' >go!</button>
      <!--<ul class="form-group text-left search" v-if="searchword !==''">
        <li v-if="filterSearch == ''" class="text-center">查無資料</li>
        <a v-for="(item,i) in filterSearch" :key="i"  @click="movetostore(item)">
          <li>{{ item.address }}</li>
        </a>
      </ul>-->
    </div>
    
      <div class="text-left">
        <h3 class="title">口罩實名制3.0 </h3><span>購買日 </span><router-link
  to="/about"><img src="../assets/ic_help.png"></router-link>
        <p class="mb-2 small text-muted">資料更新時間：{{updatedtime}}</p>
      </div>
    </div>
    <ul class="list-group">
    <a class="list-group-item text-left" @click="movetostore(item)" v-for="(item,i) in datafilter.slice(0,shownum)" :key="i">
    <div class="masknum_area">
      <div class="maskbox" 
      :class="[{'full':item.maskadult > 20},{'few':item.maskadult < 20},{'none':item.maskadult == 0}]">
        <p>成人口罩數量</p>
        <p class="mb-0"><span class="masknum">{{item.maskadult}}</span>    片</p>
      </div>
      <div class="maskbox"
      :class="[{'full':item.maskchild > 20},{'few':item.maskchild <= 20},{'none':item.maskchild == 0}]">
        <p>兒童口罩數量</p>
        <p class="mb-0"><span class="masknum">{{item.maskchild}}</span>    片</p>
      </div>
    </div>
    <h3 class="storetitle">{{item.name}}</h3>
    <p >地址：{{item.address}}<a class="small ml-3" :href="`https://www.google.com.tw/maps/place/${item.address}`" target="_blank" title="Google Map">
    地圖查看</a>
    </p>
    <p>電話：{{item.phone}}</p>
    <p>備註：{{item.note}}</p>
    </a>
    <p>尚餘{{datafilter.length > shownum ? datafilter.length - shownum : 0}}筆</p>
    <a class="more btn" @click="shownum+=5" v-if="datafilter.length > shownum">查看更多</a>
    </ul>
    </div>
    </div>
    <div class="col-lg-7">
      <div class="mapbox">
        <div id="map"></div>
      </div>
    </div>
    </div>
    
  </div>
</template>

<script>
// @ is an alias to /src
import L from 'leaflet';
import "leaflet.markercluster/dist/MarkerCluster.css";
import "leaflet.markercluster/dist/MarkerCluster.Default.css";
import "leaflet.markercluster/dist/leaflet.markercluster";

let osmMap = {};

export default {
  name: 'App',
  data: () =>({
    data: [],
    datafilter: [],
    countydata:[],
    selectcounty:'',
    selectown:'',
    towndata:[],
    townfilter:[],
    shownum: 10,
    updatedtime:'',
    searchword: '',
  }),
  methods: {
    changecounty(){
      console.log(this.selectcounty)
      let vm = this;
      vm.datafilter = vm.data;
      let filcondata = this.data.filter(function(item) {
        return item.address.indexOf(vm.selectcounty) != -1;
      }).sort(function(a,b){
        return b.maskadult - a.maskadult;
      })
      console.log(filcondata)
      vm.datafilter = filcondata;

      let newtown = []
      filcondata.forEach(item => {
        newtown.push(item.town)
      });
      this.townfilter = Array.from( new Set(newtown) );
      this.townfilter.splice(this.townfilter.indexOf(""),1);
      console.log(this.datafilter)
      osmMap.panTo([this.datafilter[0].lat, this.datafilter[0].lon]);
    },
    changetown(){
      let vm = this;
      vm.datafilter = vm.data;
      let filcondata = this.data.filter(function(item) {
        return item.address.indexOf(vm.selectcounty) != -1;
      }).sort(function(a,b){
        return b.maskadult - a.maskadult;
      }).filter(function(item) {
        return item.address.indexOf(vm.selectown) != -1;
      })
      vm.datafilter = filcondata;
      osmMap.panTo([this.datafilter[0].lat, this.datafilter[0].lon]);
    },
    movetostore(item){
      osmMap.panTo([item.lat,item.lon]);
      this.searchword='';
    },
    gosearch(){
      let vm = this;
      vm.datafilter = vm.data;
      let searchfilter = vm.datafilter.filter(item => item.address.match(this.searchword));
      vm.datafilter = searchfilter;
    },

  },
  computed:{
    showcounty: function () {
      let countyselect = Array.from( new Set(this.countydata) );
      countyselect.splice(countyselect.indexOf(""),1);
      return countyselect;
    },
    //filterSearch: function(){
    //  return this.datafilter.filter(item => item.address.match(this.searchword));
    //}
  },
  mounted(){
    let vm = this;
    osmMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 18,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 18,
    }).addTo(osmMap);



    let markers = new L.markerClusterGroup().addTo(osmMap);//座標效能優化插件


    //icon set
    let muchIcon = new L.Icon({
      iconUrl: 'https://upload.cc/i1/2020/06/04/TD2uYm.png',
      shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowSize: [41, 41]
    });
    let lessIcon = new L.Icon({
      iconUrl: 'https://upload.cc/i1/2020/06/04/gQSNDV.png',
      shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowSize: [41, 41]
    });
    let noneIcon = new L.Icon({
      iconUrl: 'https://upload.cc/i1/2020/06/04/Z14c9I.png',
      shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowSize: [41, 41]
    });


    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
    this.$http.get(url).then((response) => {
      let dataary = response.data.features;
      //因為是非同步，所以console要放在裡面。最後才會執行ajax這行
      console.log(dataary)
      this.updatedtime = dataary[0].properties.updated;
      dataary.forEach(item => {
        let ary = {};
        let countyary = {};
        let townary = {};
        ary.name = item.properties.name;
        ary.address = item.properties.address;
        ary.phone = item.properties.phone;
        //ary.updated = item.properties.updated;
        ary.id = item.properties.id;
        ary.note = item.properties.note;
        ary.maskadult = item.properties["mask_adult"];
        ary.maskchild = item.properties["mask_child"];
        ary.town = item.properties.town;
        ary.lat = item.geometry.coordinates[1];
        ary.lon = item.geometry.coordinates[0];

        countyary = item.properties.county;
        townary = item.properties.town;
  
        vm.data.push(ary);
        vm.datafilter.push(ary);
        vm.countydata.push(countyary);
        vm.towndata.push(townary);
      });
      
      
      for(let i = 0; i< this.data.length;i++){
        let mask;
        let numclass;
        let numclass_child;
        if(vm.data[i].maskadult === 0){
          mask = noneIcon;
          numclass = "nonemap";
        }else if(vm.data[i].maskadult <= 20){
          mask = lessIcon;
          numclass = "fewmap";
        }else{
          mask = muchIcon;
          numclass = "fullmap";
        }
        if(vm.data[i].maskchild === 0){
          numclass_child = "nonemap";
        }else if(vm.data[i].maskchild <= 20){
          numclass_child = "fewmap";
        }else{
          numclass_child = "fullmap";
        }
        //console.log(vm.data[i].maskadult);
        markers.addLayer(L.marker([vm.data[i].lat,vm.data[i].lon],
         {icon: mask}).bindPopup(`<div class="mapdetail">${vm.data[i].name}<br>${vm.data[i].address}<br><div class="mapbox ${numclass}">${vm.data[i].maskadult} </div><div class="mapbox ${numclass_child}">${vm.data[i].maskchild}</div></div>`));
      }
      
      //bindPopup已經有圖標了所以只是在加入文字popup
    // add more markers here...
      // L.marker().addTo(map)  
      osmMap.addLayer(markers);
       
    });
  },
}
</script>

<style lang="scss">
//導入bootstrap
@import 'bootstrap/scss/bootstrap';

$fullcolor: #11787a;
$hovercolor: #1da4a7;
$fewcolor: #e67e22;
$nonecolor: #d4d6d7;
$maincolor: #34495e;

a{
  color: $maincolor;
  &:hover{
    color: $hovercolor;
  }
}
.btn{
  &:hover{
    color: $hovercolor;
  }
}
#map {
 height: 80vh;
}
.mapbox{
  padding-right: 40px;
}
.leaflet-bottom.leaflet-right{
  display: none;
}
.toolbox {
 height: 90vh;
 background-color: #f8f9fa;
 padding: 10px 40px;
 a {
  cursor: pointer;
 }
 .title{
    font-weight: bold;
    display: inline-block;
    margin-right: 7px;
  }
  img{
    width: 20px;

  }
}
.list-group{
  overflow-y: auto;
  height: 57vh;
  padding: 10px 25px 0px;
}
.sticky-top{
  padding: 0 25px;
}
.list-group-item{
  border: none;
  margin-bottom: 20px;
  border-radius: 20px;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
  font-size: 14px;
  .storetitle{
    margin-top: 5px;
    font-size: 16px;
    color: $maincolor;
    font-weight: bold;
    position: relative;
    &::before{
      content: '';
      background-color: $fullcolor;
      position: absolute;
      top: 0;
      left: -1.25em;
      width: 5px;
      height: 100%;
      border-radius: 0 5px 5px 0;
    }
  }
  p{
    margin-bottom: 5px;
  }
  
}
.list-group-item:first-child {
    border-top-left-radius: 20px;
    border-top-right-radius: 20px;
}
.masknum_area{
  display: flex;
  justify-content: space-between;
}
.maskbox{
  color: #ffffff;
  padding: 10px;
  border-radius: .5em;
  background-repeat: no-repeat;
  background-position: 103% center;
  font-size: 14px;
  font-weight: bold;
  width: 47%;
  margin-bottom: 10px;
  p{
    margin-bottom: 5px;
  }
  .masknum{
    font-size: 30px;
    margin-right: 5px;
  }
}
.full{
  background-color: $fullcolor;
  background-image: url(../assets/ic_stock_full.png);
}
.few{
  background-color: $fewcolor;
  background-image: url(../assets/ic_stock_few.png);
}
.none{
  background-color: $nonecolor;
  background-image: url(../assets/ic_stock_none.png);
}
.more{
  background: $maincolor;
  color:#ffffff !important;
  border-radius: 50px;
  width: 60%;
  display: block;
  margin: 0 auto;
  margin-bottom: 20px;
  box-shadow: 0 0 10px rgba($maincolor,0.6);
}
li{
  list-style: none;
}
li.text-center{
  margin-top: 10px
}
.search-area{
  position: relative;
  .search{
    position: absolute;
    top: 100%;
    left: 0;
    background: #fff;
    overflow-y: auto;
    height: 40vh;
    padding: 0;
    width: 100%;
    a{
      padding: 5px 10px;
      display: inline-block;
    }
  }
}
.leaflet-popup-content{
  margin: 10px 10px;
}
.mapdetail{
  color: $maincolor;
  font-weight: bold;
  text-align: center;
  .mapbox{
    display: inline-block;
    background-color: yellow;
    margin: 3px 3px;
    text-align: center;
    border-radius: 5px;
    padding: 5px;
    color: #fff;
    font-size: 15px;
  }
  .mapbox.fullmap{
    background-color: $fullcolor;
  }
  .mapbox.fewmap{
    background-color: $fewcolor;
  }
  .mapbox.nonemap{
    background-color: $nonecolor;
  }
}
@media (max-width: 768px){
  #map{
    display: none;
  }
  .toolbox{
      padding: 10px 10px;
  }
}
</style>
