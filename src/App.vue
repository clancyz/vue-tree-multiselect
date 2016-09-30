<template>
  <div id="app">
    <pre>
      v-tree-multiselect v-ref:abc :init-data="treeData" level-text="大区,城市,区域" level-key="region_list,city_list,area_list"
    </pre>
    <v-tree-multiselect v-ref:abc :init-data="treeData" level-text="大区,城市,区域" level-key="region_list,city_list,area_list"></v-tree-multiselect>
  </div>
  <div id="result" style="float:right; width:500px;word-break: break-all">
    {{resultData}}
  </div>
</template>

<script>
  import vTreeMultiselect from './components/vue-tree-multiselect'

   export default {
       components: {
           vTreeMultiselect
       },
       data() {
           return {
               treeData: null,
           }
       },
       computed: {
        resultData () {
          return JSON.stringify(this.$refs.abc.outData);
        }
       },
       methods:{
       },
       ready() {
           let json = {
               "errno": 0,
               "errmsg": "",
               "req_params": [],
               "data": {
                   "region_list": [{
                       "key": "1742891",
                       "text": "北二大区",
                       "city_list": [{
                             "key": "58",
                             "text": "沈阳",
                             "area_list":[{
                                "key": "581",
                                "text": "沈阳1"
                             },{
                                "key": "582",
                                "text": "沈阳2"
                             }]
                         }, {
                             "key": "167",
                             "text": "大连",
                             "area_list":[{
                                "key": "1671",
                                "text": "大连1"
                             },{
                                "key": "1672",
                                "text": "大连2"
                             }]
                         }]
                   },
                   {
                       "key": "1742892",
                       "text": "北一大区",
                       "city_list": [{
                             "key": "131",
                             "text": "北京",
                             "area_list":[{
                                "key": "1311",
                                "text": "北京1"
                             },{
                                "key": "582",
                                "text": "北京2"
                             }]
                         }, {
                             "key": "289",
                             "text": "上海",
                             "area_list":[{
                                "key": "2891",
                                "text": "上海1"
                             },{
                                "key": "2892",
                                "text": "上海2"
                             }]
                         }]
                   }
                   ]
               }
           };
           this.$set('treeData',  json.data);
       }
   }

</script>

<style>
html {
  height: 100%;
}

body {
  display: flex;
  height: 100%;
}
#app {
    padding: 30px;
}

</style>
