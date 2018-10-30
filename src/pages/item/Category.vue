<template>
  <v-card>
      <v-flex xs12 sm10>
        <v-tree url="/item/category/list"
                :isEdit="isEdit"
                @handleAdd="handleAdd"
                @handleEdit="handleEdit"
                @handleDelete="handleDelete"
                @handleClick="handleClick"
        />
      </v-flex>
  </v-card>
</template>

<script>
  export default {
    name: "category",
    data() {
      return {
        isEdit:true,
      }
    },
    methods: {
      // 新增
      handleAdd(node) {
        // 弹窗输入分类的名称
        const category = Object.deepCopy(node);
        // 发送Ajax请求往数据库中增加分类
        this.$http({
          method:"post",
          url:"/item/category",
          data:this.$qs.stringify(category)
        }).then(() => {
          this.$message.success("新的分类保存成功")

        }).catch(() => {
          this.$message.error("新的分类保存失败")
        })
      },
      handleEdit(id, name) {
        // 发送Ajax请求至后台进行修改
        this.$http({
          method:"put",
          url:"/item/category",
          data:this.$qs.stringify({id:id,name:name})
        }).then(() => {
          this.$message.success("修改成功");
        }).catch(() => {
          this.$message.error("修改失败");
        })

      },
      handleDelete(id) {
        console.log("delete ... " + id);
        // 先从后台查询该分类下是否有商品数据
        this.$http.delete("/item/category/confirm/" + id).then(({data})=>{
          // 如果存在返回的信息，则进行弹窗确认是否删除分类及其关联数据
          if (data) {
            this.$message.confirm(data).then(() => {
              // 确认删除，发送请求进行删除
              this.$http.delete("/item/category/" + id).then(() => {
                this.$message.success("删除成功")
              }).catch(() => {
                this.$message.error("删除失败")
              })
            }).catch(() => {
              this.$message.info("已取消删除")
            });
          } else {
            this.$message.success("删除成功");
          }
        }).catch(() => {
          this.$message.error("删除失败")
        });


      },
      handleClick(node) {
        console.log(node)
      }
    }
  };
</script>

<style scoped>

</style>
