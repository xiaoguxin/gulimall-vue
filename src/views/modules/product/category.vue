<template>
  <div>
    <el-tree
      :data="data"
      :props="defaultProps"
      @node-click="handleNodeClick"
    ></el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      defaultProps: {
        children: "children",
        label: "label",
      },
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    // 获取数据列表
    getMenus () {
        this.dataListLoading = true
        this.$http({
            url: this.$http.adornUrl('/product/category/list/tree'),
            method: 'get'
        }).then(data=>{
            console.log("成功获取到菜单数据...",data)
        })
    }
  },
  //生命周期-创建完成（可以访问当前this实例）
  created(){
      //一旦创建组件，就调用该方法发起请求获取数据
      this.getMenus();
  }
};
</script>

<style>
</style>