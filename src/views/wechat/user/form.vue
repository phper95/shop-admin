<template>
  <el-dialog :append-to-body="true" :close-on-click-modal="false" :before-close="cancel" :visible.sync="dialog" :title="isAdd ? '新增' : '编辑'" width="500px">
    <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
      <el-form-item label="用户昵称">
        <el-input v-model="form.nickname" :disabled="true" style="width: 370px;" />
      </el-form-item>
      <el-form-item label="真实姓名">
        <el-input v-model="form.real_name" style="width: 370px;" />
      </el-form-item>
      <el-form-item label="用户备注">
        <el-input v-model="form.mark" style="width: 370px;" />
      </el-form-item>
      <el-form-item label="手机号码">
        <el-input v-model="form.phone" style="width: 370px;" />
      </el-form-item>
      <el-form-item label="用户积分">
        <el-input v-model="form.integral" style="width: 370px;" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="text" @click="cancel">取消</el-button>
      <el-button :loading="loading" type="primary" @click="doSubmit">确认</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { add, edit } from '@/api/wechat/yxUser'
export default {
  props: {
    isAdd: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      loading: false, dialog: false,
      form: {
        id: '',
        account: '',
        pwd: '',
        real_name: '',
        birthday: '',
        cardId: '',
        mark: '',
        nickname: '',
        avatar: '',
        phone: '',
        add_time: '',
        addIp: '',
        lastTime: '',
        lastIp: '',
        nowMoney: '',
        brokeragePrice: '',
        integral: '',
        signNum: '',
        status: '',
        level: '',
        spreadUid: '',
        spreadTime: '',
        userType: '',
        payCount: '',
        spreadCount: '',
        loginType: ''
      },
      rules: {
      }
    }
  },
  methods: {
    cancel() {
      this.resetForm()
    },
    doSubmit() {
      this.loading = true
      if (this.isAdd) {
        this.doAdd()
      } else this.doEdit()
    },
    doAdd() {
      add(this.form).then(res => {
        this.resetForm()
        this.$notify({
          title: '添加成功',
          type: 'success',
          duration: 2500
        })
        this.loading = false
        this.$parent.init()
      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })
    },
    doEdit() {
      edit(this.form).then(res => {
        this.resetForm()
        this.$notify({
          title: '修改成功',
          type: 'success',
          duration: 2500
        })
        this.loading = false
        this.$parent.init()
      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })
    },
    resetForm() {
      this.dialog = false
      this.$refs['form'].resetFields()
      this.form = {
        id: '',
        account: '',
        pwd: '',
        real_name: '',
        birthday: '',
        cardId: '',
        mark: '',
        nickname: '',
        avatar: '',
        phone: '',
        add_time: '',
        addIp: '',
        lastTime: '',
        lastIp: '',
        nowMoney: '',
        brokeragePrice: '',
        integral: '',
        signNum: '',
        status: '',
        level: '',
        spreadUid: '',
        spreadTime: '',
        userType: '',
        payCount: '',
        spreadCount: '',
        loginType: ''
      }
    }
  }
}
</script>

<style scoped>

</style>
