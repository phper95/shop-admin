<template>
  <div class="app-container">
    <div>
      <el-steps v-if="order.refund_status===0" :active="order_status.size" align-center process-status="process" finish-status="success">
        <el-step title="用户下单" :description="order_status.cache_key_create_order"></el-step>
        <el-step title="待发货" :description="order_status.pay_success"></el-step>
        <el-step title="待收货" :description="order_status.delivery_goods"></el-step>
        <el-step title="待评价" :description="order_status.user_take_delivery"></el-step>
        <el-step title="已完成" :description="order_status.check_order_over"></el-step>
      </el-steps>
      <el-steps v-else :active="order.refund_status+1" align-center process-status="process" finish-status="success">
        <el-step title="用户下单" :description="orderStatus.cache_key_create_order"></el-step>
        <el-step title="用户申请退款" :description="orderStatus.apply_refund"></el-step>
        <el-step title="退款申请通过" :description="orderStatus.refund_order_success"></el-step>
      </el-steps>

    </div>
    <el-card shadow="never" style="margin-top: 15px">
      <div class="operate-container">
        <i class="el-icon-warning color-danger" style="margin-left: 20px"></i>
        <span class="color-danger">当前订单状态：<span v-html="order.status_name"></span></span>
        <div class="operate-button-container" v-show="order._status===1">
          <el-button size="mini" @click="editOrder(order)">修改订单</el-button>
          <el-button size="mini" @click="remark(order)">备注订单</el-button>
        </div>
        <div class="operate-button-container" v-show="order._status===3">
          <el-button size="mini" @click="refund(order)">立即退款</el-button>
          <el-button size="mini" @click="remark(order)">备注订单</el-button>
        </div>
        <div class="operate-button-container" v-show="order._status===2">
          <el-button size="mini" @click="edit(order)">去发货</el-button>
          <el-button size="mini" @click="remark(order)">备注订单</el-button>
        </div>
        <div class="operate-button-container" v-show="order._status===4">
          <el-button size="mini" @click="showLogisticsDialog">订单跟踪</el-button>
          <el-button size="mini" @click="remark(order)">备注订单</el-button>
        </div>
        <div class="operate-button-container" v-show="order._status===6||order._status===7 ">
          <el-button size="mini" @click="showLogisticsDialog">订单跟踪</el-button>
          <el-button size="mini" @click="remark(order)">备注订单</el-button>
        </div>
      </div>
      <div style="margin-top: 20px">
        <svg-icon icon-class="marker" style="color: #606266"></svg-icon>
        <span class="font-small">基本信息</span>
      </div>
      <div class="table-layout">
        <el-row>
          <el-col :span="4" class="table-cell-title">订单编号</el-col>
          <el-col :span="4" class="table-cell-title">发货单流水号</el-col>
          <el-col :span="4" class="table-cell-title">用户账号</el-col>
          <el-col :span="4" class="table-cell-title">支付方式</el-col>
          <el-col :span="4" class="table-cell-title">订单来源</el-col>
          <el-col :span="4" class="table-cell-title">订单类型</el-col>
        </el-row>
        <el-row>
          <el-col :span="4" class="table-cell">{{order.order_id}}</el-col>
          <el-col :span="4" class="table-cell">暂无</el-col>
          <el-col :span="4" class="table-cell">{{user_dto.nickname}}</el-col>
          <el-col :span="4" class="table-cell">{{order.pay_type_name }}</el-col>
          <el-col :span="4" class="table-cell">{{order.is_channel | formatSourceType}}</el-col>
          <el-col :span="4" class="table-cell">{{order.pink_name }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="4" class="table-cell-title">配送方式</el-col>
          <el-col :span="4" class="table-cell-title">物流单号</el-col>
          <el-col :span="4" class="table-cell-title">自动确认收货时间</el-col>
          <el-col :span="4" class="table-cell-title">订单可得积分</el-col>
          <el-col :span="4" class="table-cell-title">消费积分。</el-col>
          <el-col :span="4" class="table-cell-title">活动信息</el-col>
        </el-row>
        <el-row>
          <el-col :span="4" class="table-cell">{{order.shipping_type | formatshipping_type}}</el-col>
          <el-col :span="4" class="table-cell">{{order.delivery_sn | formatNull}}</el-col>
          <el-col :span="4" class="table-cell">7天</el-col>
          <el-col :span="4" class="table-cell">{{order.gain_integral}}</el-col>
          <el-col :span="4" class="table-cell">{{order.pay_integral}}</el-col>
          <el-col :span="4" class="table-cell">
            <el-popover
              placement="top-start"
              title="活动信息"
              width="200"
              trigger="hover"
              :content="order.promotion_info">
              <span slot="reference">{{order.promotion_info | formatLongText}}</span>
            </el-popover>
          </el-col>
        </el-row>
      </div>
      <div style="margin-top: 20px">
        <svg-icon icon-class="marker" style="color: #606266"></svg-icon>
        <span class="font-small">收货人信息</span>
      </div>
      <div class="table-layout">
        <el-row>
          <el-col :span="4" class="table-cell-title">用户昵称</el-col>
          <el-col :span="4" class="table-cell-title">收货人</el-col>
          <el-col :span="4" class="table-cell-title">手机号码</el-col>
          <el-col :span="4" class="table-cell-title">收货地址</el-col>
          <el-col :span="4" class="table-cell-title">用户备注</el-col>
          <el-col :span="4" class="table-cell-title">管理员备注</el-col>
        </el-row>
        <el-row>
          <el-col :span="4" class="table-cell">{{ user_dto.nickname}}</el-col>
          <el-col :span="4" class="table-cell">{{order.real_name}}</el-col>
          <el-col :span="4" class="table-cell">{{order.user_phone}}</el-col>
          <el-col :span="4" class="table-cell">
            <el-popover
              placement="top-start"
              title="收货地址"
              width="300"
              trigger="hover"
              :content="order.user_address">
              <span slot="reference">{{order.user_address | formatLongText}}</span>
            </el-popover>
          </el-col>
          <el-col :span="4" class="table-cell">
            <el-popover
              placement="top-start"
              title="用户备注"
              width="300"
              trigger="hover"
              :content="order.mark">
              <span slot="reference">{{order.mark | formatLongText}}</span>
            </el-popover>
          </el-col>
          <el-col :span="4" class="table-cell">
            <el-popover
              placement="top-start"
              title="管理员备注"
              width="200"
              trigger="hover"
              :content="order.remark">
              <span slot="reference">{{order.remark | formatLongText}}</span>
            </el-popover>
          </el-col>
        </el-row>
      </div>
      <div style="margin-top: 20px">
        <svg-icon icon-class="marker" style="color: #606266"></svg-icon>
        <span class="font-small">商品信息</span>
      </div>
      <el-table
        :data="order.cartInfoList"
        size="small"
        style="width: 100%;margin-top: 20px" >
        <el-table-column label="商品图片" width="150" align="center">
          <template slot-scope="scope">
            <img :src="scope.row.cartInfoMap.productInfo.attrInfo.image" style="height: 80px">
          </template>
        </el-table-column>
        <el-table-column label="商品名称" width="300" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.cartInfoMap.productInfo.storeName}}</p>
          </template>
        </el-table-column>
        <el-table-column label="价格/货号" width="240" align="center">
          <template slot-scope="scope">
            <p>价格：￥{{scope.row.cartInfoMap.productInfo.attrInfo.price}}</p>
            <p>货号：{{scope.row.cartInfoMap.productInfo.attrInfo.bar_code}}</p>
          </template>
        </el-table-column>
        <el-table-column label="属性" width="240" align="center">
          <template slot-scope="scope">
            {{scope.row.cartInfoMap.productInfo.attrInfo.sku}}
          </template>
        </el-table-column>
        <el-table-column label="数量" width="180" align="center">
          <template slot-scope="scope">
            {{scope.row.cartInfoMap.cartNum}}
          </template>
        </el-table-column>
        <el-table-column label="小计"  align="center">
          <template slot-scope="scope">
            ￥{{scope.row.cartInfoMap.truePrice}}
          </template>
        </el-table-column>
      </el-table>
      <div style="float: right;margin: 20px">
        合计：<span class="color-danger">￥{{order.pay_price}}</span>
      </div>
      <div style="margin-top: 60px">
        <svg-icon icon-class="marker" style="color: #606266"></svg-icon>
        <span class="font-small">费用信息</span>
      </div>
      <div class="table-layout">
        <el-row>
          <el-col :span="6" class="table-cell-title">商品合计</el-col>
          <el-col :span="6" class="table-cell-title">运费</el-col>
          <el-col :span="6" class="table-cell-title">优惠券</el-col>
          <el-col :span="6" class="table-cell-title">积分抵扣</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell">￥{{order.total_price}}</el-col>
          <el-col :span="6" class="table-cell">￥{{order.pay_postage}}</el-col>
          <el-col :span="6" class="table-cell">-￥{{order.coupon_price}}</el-col>
          <el-col :span="6" class="table-cell">-￥{{order.deduction_price}}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell-title">活动优惠</el-col>
          <el-col :span="6" class="table-cell-title">折扣金额</el-col>
          <el-col :span="6" class="table-cell-title">订单总金额</el-col>
          <el-col :span="6" class="table-cell-title">应付款金额</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell">-￥0</el-col>
          <el-col :span="6" class="table-cell">-￥0</el-col>
          <el-col :span="6" class="table-cell">
            <span class="color-danger">￥{{order.total_price}}</span>
          </el-col>
          <el-col :span="6" class="table-cell">
            <span class="color-danger">￥{{order.pay_price}}</span>
          </el-col>
        </el-row>
      </div>
      <div style="margin-top: 20px">
        <svg-icon icon-class="marker" style="color: #606266"></svg-icon>
        <span class="font-small">操作信息</span>
      </div>
      <el-table style="margin-top: 20px;width: 100%"
                ref="orderHistoryTable"
                :data="order.storeOrderStatusList" border>
