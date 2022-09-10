<template>
  <div class="greetings">
    <h1 class="green">Смотрим дерево</h1>
    <h3>
      {{treeData.data}}
    </h3>
    <tree :data="treeData" node-text="data" layoutType="vertical" :duration="0"></tree>

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
      console.log('hierarchy',hierarchy(preparedTree,d=>d.chiildren))
			this.treeData=preparedTree;
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
