<template>
  <div class="greetings">
    <h1 class="green">Смотрим дерево</h1>
    <h3>
      {{treeData.data}}
    </h3>
    <tree :data="treeData" node-text="name" layoutType="vertical" :duration="0" :zoomable="true" height="100"  @clicked="editNode">
      <template #node="{data, node: {depth}, radius, isRetracted}">
        
        <template v-if="data.children && data.children.length">
          <circle r="6" :stroke="'red'">
            <title>{{data.data.name}}</title>
            <!--<text dy=".35em" :x="0" :dx="0" :y="-15" text-anchor="middle" transform="rotate(-90)">{{data.data.name}}</text>-->
          </circle>
          <!--<text dy=".35em" :x="0" :dx="0" :y="-15" text-anchor="start" transform="rotate(-90)">{{data.data.name}}</text>-->
        </template>
        <template v-else>
          <circle r="6" :stroke="'yellow'">
            <title>{{data.data.name}} {{depth}}</title>
            <!--<text></text>-->
          </circle>
          <!--<text dy=".35em" :x="6" :dx="0" :y="0" text-anchor="start" transform="rotate(0)">{{data.data.name}}</text>-->
          
        </template>
        
      </template>
      <template #popUp="{data,node}">
        <div class="btn-group-vertical">
          <button @click="editNode(data)">
            <i class="fa fa-plus" aria-hidden="true">добавить</i>
          </button>
          <button @click="editNode(data, node)">
            <i class="fa fa-trash" aria-hidden="true">удалить</i>
          </button>
        </div>
      </template>
    </tree>

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
      treeData:null
    }
  },
  async mounted(){
	this.getMessage();
  },
  methods:{
	async getMessage(){
		let headers = {};
    headers['Content-Type']='application/json'
    //if(localStorage.getItem('jwt'))headers['Authorization']=`Bearer ${localStorage.getItem('jwt').replace(/"/g,'')}`
    let tree__;
			try{
        tree__ = await axios.get('http://localhost:5000/pg/getTree',
			{
				headers: headers
			});
			}catch(e){
				console.log(e.message);
			}
      console.log(tree__.data)
      let preparedTree = stratify()
      .id(function(d) { return d.id; })
      .parentId(function(d) { return d.parent_id; })
      (tree__.data);
      console.log('preparedTree',preparedTree)
      let updatedPreparedTree=this.upadateTree(preparedTree);
      console.log('upadateTree(preparedTree)',updatedPreparedTree)
      //console.log('preparedTree.each',preparedTree.each(d=>{console.log(d.data.name);return{...d,name:d.data.name}}))
      //console.log('hierarchy',hierarchy(preparedTree,d=>d.chiildren))
			this.treeData=updatedPreparedTree;
	  },
    editNode(e,d,t){
        console.log(e,d,t)
    },
    upadateTree(root){
      //console.log('upadateTree(root)',root)
      let res=root
      res.name=res.data.name
      if(res.children){
        res.children=res.children.map(child=>{return this.upadateTree(child)})
      }
      return res;
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
