<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="對應的用戶id" prop="userid">
      <el-input v-model="dataForm.userid" placeholder="對應的用戶id"></el-input>
    </el-form-item>
    <el-form-item label="此用戶的信息" prop="msg">
      <el-input v-model="dataForm.msg" placeholder="此用戶的信息"></el-input>
    </el-form-item>
    <el-form-item label="此消息的状态,0代表已读，1代表未读" prop="status">
      <el-input v-model="dataForm.status" placeholder="此消息的状态,0代表已读，1代表未读"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          userid: '',
          msg: '',
          status: ''
        },
        dataRule: {
          userid: [
            { required: true, message: '對應的用戶id不能为空', trigger: 'blur' }
          ],
          msg: [
            { required: true, message: '此用戶的信息不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '此消息的状态,0代表已读，1代表未读不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/ums/lyftousermsg/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userid = data.lyfTousermsg.userid
                this.dataForm.msg = data.lyfTousermsg.msg
                this.dataForm.status = data.lyfTousermsg.status
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
              url: this.$http.adornUrl(`/ums/lyftousermsg/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'userid': this.dataForm.userid,
                'msg': this.dataForm.msg,
                'status': this.dataForm.status
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
      }
    }
  }
</script>
