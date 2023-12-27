<template>
  <div>
    <el-tree :data="treeData" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
             :default-expanded-keys="expandedKey"
             :draggable="false"
             :allow-drop="allowDrop"

    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(node,data)">
            Append
          </el-button>

          <el-button
            type="text"
            size="mini"
            @click="() => edit(node,data)">
            edit
          </el-button>
          <el-button
            v-if="node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node,data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      title="添加分类"
      :visible.sync="dialogVisible"
      width="35%"
      :close-on-click-modal="false"
    >
      <el-form :inline="true" :model="formData" ref="ruleForm" :rules="rules" v-loading="isLoading" label-width="100px">
        <el-row :span="12">
          <el-form-item label="分类名称" prop="name">
            <el-input v-model="formData.name" placeholder="请输入" ></el-input>
          </el-form-item>
          <el-form-item label="图标" prop="icon">
            <el-input v-model="formData.icon" placeholder="请输入" ></el-input>
          </el-form-item>
          <el-form-item label="计量单位" prop="productUnit">
            <el-input v-model="formData.productUnit" placeholder="请输入" ></el-input>
          </el-form-item>
        </el-row>
      </el-form>
        <span slot="footer" class="dialog-foot" >
          <el-button type="primary" @click="onSubmit">确定</el-button>
          <el-button type="text" @click="dialogVisible=false">取消</el-button>
        </span>

    </el-dialog>
  </div>
</template>
<script>

export default {
  name: 'category',
  props: {},
  components: {},
  mounted() {
    this.getTreeData()
  },
  data() {
    return {
      maxLevel:1,
      title: '',
      isLoading:false,
      dialogVisible: false,
      formData: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productCount: 0,
        productUnit:'',
        icon: '',
      },
      rules: {
        name: [
          {required: true, message: '请输入分类名称', trigger: 'blur'},
        ],
      },
      expandedKey: [],
      treeData: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    allowDrop(draggingNode, dropNode, type) {
      // 被拖拽的当前节点以及所在的父节点总层数不能大于3层
      console.log("draggingNode", draggingNode, "dropNode", dropNode, "type", type)
      this.countNodeLevel(draggingNode.data)
      //当前正在被拖动的节点+父节点的层数不能大于3层
      let deep=(this.maxLevel-draggingNode.data.catLevel)+1
      console.log("level",deep)
      if(type=="inner"){
        return deep+dropNode.level<=3;
      }else {
        return deep+dropNode.parent.level<=3;
      }
    },
    countNodeLevel(node){
      //找到所有子节点，求出最大深度
      if(node.children!=null&&node.children.length>0){
        for (let i = 0; i < node.children.length.length; i++) {
          if(node.children[i].catLevel>this.maxLevel){
            this.maxLevel=node.children[i].catLevel
          }
          this.countNodeLevel(node.children[i])

        }
      }

    },
    onSubmit() {
      this.isLoading=true
      console.log('submit!');
      let url;
      url=this.title=="新增分类"?'/product/category/save':'/product/category/update'
      this.$refs['ruleForm'].validate((valid) => {
        if (valid) {
          console.log("this.formData", this.formData)
          this.$http({
            url: this.$http.adornUrl(url),
            method: 'post',
            data: this.$http.adornData(this.formData, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: this.title=="新增分类"?"添加成功":"修改成功",
                type: 'success',
                duration: 1500,
              })
              this.dialogVisible = false
              this.getTreeData()
              this.expandedKey = [this.formData.parentCid]
            } else {
              this.$message.error(data.msg)
            }
          })
        } else {
          console.log('error submit!!');
          return false;
        }
      });
      this.isLoading=false
    },
    edit(node,data){
      this.title="修改分类"
      console.log("edit",data)
      this.dialogVisible = true
      this.isLoading=true
      this.$nextTick(() => {
        this.$refs.ruleForm.clearValidate()
        this.$http({
          url: this.$http.adornUrl('/product/category/info/'+data.catId),
          method: 'get',
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log("data",data)
            this.formData=data.category
          } else {
            this.$message.error(data.msg)
          }
        })
      })
      this.isLoading=false
    },
    append(node, data) {
      Object.keys(this.formData).forEach(key => {
        this.formData[key] = ''
      })
      this.title="新增分类"
      console.log("node", node, "data", data)
      this.dialogVisible = true
      this.formData.parentCid = data.catId
      this.formData.catLevel = data.catLevel * 1+1
      this.formData.name=''
      this.$nextTick(() => {
        this.$refs.ruleForm.clearValidate()
      })


    },
    remove(node, data) {
      console.log("data", data, "node", node)
      var ids = [data.catId]
      console.log("node", node)
      this.$confirm(`确定是否删除${data.name}菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {

            this.$message({
              message: '菜单删除成功',
              type: 'success',
              duration: 1500,
            })
            this.getTreeData()
            this.expandedKey = [node.parent.data.catId]
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {
        console.log("取消删除")
      })

    },
    getTreeData() {
      this.$http({
        url: this.$http.adornUrl('/product/category/listTree'),
        method: 'get',
      }).then(({data}) => {
        this.treeData = data.data
        console.log("成功获取到分类数据", data.data)
      })
    },
  },
  watch: {},
  computed: {}
}
</script>
<style scoped lang="scss">
  ::v-deep .el-dialog__body {
    width: 150px;
    height: 100px;
  }
</style>
