<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator='/'>
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span='8'>
          <el-input placeholder='请输入内容' v-model='queryInfo.query'>
            <el-button slot='append' icon='el-icon-search' @click='searchOrder'></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data='orderList' border stripe>
        <el-table-column type='index'></el-table-column>
        <el-table-column label='订单编号' prop='order_number'></el-table-column>
        <el-table-column label='订单价格' prop='order_price'></el-table-column>
        <el-table-column label='是否付款'>
          <template slot-scope='scope'>
            <el-tag type='success' v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type='danger'>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label='是否发货' prop='is_send'></el-table-column>
        <el-table-column label='下单时间'>
          <template slot-scope='scope'>
            {{scope.row.update_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label='操作'>
            <el-button type='primary' icon='el-icon-edit' size='mini' @click='showDialog'></el-button>
            <el-button type='success' icon='el-icon-location' size='mini' @click='showProgressBox'></el-button>
        </el-table-column>
      </el-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change='handleSizeChange'
        @current-change='handleCurrentChange'
        :current-page='queryInfo.pagenum'
        :page-sizes='[5, 10, 15, 20]'
        :page-size='queryInfo.pagesize'
        layout='total, sizes, prev, pager, next, jumper'
        :total='total'>
      </el-pagination>
    </el-card>
    <!-- 修改对话框 -->
    <el-dialog
      title='修改地址'
      :visible.sync='addressDialogVisible'
      width='50%'
      @close='addressDialogClosed'>
      <el-form :model='addressForm' :rules='addressFormRules' ref='addressFormRef' label-width='100px'>
        <el-form-item label='省市区/县' prop='address1'>
          <el-cascader :options="cityData" v-model="addressForm.address1" :props="{ expandTrigger: 'hover' }"></el-cascader>
        </el-form-item>
        <el-form-item label='详细地址' prop='address2'>
          <el-input v-model='addressForm.address2'></el-input>
        </el-form-item>
      </el-form>
      <span slot='footer'>
        <el-button @click='addressDialogVisible = false'>取 消</el-button>
        <el-button type='primary' @click='addressDialogVisible = false'>确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流进度对话框 -->
    <el-dialog
      title='物流进度'
      :visible.sync='progressDialogVisible'
      width='50%'>
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
// 导入城市数据
import cityData from './citydata'

export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderList: [],
      addressDialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressDialogVisible: false,
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$.message.error('获取订单列表失败！')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 查询订单
    searchOrder() {
      this.$message.info('订单搜索功能正在维护中！')
    },
    // 展示修改对话框
    showDialog() {
      this.addressDialogVisible = true
    },
    // 监听对话框关闭事件
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
      this.addressForm.address1 = []
    },
    showProgressBox() {
      // 接口有误，用假数据代替
      const res = {
        data: [
          {
            time: '2018-05-10 09:39:00',
            ftime: '2018-05-10 09:39:00',
            context: '已签收,感谢使用顺丰,期待再次为您服务',
            location: ''
          },
          {
            time: '2018-05-10 08:23:00',
            ftime: '2018-05-10 08:23:00',
            context: '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
            location: ''
          },
          {
            time: '2018-05-10 07:32:00',
            ftime: '2018-05-10 07:32:00',
            context: '快件到达 [北京海淀育新小区营业点]',
            location: ''
          },
          {
            time: '2018-05-10 02:03:00',
            ftime: '2018-05-10 02:03:00',
            context: '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
            location: ''
          },
          {
            time: '2018-05-09 23:05:00',
            ftime: '2018-05-09 23:05:00',
            context: '快件到达 [北京顺义集散中心]',
            location: ''
          },
          {
            time: '2018-05-09 21:21:00',
            ftime: '2018-05-09 21:21:00',
            context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
            location: ''
          },
          {
            time: '2018-05-09 13:07:00',
            ftime: '2018-05-09 13:07:00',
            context: '顺丰速运 已收取快件',
            location: ''
          },
          {
            time: '2018-05-09 12:25:03',
            ftime: '2018-05-09 12:25:03',
            context: '卖家发货',
            location: ''
          },
          {
            time: '2018-05-09 12:22:24',
            ftime: '2018-05-09 12:22:24',
            context: '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
            location: ''
          },
          {
            time: '2018-05-08 21:36:04',
            ftime: '2018-05-08 21:36:04',
            context: '商品已经下单',
            location: ''
          }
        ],
        meta: { status: 200, message: '获取物流信息成功！' }
      }
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流进度失败！')
      }
      this.$message.success('获取物流进度成功！')
      this.progressInfo = res.data
      this.progressDialogVisible = true
    }
  }
}
</script>

<style lang='less' scoped>
.el-cascader{
  width: 100%;
}
</style>
