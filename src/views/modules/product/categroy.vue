<template>
  <el-tree
    :data="data"
    :props="defaultProps"
    :expand-on-click-node="false"
    :default-expanded-keys = "expandKey"
    show-checkbox
    node-key="catId"
    ><span class="custom-tree-node" slot-scope="{ node, data }">
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
      </span>
    </span></el-tree
  >
</template>

<script>
export default {
  data() {
    return {
      data: [],
      expandKey:[],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    getmenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        console.log("获取菜单数据：", data.page);
        this.data = data.page;
      });
    },
    append(data) {
      console.log("append", data);
    },
    remove(node, data) {
      let ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
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
            console.log("删除成功...");
            this.$message({
              type: "success",
              message: `${data.name}删除成功!`
            });
            this.getmenus();
            this.expandKey=[node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });

      console.log("remove", node, data);
    }
  },
  created() {
    this.getmenus();
  }
};
</script>

<style></style>
