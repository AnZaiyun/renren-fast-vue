<template>
  <!-- <div class="custom-tree-container">
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
    ></el-tree>
  </div>
</div> -->

  <div class="block">
    <p>使用 scoped slot</p>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      default-expand-all
      :props="defaultProps"
      :expand-on-click-node="false"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
          <el-button v-if="node.childNodes.length==0" type="text" size="mini" @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
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
        console.log("sucess", data.categories);
        this.menus = data.categories;
      });
    },
    append(data) {
      console.log("append",data)
    },

    remove(node, data) {
      console.log("remove",node,data)
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
  activated() {}
}
</script>

<style scoped>
</style>