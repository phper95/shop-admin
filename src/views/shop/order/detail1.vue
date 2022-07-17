<template>
  <el-dialog :append-to-body="true" :close-on-click-modal="false" :before-close="cancel" :visible.sync="dialog" :title="isAdd ? '新增' : '订单详情'" width="700px">
    <el-card>
      <div slot="header">
        <span>收货信息</span>
      </div>
      <div class="text item">用户昵称:{{ form.nickname }}</div>
      <div class="text item">收货人: {{ form.real_name }}</div>
      <div class="text item">联系电话: {{ form.user_phone }}</div>
      <div class="text item">收货地址: {{ form.user_address }}</div>
    </el-card>
    <el-card>
      <div slot="header">
        <span>订单信息</span>
      </div>
      <el-row :gutter="24">
        <el-col :span="12">
          <div class="text item">订单编号: {{ form.order_id }}</div>
          <div class="text item">商品总数: {{ form.total_num }}</div>
          <div class="text item">支付邮费: {{ form.total_postage }}</div>
          <div class="text item">实际支付: {{ form.pay_price }}</div>
          <div class="text item">支付方式: {{ form.pay_type_name }}</div>
        </el-col>
        <el-col :span="12">
          <div class="text item">订单状态: <span v-html="form.statusName"></span></div>
          <div class="text item">商品总价: {{ form.total_price }}</div>
          <div class="text item">优惠券金额: {{ form.coupon_price }}</div>
          <div class="text item">创建时间: {{ parseTime(form.createTime) }}</div>
          <div class="text item">支付时间: {{ parseTime(form.pay_time) }}</div>
        </el-col>
      </el-row>
    </el-card>
    <el-card v-if="form.store_id == 0">
      <div slot="header">
        <span>物流信息</span>
      </div>
      <div class="text item">快递公司:{{ form.delivery_name }}</div>
      <div class="text item">快递单号:{{ form.delivery_id }}</div>

      <div><el-button :loading="loading" type="primary" @click="express">查看物流</el-button></div>
      <div style="margin-top: 20px">
      <el-timeline v-if="form.delivery_id && expressInfo.length > 0">
        <el-timeline-item
          v-for="(obj, index) in expressInfo"
          :key="index"
          :timestamp="obj.acceptTime"
          >
          {{obj.acceptStation}}
        </el-timeline-item>
      </el-timeline>
      <el-timeline :reverse="false" v-else>
        <el-timeline-item>
          暂无物流信息
        </el-timeline-item>
      </el-timeline>
      </div>
    </el-card>
    <el-card>
      <div slot="header">
        <span>备注信息</span>
      </div>
      <div class="text item">{{ form.remark }}</div>
    </el-card>
  </el-dialog>
</template>

<script>
import { add, edit, express,
  getNowOrderStatus } from '@/api/shop/yxStoreOrder'
import {formatTimeTwo, parseTime} from '@/utils/index'
export default {
  props: {
    isAdd: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      orderStatus:null,
      loading: false, dialog: false, expressInfo: [],
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
        deliverySn: '',
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
  watch: {
    'form': function(val) {
      this.getNowOrderStatus();
    }
  },
  methods: {
    parseTime,
    cancel() {
      this.dialog = false
    },
    express() {
      let params ={
        "orderCode": this.form.id,
        "shipperCode": this.form.deliverySn,
        "logisticCode": this.form.delivery_id
      }

      express(params).then(res=>{

        console.log(res)
        this.expressInfo = res.Traces

      }).catch(err => {
        this.loading = false
        console.log(err.response.data.message)
      })

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
    }, formatTime(time) {
      if (time == null || time === '') {
        return '';
      }
      let date = new Date(time);
      return formatTimeTwo(date, 'yyyy-MM-dd hh:mm:ss')
    },
    formatStepStatus(value) {
      //todo  1-未付款 2-未发货 3-退款中 4-待收货 5-待评价 6-已完成 7-已退款
      if (value === 2) {
        //待发货
        return 2;
      } else if (value === 4) {
        //已发货
        return 3;
      } else if (value === 6) {
        //已完成
        return 4;
      }else {
        //待付款、已关闭、无限订单
        return 1;
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
    getNowOrderStatus() {
      let id = this.form.id || 0;

      getNowOrderStatus(id)
        .then(res => {
          this.orderStatus = res;
        })
        .catch(err => {
          console.log(err.response.data.message);
        });
    },
  }
}
</script>

<style scoped>
  .text {
    font-size: 12px;
  }

  .item {
    padding: 6px 0;
  }

</style>