<!--        <el-table-column label="操作者"  width="120" align="center">-->
<!--          <template slot-scope="scope">-->
<!--           &lt;!&ndash; {{scope.row.operateMan}}&ndash;&gt;-->
<!--          </template>-->
<!--        </el-table-column>-->
        <el-table-column label="操作时间"  width="160" align="center">
          <template slot-scope="scope">
            {{scope.row.changeTime}}
          </template>
        </el-table-column>
<!--        <el-table-column label="订单状态"  width="120" align="center">-->
<!--          <template slot-scope="scope">-->
<!--            {{scope.row.changeType | formatStatus}}-->
<!--          </template>-->
<!--        </el-table-column>-->
<!--        <el-table-column label="付款状态"  width="120" align="center">-->
<!--          <template slot-scope="scope">-->
<!--            {{scope.row.changeType | formatPayStatus}}-->
<!--          </template>-->
<!--        </el-table-column>-->
<!--        <el-table-column label="发货状态"  width="120" align="center">-->
<!--          <template slot-scope="scope">-->
<!--            {{scope.row.changeType | formatDeliverStatus}}-->
<!--          </template>-->
<!--        </el-table-column>-->
        <el-table-column label="备注" align="center">
          <template slot-scope="scope">
            {{scope.row.changeMessage}}
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="订单跟踪"
               :visible.sync="kuaidiDialogVisible"
               width="40%">
      <el-steps direction="vertical"
                :active="90"
                finish-status="success"
                space="50px">
        <el-step  v-for="item in logisticsList"
                  :key="item.acceptStation"
                  :title="item.acceptStation"
                  :description="item.acceptTime"></el-step>
      </el-steps>
    </el-dialog>
   <!-- <el-dialog title="修改收货人信息"
              :visible.sync="receiverDialogVisible"
              width="40%">
     <el-form :model="receiverInfo"
              ref="receiverInfoForm"
              label-width="150px">
       <el-form-item label="收货人姓名：">
         <el-input v-model="receiverInfo.receiverName" style="width: 200px"></el-input>
       </el-form-item>
       <el-form-item label="手机号码：">
         <el-input v-model="receiverInfo.receiverPhone" style="width: 200px">
         </el-input>
       </el-form-item>
       <el-form-item label="邮政编码：">
         <el-input v-model="receiverInfo.receiverPostCode" style="width: 200px">
         </el-input>
       </el-form-item>
       <el-form-item label="所在区域：">
         <v-distpicker :province="receiverInfo.receiverProvince"
                       :city="receiverInfo.receiverCity"
                       :area="receiverInfo.receiverRegion"
                       @selected="onSelectRegion"></v-distpicker>
       </el-form-item>
       <el-form-item label="详细地址：">
         <el-input v-model="receiverInfo.receiverDetailAddress" type="textarea" rows="3">
         </el-input>
       </el-form-item>
     </el-form>
     <span slot="footer" class="dialog-footer">
     <el-button @click="receiverDialogVisible = false">取 消</el-button>
     <el-button type="primary" @click="handleUpdateReceiverInfo">确 定</el-button>
     </span>
   </el-dialog> -->
    <!-- <el-dialog title="修改费用信息"
               :visible.sync="moneyDialogVisible"
               width="80%">
      <div class="table-layout">
        <el-row>
          <el-col :span="6" class="table-cell-title">商品合计</el-col>
          <el-col :span="6" class="table-cell-title">运费</el-col>
          <el-col :span="6" class="table-cell-title">优惠券</el-col>
          <el-col :span="6" class="table-cell-title">积分抵扣</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell">￥{{order.total_price}}</el-col>
          <el-col :span="6" class="table-cell">
            <el-input type="number" v-model="order.pay_postage" size="mini"><template slot="prepend">￥</template></el-input>
          </el-col>
          <el-col :span="6" class="table-cell">-￥{{order.coupon_price}}</el-col>
          <el-col :span="6" class="table-cell">-￥{{order.use_integral}}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell-title">活动优惠</el-col>
          <el-col :span="6" class="table-cell-title">折扣金额</el-col>
          <el-col :span="6" class="table-cell-title">订单总金额</el-col>
          <el-col :span="6" class="table-cell-title">应付款金额</el-col>
        </el-row>
        <el-row>
          <el-col :span="6" class="table-cell">-￥{{order.promotionAmount}}</el-col>
          <el-col :span="6" class="table-cell">
            <el-input type="number" v-model="order.deduction_price" size="mini"><template slot="prepend">-￥</template></el-input>
          </el-col>
          <el-col :span="6" class="table-cell">
            <span class="color-danger">￥{{order.total_price}}</span>
          </el-col>
          <el-col :span="6" class="table-cell">
            <span class="color-danger">￥{{order.pay_price}}</span>
          </el-col>
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
      <el-button @click="moneyDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="handleUpdateMoneyInfo">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="关闭订单"
               :visible.sync="closeDialogVisible"
               width="40%">
      <el-form :model="closeInfo"
               label-width="150px">
        <el-form-item label="操作备注：">
          <el-input v-model="closeInfo.note" type="textarea" rows="3">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleCloseOrder">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="备注订单"
               :visible.sync="markOrderDialogVisible"
               width="40%">
      <el-form :model="markInfo"
               label-width="150px">
        <el-form-item label="操作备注：">
          <el-input v-model="markInfo.note" type="textarea" rows="3">
          </el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="markOrderDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleMarkOrder">确 定</el-button>
      </span>
    </el-dialog> -->

      <eForm ref="form" :is-add="isAdd" />
      <eRefund ref="form2" :is-add="isAdd" />
      <editOrder ref="form3" :is-add="isAdd" />
      <eRemark ref="form4" :is-add="isAdd" />
  </div>
