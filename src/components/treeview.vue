<template>
  <div class="greetings">
    <h1 class="green">Смотрим дерево</h1>
    <h3>
      {{treeData.data}}
    </h3>
    <tree v-for="localtree in forest" :data="localtree" node-text="name" layoutType="vertical" :duration="0" :zoomable="true" height="100">
      <template #node="{data, node: {depth}, radius, isRetracted}">
        
        <template v-if="data.children && data.children.length">
          <circle r="6" :stroke="'red'">
            <title>{{data.data.structureid}}</title>
            <!--<text dy=".35em" :x="0" :dx="0" :y="-15" text-anchor="middle" transform="rotate(-90)">{{data.data.name}}</text>-->
          </circle>
          <!--<text dy=".35em" :x="0" :dx="0" :y="-15" text-anchor="start" transform="rotate(-90)">{{data.data.name}}</text>-->
        </template>
        <template v-else>
          <circle r="6" :stroke="'yellow'">
            <title>{{data.data.structureid}}</title>
            <!--<text></text>-->
          </circle>
          <!--<text dy=".35em" :x="6" :dx="0" :y="0" text-anchor="start" transform="rotate(0)">{{data.data.name}}</text>-->
          
        </template>
        
      </template>
      <template #popUp="{data,node}">
        <div class="btn-group-vertical">
          <button @click="onAddClick(data)">
            <i class="fa fa-plus" aria-hidden="true">добавить</i>
          </button>
          <button  v-if="!data.children" @click="deleteNode(data)">
            <i class="fa fa-trash" aria-hidden="true">удалить</i>
          </button>
        </div>
      </template>
    </tree>
    <b-modal v-model="addDialogActive" @close="onCloseAddDialogActive">
            
            <b-input style="min-width:170px; max-width:170px; margin-left:37%" placeholder="название" v-model="newNodeName" :controls="false" :disabled="false"></b-input>
            <br>
            <b-button @click="addNode" type="is-warning" >Подтвердить</b-button>
                  
    </b-modal>
    <b-button @click="getTree" type="is-warning" >Обновить дерево</b-button>
  </div>
</template>

<script>
import axios from 'axios';
  import {stratify,hierarchy} from 'd3-hierarchy'
  import {tree} from 'vued3tree'

export default {
  name: 'test',
  data () {
    return {
      forest:null,
      treeData:null,
      addDialogActive:false,
      newNodeName:''
    }
  },
  async mounted(){
	await this.getTree();
  },
  methods:{
	async getTree(){
    this.forest=[]
		let headers = {};
    headers['Content-Type']='application/json'
    //if(localStorage.getItem('jwt'))headers['Authorization']=`Bearer ${localStorage.getItem('jwt').replace(/"/g,'')}`
    let tree__;
			try{
        tree__ = await axios.get('http://localhost:5000/pg/tree/1',//'/pg/getTree',
			{
				headers: headers
			});
			}catch(e){
				console.log(e.message);
			}
      console.log('tree__.data',tree__.data)
      let preparedTree = stratify()
      .id(function(d) { return d.id; })
      .parentId(function(d) { return d.parentid; })
      (tree__.data);
      console.log('preparedTree',preparedTree)
      let updatedPreparedTree=this.upadateTree(preparedTree);
      console.log('upadateTree(preparedTree)',updatedPreparedTree)
      //console.log('preparedTree.each',preparedTree.each(d=>{console.log(d.data.name);return{...d,name:d.data.name}}))
      //console.log('hierarchy',hierarchy(preparedTree,d=>d.chiildren))
      console.log('this.treeData before;',this.treeData)
      this.treeData=null;
      console.log('this.treeData=null;',this.treeData)
			this.treeData=updatedPreparedTree;
      console.log('this.treeData=updatedPreparedTree;',this.treeData)
    
      this.forest=[updatedPreparedTree]
      console.log('this.forest=[updatedPreparedTree]',this.forest)
	  },
    editNode(e,d,t){
        console.log(e,d,t)
    },
    async onCloseAddDialogActive(){
      console.log('закрыли окно добавления ноды')
    },
    async addNode(e,d,t){
      let headers = {};
      headers['Content-Type']='application/json'
        console.log('addNode',e,this.newNodeStructureid,this.newNodeParentid,this.newNodeName)
        try{
        tree__ = await axios.post('http://localhost:5000/pg/tree/node',{
            structureid:this.newNodeStructureid,
            parentid: this.newNodeParentid,
            name:this.newNodeName
				},
        {
          headers: headers
        });
        }catch(e){
          console.log('Ошибка!',e.message);
        }
        this.addDialogActive=false;
      
        await this.getTree();
    },
    async deleteNode(e,d,t){
      let headers = {};
      headers['Content-Type']='application/json'
        console.log('deleteNode',e)
        try{
          response = await axios.delete(`http://localhost:5000/pg/tree/node/${e.id}`,
				{
					headers: {
					  'Content-Type': 'application/json',
					},
					data: {
					}
				});
        }catch(e){
          console.log('Ошибка!',e.message);
        }
        this.addDialogActive=false;
      
        await this.getTree();
    },
    upadateTree(root){
      //console.log('upadateTree(root)',root)
      let res=root
      res.name=res.data.name
      if(res.children){
        res.children=res.children.map(child=>{return this.upadateTree(child)})
      }
      return res;
    },
    onAddClick(e) {
      console.log('добавляем новую ноду')
      this.newNodeName='';
      this.addDialogActive=true;
      this.newNodeStructureid=e.data.structureid
      this.newNodeParentid=e.data.id
      console.log('точки',this.baloons)
    }
  },
  components:{
    tree
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
