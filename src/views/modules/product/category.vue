<template>
    <el-tree :data="category" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>


<script>
export default {
    data() {
        return {
            category: [],
            defaultProps: {
                children: 'children',
                label: 'name'
            }
        };
    },
    methods: {
        handleNodeClick(data) {
            console.log(data);
        },
        getCategoryTree() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                this.category = data.data
                console.log(data.data)
            })
        }
    },
    created () {
      this.getCategoryTree()
    }
};
</script>
