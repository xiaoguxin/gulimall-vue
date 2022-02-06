<template>
  <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
    >
  </el-tree>
</template>

<script>
export default {
  components: {},
  props: {},
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
  methods: {
    // 获取数据列表
    getMenus() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据...", data.data);
        this.menus = data.data;
      });
    },
  },
  //生命周期-创建完成（可以访问当前this实例）
  created() {
    //一旦创建组件，就调用该方法发起请求获取数据
    this.getMenus();
  },
};
</script>

<style>
</style>