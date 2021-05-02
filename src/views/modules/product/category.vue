<template>
  <div>
    <el-tree
      :data="menus"
      show-checkbox
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            增加子菜单
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="edit(data)"
          >
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="isEdit ? '修改分类':'新增分类'" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submit()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isEdit: false,
      menus: [],
      expandedKey: [],
      dialogFormVisible: false,
      category: { name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null , icon:'' , productUnit: '' },
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    // 获取数据列表
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    //打开新增弹窗
    append(data) {
      console.log(data);
      this.dialogFormVisible = true;
      this.category.name = '';
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;

      this.category.catId = null;
      this.category.name = '';  
      this.category.icon = '';
      this.category.productUnit = '';
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    //打开修改弹窗
    edit(data) {
      console.log(data);
      this.dialogFormVisible = true;
      this.isEdit = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category.catId = data.data.catId;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    submit(){
      if(this.isEdit){ 
        this.editCategory();
      }else{
        this.addCategory();
      }
    },
    //发送添加分类请求
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单添加成功",
          type: "success",
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //发送修改分类请求
    editCategory() {
      var {name,icon,catId,productUnit} = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({name,icon,catId,productUnit}, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单添加成功",
          type: "success",
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //发送删除分类请求
    remove(node, data) {
      var ids = [data.catId];
      // 弹窗 确认
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          // 点击确定
          this.$http({
            // 给delete发送
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            // 删除成功$message
            this.$message({
              message: "菜单删除成功",
              type: "success",
            });
            //刷新出新的菜单
            this.getMenus();
            //设置需要默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    handleNodeClick() {},
  },
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>