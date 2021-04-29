<template>
 <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
data() {
      return {
        menus: []
    };
    
  },
  methods: {
      // 获取数据列表
      getMenus () {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          this.menus = data.data;
        })
      },
    },
    created() {
      this.getMenus();
    },
}
</script>

<style>

</style>