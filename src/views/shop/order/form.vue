<template>
  <el-dialog :append-to-body="true" :close-on-click-modal="false" :before-close="cancel" :visible.sync="dialog" :title="isAdd ? '新增' : '去发货'" width="500px">
    <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
      <el-form-item label="快递公司">
        <!--<el-input v-model="form.delivery_name" style="width: 370px;"/>-->
        <el-select v-model="form.delivery_name" filterable placeholder="请选择" style="width: 370px;">
          <el-option
            v-for="item in express"
            :key="item.id"
            :label="item.name"
            :value="item.name"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="快递单号">
        <el-input v-model="form.delivery_id" style="width: 370px;" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="text" @click="cancel">取消</el-button>
      <el-button :loading="loading" type="primary" @click="doSubmit">确认</el-button>
    </div>
  </el-dialog>
</template>

<script>

import { add, edit, get,updateDelivery } from '@/api/shop/yxStoreOrder'
export default {
  props: {
    isAdd: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      loading: false, dialog: false, express: [],
      form: {
        id: '',
        delivery_name: '',
        delivery_type: 'express',
        delivery_id: ''
      },
      rules: {
        unique: [
          { required: true, message: 'please enter', trigger: 'blur' }
        ]
      }
    }
  },

  created() {
    this.get()
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
      if(this.form._status == 4){
        updateDelivery(this.form).then(res => {
          this.resetForm()
          this.$notify({
            title: '操作成功',
            type: 'success',
            duration: 2500
          })
          this.loading = false
          this.$parent.init()
        }).catch(err => {
          this.loading = false
          console.log(err.response.data.message)
        })
      }else{
        edit(this.form).then(res => {
          this.resetForm()
          this.$notify({
            title: '操作成功',
            type: 'success',
            duration: 2500
          })
          this.loading = false
          this.$parent.init()
        }).catch(err => {
          this.loading = false
          console.log(err.response.data.message)
        })
      }

    },
    resetForm() {
      this.dialog = false
      this.$refs['form'].resetFields()
      this.form = {
        id: '',
        order_id: '',
        uid: '',
        real_name: '',
        user_phone: '',
        user_address: '',
        cart_id: '',
        freight_price: '',
        total_num: '',
        total_price: '',
        total_postage: '',
        pay_price: '',
        pay_postage: '',
        deduction_price: '',
        coupon_id: '',
        coupon_price: '',
        paid: '',
        pay_time: '',
        pay_type: '',
        add_time: '',
        status: '',
        refund_status: '',
        refund_reason_wap_img: '',
        refund_reason_wap_explain: '',
        refund_reason_time: '',
        refund_reason_wap: '',
        refund_reason: '',
        refund_price: '',
        delivery_name: '',
        delivery_type: '',
        delivery_id: '',
        gain_integral: '',
        use_integral: '',
        back_integral: '',
        mark: '',
        is_del: '',
        unique: '',
        remark: '',
        mer_Id: '',
        is_mer_check: '',
        combination_id: '',
        pink_id: '',
        cost: '',
        seckill_id: '',
        bargain_id: '',
        verify_code: '',
        store_id: '',
        shipping_type: '',
        is_channel: '',
        is_remind: '',
        is_system_del: ''
      }
    },
    get() {
      get().then(res => {
        this.express = res.data.content
      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })
    }
  }
}
</script>

<style scoped>

</style>
