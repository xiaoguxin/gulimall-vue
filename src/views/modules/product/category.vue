<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
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
      pCid: [],
      draggable: false,
      updateNodes: [],
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
    batchDelete() {
      let catIds = [];
      let catNames = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的元素", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
        catNames.push(checkedNodes[i].name);
      }
      this.$confirm(`是否批量删除【${catNames}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "菜单批量删除成功!",
            });
            //刷新出新菜单(可能多人删除)
            this.getMenus();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消批量删除",
          });
        });
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/list"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单拖拽更新成功!",
        });
        //刷新出新菜单
        this.getMenus();
        //设置展开
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        //this.pCid = 0;
      });
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);
      //1.当前拖动节点后最新的父id
      let pCid = 0;
      if (dropType == "before" || dropType == "after") {
        //可能为根节点undefined
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
      } else {
        pCid = dropNode.data.catId;
      }
      this.pCid.push(pCid);

      //2.当前拖拽节点的最新顺序（更新拖拽节点顺序和父级id,更新节点兄弟节点顺序）
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        siblings = dropNode.parent.childNodes;
      } else {
        siblings = dropNode.childNodes;
      }

      for (let i = 0; i < siblings.length; i++) {
        //筛选出当前拖拽的那个节点，更新父id(自己要改父类id和顺序)
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //第三点相关：判断拖拽后节点和拖拽节点的层级是否发生变化
          let catLevel = draggingNode.level; //默认层级
          if (siblings[i].level != draggingNode.level) {
            //拖拽节点层级发生变化
            // if(dropType=="before"||dropType=="after"){
            //     //和拖拽后节点层级一样
            //     catLevel=dropNode.level;
            // }else{
            //     //在拖拽后节点层级内,则level+1
            //     catLevel=dropNode.level+1;
            // }
            catLevel = siblings[i].level; //最新层级

            //修改它子节点的层级（递归）
            this.updateChildNodeLevel(siblings[i]);
          }

          this.updateNodes.push({
            catId: siblings[i].data.catId,
            parentCid: pCid,
            sort: i,
            catLevel: catLevel,
          });
        } else {
          //修改拖拽节点和兄弟节点的顺序(兄弟只改顺序)
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      console.log("updateNodes", this.updateNodes);

      //3.当前拖拽节点的最新层级（更新拖拽节点层级和子节点的层级）
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          //可能还有子节点:继续修改层级
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    //控制拖拽时判定目标节点能否被放置
    allowDrop(draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3

      //1）、被拖动的当前节点总层数
      console.log("allowDrop", draggingNode, dropNode, type);
      //
      this.countNodeLevel(draggingNode);
      console.log("this.maxLevel--：", this.maxLevel);
      //当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = this.maxLevel - draggingNode.level + 1;
      console.log("节点携带层数（深度）--：", deep);

      //拖拽完maxLevel恢复默认值
      this.maxLevel = 0;

      //this.maxLevel
      if (type == "inner") {
        if (deep + dropNode.level <= 3) {
          console.log("inner：移动成功");
          return true;
        } else {
          console.log("inner：移动失败");
          return false;
        }
        //return deep + dropNode.level <= 3;
      } else {
        if (deep + dropNode.parent.level <= 3) {
          console.log("移动成功");
          return true;
        } else {
          console.log("移动失败");
          return false;
        }
        //return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.childNodes != null) {
        if (node.childNodes.length > 0) {
          for (let i = 0; i < node.childNodes.length; i++) {
            if (node.childNodes[i].level > this.maxLevel) {
              this.maxLevel = node.childNodes[i].level;
            }
            this.countNodeLevel(node.childNodes[i]);
          }
        } else {
          console.log("node.childNodes.length<=0");
          this.maxLevel = node.level;
        }
      }

      // if (node.children != null && node.children.length > 0) {
      //   for (let i = 0; i < node.children.length; i++) {
      //     if (node.children[i].catLevel > this.maxLevel) {
      //       this.maxLevel = node.children[i].catLevel;
      //     }
      //     this.countNodeLevel(node.children[i]);
      //   }
      // }
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