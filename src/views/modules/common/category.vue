<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
    ></el-tree>
  </div>
</template>

<script>
export default {
  name: "",
  components: {},
  data() {
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    nodeClick(data, node, component) {
      console.log("子组件被点击", data, Node, component);
      this.$emit("tree-node-click", data, Node, component);
    },
  },
};
</script>
<style scoped='scss' scoped>
</style>