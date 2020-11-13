<template>
  <div class="block">
    <el-switch
      v-model="isDraggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button type="danger" @click="removeCheckedNodes">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedIds"
      :props="defaultProps"
      :expand-on-click-node="false"
      :draggable="isDraggable"
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
            @click="
              () => {
                append(data);
              }
            "
          >
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="
              () => {
                edit(data);
              }
            "
          >
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
      title="商品菜单维护"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="categoty">
        <el-form-item label="菜单名称" :label-width="formLabelWidth">
          <el-input v-model="categoty.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="() => saveData(categoty.name)"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
// 这里可以导入其他文件，如组件、工具js、第三方插件js、json文件、图片文件等
// 例如：import 《组件名称》 from 《组件路径》

export default {
  components: {},
  props: {},
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      dialogTableVisible: false,
      dialogFormVisible: false,
      expandedIds: [],
      categoty: {
        name: "",
        catLevel: 0,
        parentCid: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
      },
      fromType: "",
      formLabelWidth: "120px",
      maxLen: 0,
      isDraggable: true,
      draggingNodes: [],
    };
  },
  // 计算属性 类似data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 方法集合
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.categories;
      });
    },
    append(data) {
      //给要增加的分类添加初始值
      this.categoty.catId = null;
      this.categoty.name = null;
      this.categoty.catLevel = data.catLevel * 1 + 1;
      this.categoty.parentCid = data.catId;

      this.fromType = "append";

      //对话框弹出
      this.dialogFormVisible = true;
    },
    edit(data) {
      console.log("edit", data);
      //发送请求获取该节点的最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("回显的数据", data);
        //给要增加的分类添加初始值
        this.categoty.name = data.category.name;
        this.categoty.catId = data.category.catId;
        this.categoty.parentCid = data.category.parentCid;

        console.log("数据展示", this.categoty);
      });

      this.fromType = "edit";

      //对话框弹出
      this.dialogFormVisible = true;
    },

    saveData(name) {
      this.categoty.name = name;
      if (this.fromType == "append") {
        this.addCategoty();
      } else if (this.fromType == "edit") {
        this.editCategoty();
      }
    },

    addCategoty() {
      this.$confirm(`请确认是否增加菜单${this.categoty.name}`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/add"),
            method: "post",
            data: this.$http.adornData(this.categoty, false),
          }).then(({ data }) => {
            this.$message({
              message: `菜单新增成功`,
              type: "success",
            });
            this.getMenus();
          });
          this.dialogFormVisible = false;
          this.expandedIds = [this.categoty.parentCid];
        })
        .catch(() => {});
    },
    editCategoty() {
      var { name, catId } = this.categoty;
      var data = { name: name, catId: catId };
      this.$confirm(`请确认是否修改菜单${data.name}`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/update"),
            method: "post",
            data: this.$http.adornData(data, false),
          }).then(({ data }) => {
            this.$message({
              message: `菜单修改成功`,
              type: "success",
            });
            this.getMenus();
          });
          this.dialogFormVisible = false;
          this.expandedIds = [this.categoty.parentCid];
        })
        .catch(() => {});
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`此操作将永久删除菜单【${data.name}】, 是否继续?`, "提示", {
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
              message: `菜单已删除`,
              type: "success",
            });
            this.getMenus();
          });
        })
        .catch(() => {
          this.$message("删除操作已取消");
        });
    },

    allowDrop(draggingNode, dropNode, type) {
      //被拖动的当前节点以及所在的父节点总层数不能大于3
      this.maxLen = 0;
      this.calculLen(draggingNode);
      //计算拖动节点的最大长度：子节点深度-当前节点深度+1=整个树的深度
      this.maxLen = this.maxLen - draggingNode.data.catLevel + 1;
      //加上要移动到的节点的当前深度，便是移动后该节点的深度
      this.maxLen = this.maxLen + dropNode.data.catLevel;

      if (this.maxLen > 3 && type == "inner") {
        return false;
      } else {
        return true;
      }
    },
    calculLen(node) {
      //如果当前节点的层级大于最大层级，则最大层级为当前层级
      if (node.data.catLevel > this.maxLen) {
        this.maxLen = node.data.catLevel;
      }

      //递归查询子节点层级
      if (node.childNodes != null && node.childNodes.length != 0) {
        for (var i = 0; i < node.childNodes.length; i++) {
          this.calculLen(node.childNodes[i]);
        }
      }
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // console.log(draggingNode, dropNode.data.catLevel, dropType)
      if (dropNode != null) {
        this.categoty.catId = draggingNode.data.catId;
        if (dropType != "inner") {
          //如果不是拖拽到一个节点里，那么当前节点的父id就是拖拽到节点的父id，层级一致
          draggingNode.data.parentCid = dropNode.data.parentCid;
          draggingNode.data.catLevel = dropNode.data.catLevel;
          draggingNode.data.sort = dropNode.data.sort;

          var { catId, parentCid, catLevel ,sort} = draggingNode.data;
          //先把当前拖拽的节点放入要更新的节点中
          this.draggingNodes.push({ catId, parentCid, catLevel ,sort});
          
        } else {
          //如果是拖拽到一个节点里，那么当前节点的父id就是拖拽到节点的id，层级为拖拽到节点的层级+1
          draggingNode.data.parentCid = dropNode.data.catId;
          draggingNode.data.catLevel = dropNode.data.catLevel * 1 + 1;
          //对拖拽到节点下的数据顺序进行重排，其实这个排序也还是不准确，这里只是实现一个排序的大致效果
          for (var i = 0; i < dropNode.childNodes.length; i++) {
            dropNode.childNodes[i].data.sort = i;

            var { catId, parentCid, catLevel ,sort} = dropNode.childNodes[i].data;
            //先把当前拖拽的节点放入要更新的节点中
            this.draggingNodes.push({ catId, parentCid, catLevel ,sort });
          }
        }

        //遍历子节点,更新拖拽节点子节点的层级
        this.updateDraggingNodes(draggingNode);

        console.log(this.draggingNodes)

        //发送请求，将拖拽节点的信息更新入库，需要注意的是，拖拽节点的子节点的层级也要更新，这部分放到后台去更新
        this.$http({
          url: this.$http.adornUrl("/product/category/draggingNodeUpdate"),
          method: "post",
          data: this.$http.adornData(this.draggingNodes, false),
        }).then(({ data }) => {
          this.$message({
            message: `菜单更新成功`,
            type: "success",
          });
          this.getMenus();
          //选择展开的节点
          this.expandedIds = [draggingNode.data.parentCid];
        });

        console.log("handleDrop:", draggingNode, dropNode, dropType);
      }
    },
    updateDraggingNodes(nodes) {
      for (var i = 0; i < nodes.childNodes.length; i++) {
        nodes.childNodes[i].data.catLevel = nodes.data.catLevel * 1 + 1;
        var { catId, catLevel } = nodes.childNodes[i].data;
        this.draggingNodes.push({ catId, catLevel });
        this.updateDraggingNodes(nodes.childNodes[i]);
      }
    },
    removeCheckedNodes() {
      //获取所有已选择的节点，需要注意的是一级节点不可删除，所以需要对获取到的节点进行删除
      var CheckedNodes = this.$refs.menuTree.getCheckedNodes();
      var ids = [];
      for (var i = 0; i < CheckedNodes.length; i++) {
        if (CheckedNodes[i].parentCid != 0) {
          ids.push(CheckedNodes[i].catId);
        }
      }

      console.log(ids);
      this.$confirm(`此操作将永久删除选择菜单, 是否继续?`, "提示", {
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
              message: `菜单已删除`,
              type: "success",
            });
            this.getMenus();
          });
        })
        .catch(() => {
          this.$message("删除操作已取消");
        });
    },
  },
  created() {
    this.getMenus();
  },
  mounted() {},
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  activated() {},
};
</script>

<style scoped>
</style>