<template>
  <el-dialog :append-to-body="true" :close-on-click-modal="false" :before-close="cancel" :visible.sync="dialog" :title="isAdd ? '新增' : '编辑'" width="500px">
    <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
      <el-form-item label="订单号">
        <el-input v-model="form.order_id" :disabled="true" style="width: 370px;" />
      </el-form-item>
      <el-form-item label="订单备注">
        <el-input v-model="form.remark" style="width: 370px;" rows="5" type="textarea" />
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="text" @click="cancel">取消</el-button>
      <el-button :loading="loading" type="primary" @click="doSubmit">确认</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { add, remark } from '@/api/shop/yxStoreOrder'
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
      },
      rules: {
        unique: [
          { required: true, message: 'please enter', trigger: 'blur' }
        ]
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
      remark(this.form).then(res => {
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
    }
  }
}
</script>

<style scoped>

</style>
