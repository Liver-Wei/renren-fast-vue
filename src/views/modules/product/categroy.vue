<template>
  <div>
    <el-button type="danger" @click="batchDelete" round>批量删除</el-button>
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandKey"
      ref="menuTree"
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
            添加
          </el-button>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span></el-tree
    >
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
      </el-form>
      <el-form :model="category">
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
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
export default {
  data() {
    return {
      title: "",
      category: {
        icon: "",
        productUnit: "",

        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        dialogType: ""
      },
      dialogVisible: false,
      data: [],
      expandKey: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    batchDelete(){
      let checkKeys = this.$refs.menuTree.getCheckedKeys();
      console.log("选中的节点",checkKeys);
      this.$confirm(`是否批量删除菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(checkKeys, false)
          }).then(() => {
            console.log("删除成功...");
            this.$message({
              type: "success",
              message: `删除成功!`
            });
            this.getmenus();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
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
      (this.title = "添加分类"),
        (this.category.dialogType = "add"),
        (this.dialogVisible = true);
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    edit(data) {
      console.log("要修改的数据：", data);
      (this.title = "修改分类"), (this.category.dialogType = "edit");
      this.dialogVisible = true;

      //获取当前节点最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        console.log("------------------");
        console.log("查看回调数据：", data);
      });
    },
    submitData() {
      if (this.category.dialogType == "add") {
        this.addCategory();
      } else {
        this.editCategory();
      }
    },
    editCategory() {
      var { name, catId, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ name, catId, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单修改成功！"
        });
        this.dialogVisible = false;
        this.getmenus();
        this.expandKey = [this.category.parentCid];
      });
    },
    addCategory() {
      console.log("提交的数据：", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单保存成功！"
        });
        this.dialogVisible = false;
        this.getmenus();
        this.expandKey = [this.category.parentCid];
      });
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
            this.expandKey = [node.parent.data.catId];
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
