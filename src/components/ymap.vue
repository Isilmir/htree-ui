<template>

  <yandex-map 
    :coords="baseCoords"
    :zoom="10" 
    @click="onClick"
  >
    <div v-for="marker in baloons">
    <ymap-marker 
      :coords="marker.coords" 
      :icon="marker.icon"
      marker-id="123" 
      :hint-content="marker.date" 
    />
  </div>
  <b-modal v-model="addDialogActive" @close="onCloseDialogActive">
            
            <b-input style="min-width:170px; max-width:170px; margin-left:37%" placeholder="описание точки" v-model="newPoint_markerText" :controls="false" :disabled="false"></b-input>
            <br>
            <b-select placeholder="Тип точки" v-model="newPoint_markerType" style="min-width:200px">
                      <option
                          v-for="option in [{id:'blockHeavyArmament',name:'Тяжелое вооружение'}
                                           ,{id:'blockAres',name:'Живая сила'}
                                           ,{id:'blockLicense',name:'Техника'}
                                           ,{id:'OlympicGamesWinner',name:'Укрепления'}]"
                          :value="option.id"
                          :key="option.id">
                          {{ option.name }}
                      </option>
                  </b-select><br>
            <b-button @click="addMarker" type="is-warning" >Подтвердить</b-button>
                  
          </b-modal>
  </yandex-map>
</template>

<script>
import axios from 'axios';
import { yandexMap, ymapMarker } from 'vue-yandex-maps'

export default {
  name: 'ymap',
  data () {
    return {
      coords: [
      54.82896654088406,
      39.831893822753904,
    ],
    baseCoords:[
      54.82896654088406,
      39.831893822753904
    ],
    baloons:[],
    addDialogActive:false,
    newPoint_markerText:'',
    newPoint_coords:null,
    newPoint_markerType:null,
    socket : new WebSocket("ws://192.168.0.148:3000/")
    }
  },
  async mounted(){
    let thus=this;
    this.socket.onmessage = function(event){
      console.log('from socket',JSON.parse(event.data))
      thus.baloons=JSON.parse(event.data);
      console.log('baloons',thus.baloons)
    //let message=document.createElement('DIV');
    //message.innerHTML=event.data;
    //document.body.appendChild(message);
    }
    this.socket.onclose = function(event) {
      if (event.wasClean) {
        alert(`[close] Соединение закрыто чисто, код=${event.code} причина=${event.reason}`);
      } else {
        // например, сервер убил процесс или сеть недоступна
        // обычно в этом случае event.code 1006
        alert(`[close] Соединение прервано ${event.code}`);
      }
    };
    //console.log(this.socket)
  },
  methods:{
	  onClick(e) {
      this.coords = e.get('coords');
      console.log('координаты точки',e.get('coords'))
      this.newPoint_coords=e.get('coords');
      this.newPoint_markerText='';
      this.addDialogActive=true;
      /*this.baloons.push({coords:e.get('coords')
                        ,icon: {
                        layout: 'default#imageWithContent',
                        imageHref: this.getImg('blockHeavyArmament'),
                        imageSize: [44, 44],
                        imageOffset: [-22, -22],
                        content: 'some text',
                        contentOffset: [0, 15],
                        contentLayout: '<div style="width: 50px; color: #00FFFF; font-weight: bold;">$[properties.iconContent]</div>'
    }})*/
      console.log('точки',this.baloons)
    },
    getImg(deedGroup){
      let res
      try{
        res=require(`../assets/deeds/${deedGroup}.png`);
      }
      catch(e){
        res=require(`../assets/deeds/feat.png`);
      }
      return res
	  },
    async onCloseDialogActive(){
      console.log('закрыли окно добавления точки')
    },
    addMarker(){
      console.log('добавляем маркер')
      this.baloons.push({coords:this.newPoint_coords
                        ,icon: {
                          layout: 'default#imageWithContent',
                          imageHref: this.getImg(this.newPoint_markerType),
                          imageSize: [44, 44],
                          imageOffset: [-22, -22],
                          content: this.newPoint_markerText,
                          contentOffset: [0, 15],
                          contentLayout: '<div style="width: 50px; color: #005555; font-weight: bold;">$[properties.iconContent]</div>'
                        }
                        ,date:(new Date())+''
                      })
    this.addDialogActive=false;
    this.socket.send(JSON.stringify(this.baloons))
    }
  },
  components:{
    yandexMap, ymapMarker
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
