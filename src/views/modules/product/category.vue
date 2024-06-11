<template xmlns="http://www.w3.org/1999/html">
  <div>
    <el-tree :data="categories" :props="defaultProps" :expand-on-click-node="false"
             node-key="catId"
             show-checkbox
             :default-expanded-keys="expandedKeys">
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini"
                     @click="() => openView(data,'add')">Append</el-button>
          <el-button v-if="node.level <= 2" type="text" size="mini"
                     @click="() => openView(data,'edit')">Edit</el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
                     @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="categoryForm">
        <el-form-item label="分类名称">
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="图标">
          <el-input v-model="categoryForm.icon" size="mini" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="计量单位">
          <el-input v-model="categoryForm.productUnit" size="mini" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="排序">
          <el-input-number v-model="categoryForm.sort" size="mini" autocomplete="off"></el-input-number>
        </el-form-item>

        <el-form-item label="是否展示">
          <el-select v-model="categoryForm.showStatus" size="mini" placeholder="请选择">
            <el-option
              v-for="item in showOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <!--          <el-input v-model="categoryForm.showStatus" autocomplete="off"></el-input>-->
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="()=>saveOrUpdateCategory(this.categoryForm)">确 定</el-button>
      </div>
    </el-dialog>
  </div>

</template>

<script>
export default {
  data () {
    return {
      showOptions: [{
        value: '1',
        label: '是'
      }, {
        value: '0',
        label: '否'
      }],
      categoryForm: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: '',
        icon: null,
        productUnit: '',
        sort: 0
      },
      dialogFormVisible: false,
      expandedKeys: [],
      categories: [],
      defaultProps: {
        children: 'categoryEntities',
        label: 'name'
      },
      title: '',
      operation: ''
    }
  },
  methods: {
    // 查询菜单数据 渲染视图
    queryCategory () {
      this.$http({
        url: this.$http.adornUrl('/ecmall4cloudproduct/category/listTree'),
        method: 'post'
        // params: this.$http.adornParams()
      }).then(({data}) => {
        this.categories = data.data
      })
    },
    openView (data, operation) {
      this.dialogFormVisible = true
      this.operation = operation
      if (operation === 'edit') {
        this.title = '修改商品分类'
        this.getCategory(data)
      } else {
        this.title = '添加商品分类'
        this.categoryForm = {
          catId: null,
          name: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: '',
          icon: null,
          productUnit: '',
          sort: 0
        }
        this.categoryForm.parentCid = data.catId
        this.categoryForm.catLevel = data.catLevel + 1
      }
    },
    getCategory (data) {
      this.$http({
        url: this.$http.adornUrl(`/ecmall4cloudproduct/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        this.categoryForm = data.category
      })
    },
    saveOrUpdateCategory () {
      let pathSuffix = this.operation === 'edit' ? 'update' : 'save'
      this.$http({
        url: this.$http.adornUrl(`/ecmall4cloudproduct/category/${pathSuffix}`),
        method: 'post',
        data:
          this.$http.adornData(this.categoryForm, false)
      }).then(({data}) => {
        this.expandedKeys = [this.categoryForm.parentCid]
        this.dialogFormVisible = false
        this.queryCategory()
      })
    },
    remove (node, data) {
      let ids = [data.catId]
      this.$confirm(`是否删除菜单[${data.name}]?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/ecmall4cloudproduct/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          // 确定后的操作
          if (data && data.code === 0) {
            this.queryCategory()
            this.expandedKeys = [node.parent.data.catId]
            this.$message({
              // onClose: () => {
              //   this.queryCategory()
              //   this.expandedKeys = [node.parent.data.catId]
              // },
              message: '操作成功',
              type: 'success',
              duration: 1500
            })
          } else {
            this.$message.error(data.msg)
          }
        }).catch(() => {
          // 取消后的操作
        })
      })
    }
  },
  created () {
    this.queryCategory()
  }
}
</script>
