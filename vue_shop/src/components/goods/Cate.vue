<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <!-- 添加分类按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expanded-type="false" show-index index-text="#" border :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted===false" class="el-icon-success" style="color: lightgreen" />
          <i v-else class="el-icon-error" style="color: red" />
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0" size="mini">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level===1" size="mini" type="success">二级</el-tag>
          <el-tag v-else size="mini" type="warning">三级</el-tag>
        </template>
        <!-- 排序 -->
        <template slot="opt">
          <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
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
  </div>
</template>

<script>
export default {
  name: 'Cate',
  data() {
    return {
      // 商品分类的数据类别，默认为空
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 3
      },
      // 分类总数
      total: 0,
      // 为table指定列名
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        }, {
          label: '是否有效',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示，将当前列使用模板名称
          template: 'isok'
        }, {
          label: '排序',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示，将当前列使用模板名称
          template: 'order'
        }, {
          label: '操作',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示，将当前列使用模板名称
          template: 'opt',
          width: '200px'
        }
      ]
    }
  },
  created() {
    // 获取分类列表
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取分类列表失败')
      this.cateList = res.data.result
      this.total = res.data.total
    }
  },
  // 监听 pagesize改变的事件
  handleSizeChange(val) {
    this.queryInfo.pagesize = val
    this.getCateList()
  },
  // 监听 页码值 改变的事件
  handleCurrentChange(val) {
    this.queryInfo.pagenum = val
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.treeTable{
  margin-top: 15px;
}
</style>
