<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
      draggable
      :allow-drop="allowDrop"
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
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      maxLevel: 0,
      title: "",
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null,
      },
      dialogVisible: false,
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

    append(data) {
      this.title = "添加分类";
      this.dialogType = "add";
      this.dialogVisible = true;
      //添加所需数据
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      //设置回默认值
      this.category.name = "";
      this.category.catId = null; //自增
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      console.log("append", data);
    },
    edit(data) {
      console.log("edit", data);
      this.title = "修改分类";
      this.dialogType = "edit";
      this.dialogVisible = true;
      //编辑的数据：通过发送请求获取当前节点最新的数据
      //this.category.name = data.name;
      //this.category.catId= data.catId;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功
        console.log("成功获取到节点的数据(要回显的数据)...", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    //修改分类数据
    editCategory() {
      console.log("编辑表单数据", this.category);
      //解构出需要编辑的字段
      var { catId, name, icon, productUnit } = this.category;

      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单修改成功!",
        });
        //关闭弹框
        this.dialogVisible = false;
        //刷新出新菜单
        this.getMenus();
        //设置展开
        this.expandedKey = [this.category.parentCid];
      });
    },
    //添加分类数据
    addCategory() {
      console.log("提交表单数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单保存成功!",
        });
        //关闭弹框
        this.dialogVisible = false;
        //刷新出新菜单
        this.getMenus();
        //设置展开
        this.expandedKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      //var parentCid=data.parentCid;
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "菜单删除成功!",
            });
            //刷新出新菜单
            this.getMenus();
            //设置展开
            this.expandedKey = [node.parent.data.catId];
            //this.expandedKey = [parentCid]
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
      console.log("remove", node, data);
    },
    //控制拖拽时判定目标节点能否被放置
    allowDrop(draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3

      //1）、被拖动的当前节点总层数
      console.log("allowDrop", draggingNode, dropNode, type);
      //
      this.countNodeLevel(draggingNode.data);
      //当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;
      console.log("深度--：", deep);

      //拖拽完maxLevel恢复默认值
      this.maxLevel=0;
      
      //this.maxLevel
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
      
    },
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel;
          }
          this.countNodeLevel(node.children[i]);
        }
      }
    },
    handleNodeClick(data) {
      //console.log(data);
      //this.dialogVisible = true;
      //this.category.name = data.name;
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