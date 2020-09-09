<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input v-model="queryInfo.query" placeholder="请输入内容" clearable @clear="orderClick">
            <el-button slot="append" icon="el-icon-search" @click="orderClick" />
          </el-input>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" />
        <el-table-column label="订单编号" prop="order_number" />
        <el-table-column label="订单价格" prop="order_price" />
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '1'" type="success">已付款</el-tag>
            <el-tag v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" />
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template>
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showBox" />
            <el-button size="mini" type="success" icon="el-icon-location" @click="showProgressBox" />
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        layout="total, sizes, prev, pager, next, jumper"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>
    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
      @close="addressVisibleClosed"
    >
      <!-- 内容主体区域 -->
      <el-form ref="addressFormRef" :model="addressForm" :rules="addressFormRules" label-width="100px">
        <el-form-item label="省市区/县" prop="address1">
          <!-- 商品分类级联区域 -->
          <el-cascader
            v-model="addressForm.address1"
            :options="citydata"
            clearable
          />
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input/>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressVisible = false">确 定</el-button>
  </span>
    </el-dialog>
    <el-dialog
      title="修改地址"
      :visible.sync="progressVisible"
      width="50%">
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
import citydata from './citydata'
export default {
  name: 'Order',
  data() {
    return {
    // 获取订单列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 订单列表
      orderList: [],
      // 订单总数
      total: 0,
      // 控制修改地址对话框显示与隐藏
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [{ required: true, message: '请选择省市区/县', trigger: 'blur' }],
        address2: [{ required: true, message: '请输入详细地址', trigger: 'blur' }]
      },
      citydata,
      // 控制物流进度对话框显示与隐藏
      progressVisible: false,
      progressInfo: {}
    }
  },
  created() {
    this.getOrdersList()
  },
  methods: {
    // 监听 pagesize改变的事件
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getOrdersList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getOrdersList()
    },
    orderClick() {
      this.queryInfo.pagenum = 1
      this.getOrdersList()
    },
    // 获取订单列表
    async getOrdersList() {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    // 显示修改地址对话框
    showBox() {
      this.addressVisible = true
    },
    addressVisibleClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      if (res.meta.status !== 200) return this.$message.error('获取物流进度失败')
      this.progressInfo = res.data
      this.progressVisible = true
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader{
  width: 100%;
}
</style>
