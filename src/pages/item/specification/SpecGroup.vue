<template>
  <div>
    <!-- 定义表格 -->
    <v-data-table>

    </v-data-table>
  </div>
</template>

<script>
    export default {
      name: "spec-group",
      props:{
        cid:{
          type:Number,
          default:0
        }
      },
      data(){
        return{
          headers:[
            {text:"id",align:"center",value:"id"},
            {text:"组名",align:"center",value:"name",sortable:false},
            {text:"操作",align:"center",sortable:false}
          ],
          groups:[],    // 当前页的规格内容
          show:false,   // 是否打开编辑窗口
          isEdit:false, // 是否是编辑
          group:{name:""}   // 初始化被选中的分组
        }
      }
    }
</script>

<style scoped>

</style>
