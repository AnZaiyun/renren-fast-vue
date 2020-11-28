<template>
    <div class="block">
        <el-tree :data="menus" node-key=" cscatId" :props="defaultProps" ref="menuTree" @node-click="nodeclick"></el-tree>
    </div>
</template>
<script>
//这里可以导入其他文件，如组件、工具js、第三方插件js、json文件、图片文件等
//例如：import 《组件名称》 from 《组件路径》
export default {
    components: {},
    props: {},
    data() {
        //这里存放数据
        return {
            menus: [],
            defaultProps: {
                children: "children",
                label: "name",
            },
        };
    },
    //计算属性 类似data概念
    computed: {},
    //监控data中的数据变化
    watch: {},
    //方法集合
    methods: {
        getMenus() {
            this.$http({
                url: this.$http.adornUrl("/product/category/list/tree"),
                method: "get",
            }).then(({ data }) => {
                this.menus = data.categories;
            });
        },
        nodeclick(data, node, component) {
            // console.log('节点被点击', data,node,component)
            //向父组件发送事件
            this.$emit("tree-node-click",data, node, component)
        }
    },
    created() {
        this.getMenus();
    },
    mounted() { },
    beforeCreate() { },
    beforeMount() { },
    beforeUpdate() { },
    updated() { },
    beforeDestroy() { },
    destroyed() { },
    activated() { },
};
</script>
<style scoped>
</style>