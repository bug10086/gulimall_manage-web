<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    ></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
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
          <el-button type="text" size="mini" @click="edit(data)">
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

    <el-dialog
      :title="isEdit ? '修改分类' : '新增分类'"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
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
      pCid: [],
      updateNodes: [],
      isEdit: false,
      maxLevel: 1,
      menus: [],
      expandedKey: [],
      dialogFormVisible: false,
      draggable: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
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
    // 打开新增弹窗
    append(data) {
      console.log(data);
      this.dialogFormVisible = true;
      this.category.name = "";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;

      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    // 打开修改弹窗
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
    submit() {
      if (this.isEdit) {
        this.editCategory();
      } else {
        this.addCategory();
      }
    },
    // 发送添加分类请求
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
    // 发送修改分类请求
    editCategory() {
      var { name, icon, catId, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ name, icon, catId, productUnit }, false),
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
    // 发送删除分类请求
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
    // 判断菜单是否可以拖拽
    allowDrop(draggingNode, dropNode, type) {
      this.countNodeLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      // this.maxLevel = 0;

      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    // 计算节点深度
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            // 是赋值给了共享变量maxLevel
            this.maxLevel = node.childNodes[i].level;
          }
          // 递归子节点
          this.countNodeLevel(node.childNodes[i]);
        }
      } else {
        this.maxLevel = node.level;
      }
    },
    // 拖拽过程
    handleDrop(draggingNode, dropNode, dropType, ev) {
      let pCid = 0;
      let siblings = null;
      if (dropType == "after" || dropType == "before") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);

      for (let i = 0; i < siblings.length; i++) {
        // 如果遍历的是当前正在拖拽的节点
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            catLevel = siblings[i].level;
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
    },
    // 收集修改菜单后需要改变的数据
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    // 批量修改菜单顺序
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单顺序修改成功",
          type: "success",
        });
        this.getMenus();
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = [];
      });
    },
    // 批量删除
    batchDelete() {
      let checkNodesCatIds = this.$refs.menuTree.getCheckedKeys();
      // console.log(checkNodes);
      this.$confirm(`是否批量删除菜单?`, "提示", {
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
            data: this.$http.adornData(checkNodesCatIds, false),
          }).then(({ data }) => {
            // 删除成功$message
            this.$message({
              message: "菜单批量删除成功",
              type: "success",
            });
            //刷新出新的菜单
            this.getMenus();
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