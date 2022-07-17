<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <!-- 搜索 -->
      <el-input v-model="query.value" clearable placeholder="输入搜索内容" style="width: 200px;" class="filter-item" @keyup.enter.native="toQuery" />
      <el-select v-model="cateId"  clearable placeholder="商品分类" class="filter-item" filterable :filter-method="dataFilter" style="width: 130px">
        <el-option v-for="item in optionsMetaShow" :disabled="item.disabled === 0"
                   :value="item.value"
                   :key="item.id"
                   :label="item.label"></el-option>
      </el-select>
      <el-button class="filter-item" size="mini" type="success" icon="el-icon-search" @click="toQuery">搜索</el-button>
      <!-- 新增 -->
      <div style="display: inline-block;margin: 0px 2px;">
        <el-button
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-plus"
          @click="toAddURL"
        >
            新增
        </el-button>
        <el-button
          type="danger"
          class="filter-item"
          size="mini"
          icon="el-icon-refresh"
          @click="toQuery"
        >刷新</el-button>
      </div>
    </div>
    <!--表单组件-->
    <!--表格渲染-->
    <el-table v-loading="loading" :data="data" size="small" style="width: 100%;">
      <el-table-column prop="id" label="商品id" />
      <el-table-column ref="table" prop="image" label="商品图片">
        <template slot-scope="scope">
          <a :href="scope.row.image" style="color: #42b983" target="_blank"><img :src="scope.row.image" alt="点击打开" class="el-avatar"></a>
        </template>
      </el-table-column>
      <el-table-column prop="store_name" label="商品名称" />
      <el-table-column prop="product_cate.cate_name" label="分类名称" />
      <el-table-column prop="price" label="商品价格" />
      <el-table-column prop="sales" label="销量" />
      <el-table-column prop="stock" label="库存" />
      <el-table-column label="状态" align="center">
        <template slot-scope="scope">
          <div @click="onSale(scope.row.id,scope.row.is_show)">
            <el-tag v-if="scope.row.is_show === 1" style="cursor: pointer" :type="''">已上架</el-tag>
            <el-tag v-else style="cursor: pointer" :type=" 'info' ">已下架</el-tag>
          </div>
        </template>
      </el-table-column>
        <el-table-column  prop="createTime" label="创建时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.create_time) }}</span>
          </template>
        </el-table-column>
      <el-table-column label="操作" width="265px" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="toUpdateURL(scope.row.id)"
          >
            编辑
          </el-button>
          <el-popover
            :ref="scope.row.id"
            placement="top"
            width="180"
          >
            <p>确定删除本条数据吗？</p>
            <div style="text-align: right; margin: 0">
              <el-button size="mini" type="text" @click="$refs[scope.row.id].doClose()">取消</el-button>
              <el-button :loading="delLoading" type="primary" size="mini" @click="subDelete(scope.row.id)">确定</el-button>
            </div>
            <el-button slot="reference" type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          </el-popover>
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <el-pagination
      :total="total"
      :current-page="page + 1"
      style="margin-top: 8px;"
      layout="total, prev, pager, next, sizes"
      @size-change="sizeChange"
      @current-change="pageChange"
    />
  </div>
</template>

