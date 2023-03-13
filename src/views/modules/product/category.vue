<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      draggable
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
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
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
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
      <!-- 嵌套表格 -->
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
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      // 对话框标题
      title: "",
      // 对话框类型，增加或修改
      dialogType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        catId: null
      },
      dialogVisible: false,
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      },
      expandedKey: []
    };
  },
  components: {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list"),
        method: "get"
      }).then(({ data }) => {
        this.menus = data.data;
        console.log("成功获取到菜单数据...", this.menus);
      });
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    append(data) {
      console.log("append:", data);
      this.dialogType = "add";
      this.title = "添加分类";
      // 清空
      this.category = {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        catId: null
      };
      this.dialogVisible = true;
      // 计算新节点的默认值
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    addCategory() {
      console.log("提交的分类名:", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(() => {
        this.$message({
          message: `菜单添加成功：${this.category.name}`,
          type: "success"
        });
        // 关闭对话框
        this.dialogVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      console.log("remove:", node, data);
      let ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(() => {
            this.$message({
              message: `菜单删除成功：${data.catId},${data.name}`,
              type: "success"
            });
            // 刷新菜单
            this.getMenus();
            //让父节点默认展开
            this.expandedKey = [data.parentCid];
          });
        })
        .catch(() => {});
    },

    edit(data) {
      console.log("edit:", data);
      this.dialogType = "edit";
      this.title = "修改分类";
      this.dialogVisible = true;

      // 发送请求，获得目标分类最新信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        console.log("回显数据:", data);
        // 让输入框内默认为原值
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    editCategory() {
      let { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(
          {
            catId,
            name,
            icon,
            productUnit
          },
          false
        )
      }).then(() => {
        this.$message({
          message: `菜单修改成功：${data.catId},${this.category.name}`,
          type: "success"
        });
        // 关闭对话框
        this.dialogVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    /**
     * 判断能否放置
     * @param {被拖动的结点} draggingNode
     * @param {目标结点} dropNode
     * @param {相对位置} type prev/inner/next
     */
    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop: ", draggingNode, dropNode, type);

      //被拖动的当前结点加上目标结点的总层数不超过3
      let draggingMaxLevel = this.countNodeMaxGlobalDepth(draggingNode.data);

      // 被拖曳的结点的深度（包括自己到叶子的层数）
      let depth = draggingMaxLevel - draggingNode.data.catLevel + 1;
      console.log("depth: ", depth);

      // 判断是否可以放置
      if (type == "inner") {
        return depth + dropNode.level <= 3; //如有一层孩子的结点（depth==2）拖到根结点（catLevel==1）则刚好是3
      } else {
        //相当于插入到父节点里
        //dropNode.parent可能没有data属性，因此只能用element提供的level属性判断
        return depth + dropNode.parent.level <= 3;
      }
    },
    // 计算结点相对全局的最大深度
    countNodeMaxGlobalDepth(node) {
      let maxLevel = node.catLevel;
      let tempLevel = 0;
      if (node.children == null) {
        return maxLevel;
      }
      // 取子节点获得的最大深度
      for (let i = 0; i < node.children.length; i++) {
        tempLevel = this.countNodeMaxGlobalDepth(node.children[i]);
        if (tempLevel > maxLevel) {
          maxLevel = tempLevel;
        }
      }
      return maxLevel;
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 红豆泥私密马赛，拖拽弃坑！！！
      let newParentCid = 0;
      let siblings=[];// 新兄弟
      if (dropType == "inner") {
        newParentCid = dropNode.data.catId;
        siblings=dropNode.childNodes;
      } else {
        // 根层问题
        newParentCid = dropNode.parent.data.catId;
        siblings=dropNode.parent.childNodes;
      }
    }
  },
  created() {
    this.getMenus();
  }
};
</script>

<style scoped lang="scss"></style>
