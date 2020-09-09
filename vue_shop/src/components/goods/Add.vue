<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 提示信息区域 -->
      <el-alert
        title="添加商品信息"
        type="info"
        show-icon
        center
        :closable="false"
      />
      <!-- 步骤条区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息" />
        <el-step title="商品参数" />
        <el-step title="商品属性" />
        <el-step title="商品图片" />
        <el-step title="商品内容" />
        <el-step title="已完成" />
      </el-steps>
      <!-- tab页签区域 -->
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="100px" label-position="top">
        <el-tabs
          v-model="activeIndex"
          tab-position="left"
          style="height: auto;"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name" />
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number" />
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number" />
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number" />
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="casCaDerProps"
                clearable
                @change="handleChanged"
              />
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox v-for="(cb,i) in item.attr_vals" :key="i" :label="cb" border />
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals" />
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action便是图片要上传到的后台API地址 -->
            <el-upload
              class="upload-demo"
              :headers="headerObj"
              :action="uploadURL"
              :on-success="handleSuccess"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :file-list="addForm.goods_fileList"
              list-type="picture"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor
            v-model="addForm.goods_introduce" />
            <!-- 添加商品按钮 -->
            <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="30%"
    >
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'Add',
  data() {
    return {
      // 步骤条下标
      activeIndex: '0',
      // 添加商品的表单对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        goods_fileList: [],
        pics: [],
        // 商品详情内容
        goods_introduce: '',
        // 商品的参数属性
        attrs: []
      },
      // 添加商品的表单校验规则对象
      addFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品数量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      // 商品分类数据
      cateList: [],
      // 指定级联选择器的配置对象
      casCaDerProps: {
        // 次级菜单的展开方式
        expandTrigger: 'hover',
        // 指定选项的值为选项对象的某个属性值
        value: 'cat_id',
        // 指定选项标签为选项对象的某个属性值
        label: 'cat_name',
        // 指定选项的子选项为选项对象的某个属性值
        children: 'children',
        // 是否严格的遵守父子节点不互相关联
        checkStrictly: true
      },
      // 动态参数数据
      manyTableData: [],
      // 静态数据
      onlyTableData: [],
      // 上传图片的URL地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览地址
      previewPath: '',
      // 控制图片预览对话框显示与隐藏
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败')
      this.cateList = res.data
    },
    // 级联选择器选中项变化，会触发这个函数
    handleChanged() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 离开商品基本信息进行校验
    beforeTabLeave(activeName, oldName) {
      if (oldName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    // 当前选中的三级分类的Id
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    },
    // 点击tab的时候，触发这个函数
    async tabClicked() {
      // 证明访问的是动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId()}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取动态参数失败')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId()}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取静态属性失败')
        this.onlyTableData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除图片操作
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(x => {
        x.pic === filePath
      })
      this.addForm.pics.splice(index, 1)
    },
    // 监听图片上传成功的时间
    handleSuccess(response) {
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    // 添加商品
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        // 添加业务逻辑处理
        // lodash cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 5px 0 0 !important;
}
  .previewImg {
    width: 100%;
  }
  .btnAdd{
    margin-top: 15px;
  }
</style>