<script>
import checkPermission from '@/utils/permission'
import initData from '@/mixins/crud'
import { del, onsale } from '@/api/shop/StoreProduct'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import Treeselect from '@riophae/vue-treeselect'
export default {
  components: {  Treeselect },
  mixins: [initData],
  data() {
    return {
      dropDownValue: '',
      optionsMetaShow: [],
      delLoading: false,
      visible: false,
      queryTypeOptions: [
        { key: 'store_name', display_name: '商品名称' }
      ],
      isAttr: false,
      cateId: null,
    }
  },
  created() {
    this.$nextTick(() => {
      this.init().then(() =>{
        this.optionsMetaShow = this.cateList
      })
    })
  },
  methods: {
    toAddURL(){
      this.$router.push({ path: '/mall/goodsAdd' })
    },
    toUpdateURL(id){
      this.$router.push({ path: '/mall/goodsEdit/'+id })
    },
    dataFilter(val){
      this.value=val
      if(val){
        this.optionsMetaShow=this.cateList.filter((item=>{
          if (!!~item.label.indexOf(val) || !!~item.label.toUpperCase().indexOf(val.toUpperCase())) {
            return true
          }
        }))
      }else{
        this.optionsMetaShow=this.cateList
      }
    },
    checkPermission,
    beforeInit() {
      this.url = 'shop/product'
      const sort = 'id,desc'
      this.params = { page: this.page, size: this.size, sort: sort, is_show: 1, is_del: 0,cateId: this.cateId  }
      const query = this.query
      const value = query.value
      if (value) { this.params['blurry'] = value }
      return true
    },
    subDelete(id) {
      this.delLoading = true
      del(id).then(res => {
        this.delLoading = false
        this.$refs[id].doClose()
        this.dleChangePage()
        this.init()
        this.$notify({
          title: '删除成功',
          type: 'success',
          duration: 2500
        })
      }).catch(err => {
        this.delLoading = false
        this.$refs[id].doClose()
        console.log(err.response.data.message)
      })
    },
    onSale(id, status) {
      this.$confirm(`确定进行[${status ? '下架' : '上架'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          if (status===0){
            status=1
          }else {
            status = 0
          }
          onsale(id, { status: status }).then(({ data }) => {
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
                this.init()
              }
            })
          })
        })
        .catch(() => { })
    },
    add() {
      this.isAdd = true
      this.$refs.form.dialog = true
      this.$refs.form.getCates()
    },
    edit(data) {
      this.isAdd = false
      const _this = this.$refs.form
      _this.getCates()
      _this.form = {
        id: data.id,
        mer_id: data.mer_id,
        image: data.image,
        slider_image: data.slider_image,
        image_arr: data.image.split(','),
        slider_image_arr: data.slider_image.split(','),
        store_name: data.store_name,
        store_info: data.store_info,
        keyword: data.keyword,
        bar_code: data.bar_code,
        store_category: data.store_category || {id:null},
        price: data.price,
        vip_price: data.vip_price,
        ot_price: data.ot_price,
        postage: data.postage,
        unit_name: data.unit_name,
        sort: data.sort,
        sales: data.sales,
        stock: data.stock,
        is_show: data.is_show,
        is_hot: data.is_hot,
        is_benefit: data.is_benefit,
        is_best: data.is_best,
        is_new: data.is_new,
        description: data.description,
        add_time: data.add_time,
        is_postage: data.is_postage,
        is_del: data.is_del,
        mer_use: data.mer_use,
        give_integral: data.give_integral,
        cost: data.cost,
        is_seckill: data.is_seckill,
        is_bargain: data.is_bargain,
        is_good: data.is_good,
        ficti: data.ficti,
        browse: data.browse,
        code_path: data.code_path,
        soure_link: data.soure_link
      }
      _this.dialog = true
    },
    attr(data) {
      console.log(3333)
      this.isAttr = false
      const _this = this.$refs.form2
      _this.form = {
        id: data.id,
        mer_id: data.mer_id,
        image: data.image,
        sliderImage: data.sliderImage,
        store_name: data.store_name,
        store_info: data.store_info,
        keyword: data.keyword,
        bar_code: data.bar_code,
        store_category: data.store_category,
        price: data.price,
        vip_price: data.vip_price,
        ot_price: data.ot_price,
        postage: data.postage,
        unit_name: data.unit_name,
        sort: data.sort,
        sales: data.sales,
        stock: data.stock,
        is_show: data.is_show,
        is_hot: data.is_hot,
        is_benefit: data.is_benefit,
        is_best: data.is_best,
        is_new: data.is_new,
        description: data.description,
        add_time: data.add_time,
        is_postage: data.is_postage,
        is_del: data.is_del,
        mer_use: data.mer_use,
        give_integral: data.give_integral,
        cost: data.cost,
        is_seckill: data.is_seckill,
        is_bargain: data.is_bargain,
        is_good: data.is_good,
        ficti: data.ficti,
        browse: data.browse,
        code_path: data.code_path,
        soure_link: data.soure_link
      }
      _this.dialog = true
      this.$refs.form2.getAttrs(data.id)
    }
  }
}
</script>

<style scoped>

</style>
