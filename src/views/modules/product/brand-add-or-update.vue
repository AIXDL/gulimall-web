<template>
  <el-dialog
    :title="!dataForm.brandId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="120px">
    <el-form-item label="品牌名" prop="name">
      <el-input v-model="dataForm.name" placeholder="品牌名"></el-input>
    </el-form-item>
    <el-form-item label="品牌logo地址" prop="logo">
      <el-upload
        class="upload-demo"
        action="https://aixdl.oss-cn-shenzhen.aliyuncs.com"
        :data="uploadObj"
        :multiple="false"
        list-type="picture"
        :before-upload="beforeUpload"
        :file-list="fileList">
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
    </el-form-item>
    <el-form-item label="介绍" prop="descript">
      <el-input v-model="dataForm.descript" placeholder="介绍"></el-input>
    </el-form-item>
    <el-form-item label="显示状态" prop="showStatus">
<!--      <el-input v-model="dataForm.showStatus" placeholder="显示状态[0-不显示；1-显示]"></el-input>-->
      <el-switch v-model="dataForm.showStatus"></el-switch>
    </el-form-item>
    <el-form-item label="检索首字母" prop="firstLetter">
      <el-input v-model="dataForm.firstLetter" placeholder="检索首字母"></el-input>
    </el-form-item>
    <el-form-item label="排序" prop="sort">
      <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import {getUUID} from "../../../utils";

  export default {
    data () {
      return {
        uploadObj:{
          policy:'',
          signature:'',
          OSSAccessKeyId:'',
          key:'',
          dir:'',
          host:'',
          expire:''
        },
        visible: false,
        fileList: [],
        dataForm: {
          brandId: 0,
          name: '',
          logo: '',
          descript: '',
          showStatus: '',
          firstLetter: '',
          sort: ''
        },
        dataRule: {
          name: [
            { required: true, message: '品牌名不能为空', trigger: 'blur' }
          ],
          logo: [
            { required: true, message: '品牌logo地址不能为空', trigger: 'blur' }
          ],
          descript: [
            { required: true, message: '介绍不能为空', trigger: 'blur' }
          ],
          showStatus: [
            { required: true, message: '显示状态[0-不显示；1-显示]不能为空', trigger: 'blur' }
          ],
          firstLetter: [
            { required: true, message: '检索首字母不能为空', trigger: 'blur' }
          ],
          sort: [
            { required: true, message: '排序不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.brandId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.brandId) {
            this.$http({
              url: this.$http.adornUrl(`/product/brand/info/${this.dataForm.brandId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.brand.name
                this.dataForm.logo = data.brand.logo
                this.dataForm.descript = data.brand.descript
                this.dataForm.showStatus = data.brand.showStatus
                this.dataForm.firstLetter = data.brand.firstLetter
                this.dataForm.sort = data.brand.sort
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/product/brand/${!this.dataForm.brandId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'brandId': this.dataForm.brandId || undefined,
                'name': this.dataForm.name,
                'logo': this.dataForm.logo,
                'descript': this.dataForm.descript,
                'showStatus': this.dataForm.showStatus,
                'firstLetter': this.dataForm.firstLetter,
                'sort': this.dataForm.sort
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
       getBucket(){
        this.$http({
          url: this.$http.adornUrl(`/thirdparty/oss/getBucket`),
          method: 'get'
        }).then(({data})=>{
          console.log("getBucket")

        }).catch(err=>{
          this.$message.error(err)
        })
      },
      getAccessId(){
        this.$http({
          url: this.$http.adornUrl(`/thirdparty/oss/getAccessId`),
          method: 'get'
        }).then(({data})=>{
          console.log("getAccessId")
        }).catch(err=>{
          this.$message.error(err)
        })
      },
      beforeUpload(file){
        console.log("beforeUpload",file)
        return new Promise((resolve, reject) => {
          this.$http({
            url: this.$http.adornUrl(`/thirdparty/oss/policy`),
            method: 'get'
          }).then(({data})=>{
            console.log("data",data.data)
            this.uploadObj = data.data
            this.uploadObj.dir=data.data.dir
            this.uploadObj.host=data.data.host
            this.uploadObj.signature=data.data.signature
            this.uploadObj.policy=data.data.policy
            this.uploadObj.OSSAccessKeyId=data.data.accessid
            this.uploadObj.key = this.uploadObj.dir +getUUID()+ file.name
            this.uploadObj.expire=data.data.expire
            this.dataForm.logo=this.uploadObj.host+"/"+this.uploadObj.key
            resolve(true)
          }).catch(err=>{
            this.$message.error(err)
            reject(false)
          })
        })
        console.log("this.uploadObj",this.uploadObj)
      }
    }
  }
</script>
