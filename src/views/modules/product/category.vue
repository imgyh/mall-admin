<template>
  <div>
    <el-tree :data="category" :props="defaultProps" show-checkbox node-key="catId" :expand-on-click-node="false"
      :default-expanded-keys="expandkey" draggable :allow-drop="allowDrop">

      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            新增
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            修改
          </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="categoryform">
        <el-form-item label="分类名称" :required="true">
          <el-input v-model="categoryform.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="categoryform.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类单位">
          <el-input v-model="categoryform.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>


<script>
export default {
  data() {
    return {
      maxLevel: 0,
      type: '',
      title: '',
      // 值为 null 时 sql语句不会有该字段 不会对其修改
      categoryform: { catId: null, name: null, parentCid: null, catLevel: null, showStatus: null, sort: null, icon: null, productUnit: null, productCount: null },
      dialogFormVisible: false,
      category: [],
      expandkey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    getCategoryTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        this.category = data.data
        // console.log(data.data)
      })
    },
    allowDrop(draggingNode, dropNode, type) {
      // console.log(draggingNode, dropNode, type)
      this.maxLevel = 0
      // 获取最大的 catLevel [0,1,2]三级分类
      this.getDeepth(draggingNode.data)
      // 拖拽节点的最大深度
      let deepth = this.maxLevel - draggingNode.data.catLevel + 1
      if(type=='inner'){
        return (deepth + dropNode.data.catLevel) < 3
      }else{
        return (deepth + dropNode.data.parentCid) <= 3
      }
    },
    getDeepth(node){
      if(node.children != null && node.children.length > 0){
        for(let i = 0; i < node.children.length; i++){
          if(node.children[i].catLevel > this.maxLevel){
            this.maxLevel = node.children[i].catLevel
          }
          this.getDeepth(node.children[i])
        }
      }
    },
    submitCategory() {
      if (this.type === 'add') {
        this.addCategory()
      } else if (this.type === 'edit') {
        this.editCategory()
      } else {
        this.dialogFormVisible = false
      }
    },
    edit(data) {
      this.type = 'edit'
      this.title = '修改分类'
      this.dialogFormVisible = true

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        if (data && data.code === 0) {
          // console.log(data)
          this.categoryform.catId = data.data.catId
          this.categoryform.name = data.data.name
          this.categoryform.parentCid = data.data.parentCid
          this.categoryform.catLevel = null // 值为 null 时 sql语句不会有该字段 不会对其修改
          this.categoryform.showStatus = null // 值为 null 时 sql语句不会有该字段 不会对其修改
          this.categoryform.sort = null // 值为 null 时 sql语句不会有该字段 不会对其修改
          this.categoryform.icon = data.data.icon
          this.categoryform.productUnit = data.data.productUnit
          this.categoryform.productCount = null // 值为 null 时 sql语句不会有该字段 不会对其修改
        } else {
          this.$message.error(data.msg)
        }
      })


    },
    editCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(this.categoryform, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: '修改成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
            }
          })
        } else {
          this.$message.error(data.msg)
        }
        // 刷新Category
        this.getCategoryTree()
        // 设置展开的节点
        this.expandkey = [this.categoryform.parentCid]
        this.dialogFormVisible = false
      })

    },
    // 新增 不需要设置 catId
    append(data) {
      this.type = 'add'
      this.title = '新增分类'
      this.dialogFormVisible = true

      this.categoryform.catId = null // 值为 null 时 sql语句不会有该字段 不会对其修改
      this.categoryform.name = null // 表单提供
      this.categoryform.parentCid = data.catId
      this.categoryform.catLevel = data.catLevel * 1 + 1
      this.categoryform.showStatus = 1
      this.categoryform.sort = 0
      this.categoryform.icon = null // 表单提供
      this.categoryform.productUnit = null // 表单提供
      this.categoryform.productCount = 0
    },
    addCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categoryform, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: '保存成功',
            type: 'success',
            duration: 1000,
            onClose: () => {
            }
          })
        } else {
          this.$message.error(data.msg)
        }
        // 刷新Category
        this.getCategoryTree()
        // 设置展开的节点
        this.expandkey = [this.categoryform.parentCid]
        this.dialogFormVisible = false
      })
    },
    remove(node, data) {
      var ids = [data.catId]

      this.$confirm(`此操作将永久删除 [${data.name}] 分类, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete/batch'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: '删除成功',
              type: 'success',
              duration: 1000,
              onClose: () => {
              }
            })
          } else {
            this.$message.error(data.msg)
          }
          // 刷新Category
          this.getCategoryTree()
          // 设置展开的节点
          this.expandkey = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }

  },
  created() {
    this.getCategoryTree()
  }
}
</script>
