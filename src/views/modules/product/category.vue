<template>
  <div>
    <el-tree :data="treeData" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

  </div>
</template>
<script>
import axios from "axios";

export default {
  name: 'category',
  props: {},
  components: {},
  mounted() {
    this.getTreeData()
  },
  data() {
    return {
      treeData:[],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    append(data){
      console.log("data",data)
    },
    remove(node){
      console.log("node",node)

    },
    getTreeData(){
      this.$http({
        url: this.$http.adornUrl('/product/category/listTree'),
        method: 'get',
      }).then(({data}) => {
        this.treeData = data.data
        console.log("成功获取到分类数据",data.data)
      })
    },
  },
  watch: {

  },
  computed:{

  }
}
</script>
<style scoped lang="scss">

</style>
