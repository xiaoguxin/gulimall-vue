<template>
  <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeclick"
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
    nodeclick(data,node,component){
        console.log("子组件category的节点被点击",data,node,component);
        //向父组件发送事件;
        this.$emit("tree-node-click", data, node, component);
    }
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