</template>
<script>
import { express, getOrderDetail,
  getNowOrderStatus} from '@/api/shop/yxStoreOrder'
import {formatTimeTwo} from '@/utils/index';
import eForm from './form'
import eRefund from './refund'
import editOrder from './edit'
import eRemark from './remark'
  const defaultReceiverInfo = {
    order_id:null,
    receiverName:null,
    receiverPhone:null,
    receiverPostCode:null,
    receiverDetailAddress:null,
    receiverProvince:null,
    receiverCity:null,
    receiverRegion:null,
    status:null
  };
  export default {
    components: {eForm, eRefund, editOrder, eRemark},
    data() {
      return {
        orderStatus:null,
        isAdd: false,
        id: null,
        order: {

        },
        user:{

        },
        logisticsList:[],
        receiverDialogVisible:false,
        receiverInfo:Object.assign({},defaultReceiverInfo),
        moneyDialogVisible:false,
        moneyInfo:{order_id:null, freightAmount:0, discountAmount:0,status:null},
        messageDialogVisible:false,
        message: {title:null, content:null},
        closeDialogVisible:false,
        kuaidiDialogVisible:false,
        closeInfo:{note:null,id:null},
        markOrderDialogVisible:false,
        markInfo:{note:null},
        user_dto: {},
        logisticsDialogVisible: {
          visible: false,
          list: []
        }
      }
    },
    filters: {
      formatNull(value) {
        if(value===undefined||value===null||value===''){
          return '暂无';
        }else{
          return value;
        }
      },
      formatLongText(value) {
        if(value===undefined||value===null||value===''){
          return '暂无';
        }else if(value.length>8){
          return value.substr(0, 8) + '...';
        }else{
          return value;
        }
      },
      formatSourceType(value) {
        if (value === 1) {
          return '小程序';
        } else {
          return '公众号/H5';
        }
      },
      formatshipping_type(value){
        if (value === 1) {
          return '快递';
        } else {
          return '门店自提';
        }
      },
      formatAddress(order) {
        let str = order.receiverProvince;
        if (order.receiverCity != null) {
          str += "  " + order.receiverCity;
        }
        str += "  " + order.receiverRegion;
        str += "  " + order.receiverDetailAddress;
        return str;
      },
      formatStatus(value) {
        if (value === 1) {
          return '待发货';
        } else if (value === 2) {
          return '已发货';
        } else if (value === 3) {
          return '已完成';
        } else if (value === 4) {
          return '已关闭';
        } else if (value === 5) {
          return '无效订单';
        } else {
          return '待付款';
        }
      },
      formatPayStatus(value) {
        if (value === 0) {
          return '未支付';
        } else if(value===4){
          return '已退款';
        }else{
          return '已支付';
        }
      },
      formatDeliverStatus(value) {
        if (value === 0||value === 1) {
          return '未发货';
        } else {
          return '已发货';
        }
      },
      formatProductAttr(value){
        if(value==null){
          return '';
        }else{
          let attr = JSON.parse(value);
          let result='';
          for(let i=0;i<attr.length;i++){
            result+=attr[i].key;
            result+=":";
            result+=attr[i].value;
            result+=";";
          }
          return result;
        }
      }
    },
    mounted () {
      this.init();
    },
    methods: {
       refund(data) {
        this.isAdd = false
        const _this = this.$refs.form2
        _this.form = {
          id: data.id,
          order_id: data.order_id,
          uid: data.uid,
          real_name: data.real_name,
          user_phone: data.user_phone,
          user_address: data.user_address,
          cart_id: data.cart_id,
          freight_price: data.freight_price,
          total_num: data.total_num,
          total_price: data.total_price,
          total_postage: data.total_postage,
          pay_price: data.pay_price,
          pay_postage: data.pay_postage,
          deduction_price: data.deduction_price,
          coupon_id: data.coupon_id,
          coupon_price: data.coupon_price,
          paid: data.paid,
          pay_time: data.pay_time,
          pay_type: data.pay_type,
          add_time: data.add_time,
          status: data.status,
          refund_status: data.refund_status,
          refund_reason_wap_img: data.refund_reason_wap_img,
          refund_reason_wap_explain: data.refund_reason_wap_explain,
          refund_reason_time: data.refund_reason_time,
          refund_reason_wap: data.refund_reason_wap,
          refund_reason: data.refund_reason,
          refund_price: data.refund_price,
          delivery_name: data.delivery_name,
          delivery_type: data.delivery_type,
          delivery_id: data.delivery_id,
          gain_integral: data.gain_integral,
          use_integral: data.use_integral,
          back_integral: data.back_integral,
          mark: data.mark,
          is_del: data.is_del,
          unique: data.unique,
          remark: data.remark,
          mer_Id: data.mer_Id,
          is_mer_check: data.is_mer_check,
          combination_id: data.combination_id,
          pink_id: data.pink_id,
          cost: data.cost,
          seckill_id: data.seckill_id,
          bargain_id: data.bargain_id,
          verify_code: data.verify_code,
          store_id: data.store_id,
          shipping_type: data.shipping_type,
          is_channel: data.is_channel,
          is_remind: data.is_remind,
          pay_integral: data.pay_integral,
          is_system_del: data.is_system_del
        }
        _this.dialog = true
      },
       edit(data) {
        this.isAdd = false
        const _this = this.$refs.form
        _this.form = {
          id: data.id,
          order_id: data.order_id,
          uid: data.uid,
          real_name: data.real_name,
          user_phone: data.user_phone,
          user_address: data.user_address,
          cart_id: data.cart_id,
          freight_price: data.freight_price,
          total_num: data.total_num,
          total_price: data.total_price,
          total_postage: data.total_postage,
          pay_price: data.pay_price,
          pay_postage: data.pay_postage,
          deduction_price: data.deduction_price,
          coupon_id: data.coupon_id,
          coupon_price: data.coupon_price,
          paid: data.paid,
          pay_time: data.pay_time,
          pay_type: data.pay_type,
          add_time: data.add_time,
          status: data.status,
          refund_status: data.refund_status,
          refund_reason_wap_img: data.refund_reason_wap_img,
          refund_reason_wap_explain: data.refund_reason_wap_explain,
          refund_reason_time: data.refund_reason_time,
          refund_reason_wap: data.refund_reason_wap,
          refund_reason: data.refund_reason,
          refund_price: data.refund_price,
          delivery_name: data.delivery_name,
          deliverySn: data.deliverySn,
          delivery_type: data.delivery_type,
          delivery_id: data.delivery_id,
          gain_integral: data.gain_integral,
          use_integral: data.use_integral,
          back_integral: data.back_integral,
          mark: data.mark,
          is_del: data.is_del,
          unique: data.unique,
          remark: data.remark,
          mer_Id: data.mer_Id,
          is_mer_check: data.is_mer_check,
          combination_id: data.combination_id,
          pink_id: data.pink_id,
          cost: data.cost,
          seckill_id: data.seckill_id,
          bargain_id: data.bargain_id,
          verify_code: data.verify_code,
          store_id: data.store_id,
          shipping_type: data.shipping_type,
          is_channel: data.is_channel,
          is_remind: data.is_remind,
          pay_integral: data.pay_integral,
          is_system_del: data.is_system_del
        }
        _this.dialog = true
      },
       editOrder(data) {
        this.isAdd = false
        const _this = this.$refs.form3
        _this.form = {
          id: data.id,
          order_id: data.order_id,
          uid: data.uid,
          real_name: data.real_name,
          user_phone: data.user_phone,
          user_address: data.user_address,
          cart_id: data.cart_id,
          freight_price: data.freight_price,
          total_num: data.total_num,
          total_price: data.total_price,
          total_postage: data.total_postage,
          pay_price: data.pay_price,
          pay_postage: data.pay_postage,
          deduction_price: data.deduction_price,
          coupon_id: data.coupon_id,
          coupon_price: data.coupon_price,
          paid: data.paid,
          pay_time: data.pay_time,
          pay_type: data.pay_type,
          add_time: data.add_time,
          status: data.status,
          refund_status: data.refund_status,
          refund_reason_wap_img: data.refund_reason_wap_img,
          refund_reason_wap_explain: data.refund_reason_wap_explain,
          refund_reason_time: data.refund_reason_time,
          refund_reason_wap: data.refund_reason_wap,
          refund_reason: data.refund_reason,
          refund_price: data.refund_price,
          delivery_name: data.delivery_name,
          delivery_type: data.delivery_type,
          delivery_id: data.delivery_id,
          gain_integral: data.gain_integral,
          use_integral: data.use_integral,
          back_integral: data.back_integral,
          mark: data.mark,
          is_del: data.is_del,
          unique: data.unique,
          remark: data.remark,
          mer_Id: data.mer_Id,
          is_mer_check: data.is_mer_check,
          combination_id: data.combination_id,
          pink_id: data.pink_id,
          cost: data.cost,
          seckill_id: data.seckill_id,
          bargain_id: data.bargain_id,
          verify_code: data.verify_code,
          store_id: data.store_id,
          shipping_type: data.shipping_type,
          is_channel: data.is_channel,
          is_remind: data.is_remind,
          pay_integral: data.pay_integral,
          is_system_del: data.is_system_del
        }
        _this.dialog = true
      },
       remark(data) {
        this.isAdd = false
        const _this = this.$refs.form4
        _this.form = {
          id: data.id,
          order_id: data.order_id,
          uid: data.uid,
          real_name: data.real_name,
          user_phone: data.user_phone,
          user_address: data.user_address,
          cart_id: data.cart_id,
          freight_price: data.freight_price,
          total_num: data.total_num,
          total_price: data.total_price,
          total_postage: data.total_postage,
          pay_price: data.pay_price,
          pay_postage: data.pay_postage,
          deduction_price: data.deduction_price,
          coupon_id: data.coupon_id,
          coupon_price: data.coupon_price,
          paid: data.paid,
          pay_time: data.pay_time,
          pay_type: data.pay_type,
          add_time: data.add_time,
          status: data.status,
          refund_status: data.refund_status,
          refund_reason_wap_img: data.refund_reason_wap_img,
          refund_reason_wap_explain: data.refund_reason_wap_explain,
          refund_reason_time: data.refund_reason_time,
          refund_reason_wap: data.refund_reason_wap,
          refund_reason: data.refund_reason,
          refund_price: data.refund_price,
          delivery_name: data.delivery_name,
          delivery_type: data.delivery_type,
          delivery_id: data.delivery_id,
          gain_integral: data.gain_integral,
          use_integral: data.use_integral,
          back_integral: data.back_integral,
          mark: data.mark,
          is_del: data.is_del,
          unique: data.unique,
          remark: data.remark,
          mer_Id: data.mer_Id,
          is_mer_check: data.is_mer_check,
          combination_id: data.combination_id,
          pink_id: data.pink_id,
          cost: data.cost,
          seckill_id: data.seckill_id,
          bargain_id: data.bargain_id,
          verify_code: data.verify_code,
          store_id: data.store_id,
          shipping_type: data.shipping_type,
          is_channel: data.is_channel,
          is_remind: data.is_remind,
          pay_integral: data.pay_integral,
          is_system_del: data.is_system_del
        }
        _this.dialog = true
      },

      express() {
        let params ={
          "orderCode": this.order.id,
          "shipperCode": this.order.delivery_sn,
          "logisticCode": this.order.delivery_id
        }

        express(params).then(res=>{
          this.expressInfo = res.Traces
          this.kuaidiDialogVisible = true;
          this.logisticsList = this.expressInfo
        }).catch(err => {
        })

      },
      init(){
        let that = this;
        let id = that.$route.params.id || 0;
        this.getNowOrderStatus();
        getOrderDetail(id).then(response => {
          this.order = response;
          this.user_dto = this.order.user_dto;
        });
      },
      onSelectRegion(data){
        this.receiverInfo.receiverProvince = data.province.value;
        this.receiverInfo.receiverCity = data.city.value;
        this.receiverInfo.receiverRegion = data.area.value;
      },
      formatTime(time) {
        if (time == null || time === '') {
          return '';
        }
        let date = new Date(time);
        return formatTimeTwo(date, 'yyyy-MM-dd hh:mm:ss')
      },
      formatStepStatus(value) {
        //todo  1-未付款 2-未发货 3-退款中 4-待收货 5-待评价 6-已完成 7-已退款
        if (value === 1) {
          //待发货
          return 1;
        } else if (value === 2) {
          //已发货
          return 3;
        } else if (value === 3) {
          //已完成
          return 4;
        } else if (value === 4) {
          //已完成
          return 5;
        } else if (value === 5) {
          //已完成
          return 4;
        }else {
          //待付款、已关闭、无限订单
          return 1;
        }
      },
      showUpdateReceiverDialog(){
        this.receiverDialogVisible=true;
        this.receiverInfo={
          order_id:this.order.id,
          receiverName:this.order.receiverName,
          receiverPhone:this.order.receiverPhone,
          receiverPostCode:this.order.receiverPostCode,
          receiverDetailAddress:this.order.receiverDetailAddress,
          receiverProvince:this.order.receiverProvince,
          receiverCity:this.order.receiverCity,
          receiverRegion:this.order.receiverRegion,
          status:this.order._status
        }
      },
      handleUpdateReceiverInfo(){
        this.$confirm('是否要修改收货信息?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // updateReceiverInfo(this.receiverInfo).then(response=>{
          //   this.receiverDialogVisible=false;
          //   this.$message({
          //     type: 'success',
          //     message: '修改成功!'
          //   });
          //   getOrderDetail(this.id).then(response => {
          //     this.order = response.data;
          //   });
          // });
        });
      },
      showUpdateMoneyDialog(){
        this.moneyDialogVisible=true;
        this.moneyInfo.order_id=this.order.id;
        this.moneyInfo.freightAmount=this.order.freightAmount;
        this.moneyInfo.discountAmount=this.order.discountAmount;
        this.moneyInfo.status=this.order._status;
      },
      handleUpdateMoneyInfo(){
        this.$confirm('是否要修改费用信息?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.moneyDialogVisible=false;
          // updateMoneyInfo(this.moneyInfo).then(response=>{
          //   this.moneyDialogVisible=false;
          //   this.$message({
          //     type: 'success',
          //     message: '修改成功!'
          //   });
          //   getOrderDetail(this.id).then(response => {
          //     this.order = response.data;
          //   });
          // });
        });
      },
      showMessageDialog(){
        this.messageDialogVisible=true;
        this.message.title=null;
        this.message.content=null;
      },
      handleSendMessage(){
        this.$confirm('是否要发送站内信?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.messageDialogVisible=false;
          this.$message({
            type: 'success',
            message: '发送成功!'
          });
        });
      },
      showCloseOrderDialog(){
        this.closeDialogVisible=true;
        this.closeInfo.note=null;
        this.closeInfo.id=this.id;
      },

      // handleCloseOrder(){
      //   this.$confirm('是否要关闭?', '提示', {
      //     confirmButtonText: '确定',
      //     cancelButtonText: '取消',
      //     type: 'warning'
      //   }).then(() => {
      //     let params = new URLSearchParams();
      //     params.append("ids",[this.closeInfo.id]);
      //     params.append("note",this.closeInfo.note);
      //     closeOrder(params).then(response=>{
      //       this.closeDialogVisible=false;
      //       this.$message({
      //         type: 'success',
      //         message: '订单关闭成功!'
      //       });
      //       getOrderDetail(this.id).then(response => {
      //         this.order = response.data;
      //       });
      //     });
      //   });
      // },
      showMarkOrderDialog(){
        this.markOrderDialogVisible=true;
        this.markInfo.id=this.id;
        this.order.remark=null;
      },
      // 备注订单
      handleMarkOrder(){
        this.$confirm('是否要备注订单?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let params = new URLSearchParams();
          params.append("id",this.markInfo.id);
          params.append("note",this.markInfo.note);
          params.append("status",this.order._status);
          // updateOrderNote(params).then(response=>{
          //   this.markOrderDialogVisible=false;
          //   this.$message({
          //     type: 'success',
          //     message: '订单备注成功!'
          //   });
          //   getOrderDetail(this.id).then(response => {
          //     this.order = response.data;
          //   });
          // });
        });
      },
      handleDeleteOrder(){
        this.$confirm('是否要进行该删除操作?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let params = new URLSearchParams();
          params.append("ids",[this.id]);
          // deleteOrder(params).then(response=>{
          //   this.$message({
          //     message: '删除成功！',
          //     type: 'success',
          //     duration: 1000
          //   });
          //   this.$router.back();
          // });
        })
      },
      showLogisticsDialog(){
        this.express();

      },
      //获取当前订单状态
      getNowOrderStatus() {
        let id = this.$route.params.id || 0;

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
.detail-container {
  width: 80%;
  padding: 20px 20px 20px 20px;
  margin: 20px auto;
}

.operate-container {
  background: #F2F6FC;
  height: 80px;
  margin: -20px -20px 0;
  line-height: 80px;
}

.operate-button-container {
  float: right;
  margin-right: 20px
}

.table-layout {
  margin-top: 20px;
  border-left: 1px solid #DCDFE6;
  border-top: 1px solid #DCDFE6;
}

.table-cell {
  height: 60px;
  line-height: 40px;
  border-right: 1px solid #DCDFE6;
  border-bottom: 1px solid #DCDFE6;
  padding: 10px;
  font-size: 14px;
  color: #606266;
  text-align: center;
  overflow: hidden;
}

.table-cell-title {
  border-right: 1px solid #DCDFE6;
  border-bottom: 1px solid #DCDFE6;
  padding: 10px;
  background: #F2F6FC;
  text-align: center;
  font-size: 14px;
  color: #303133;
}
</style>
