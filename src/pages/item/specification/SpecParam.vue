<template>
  <div>
    <!-- 定义表格 -->
    <v-data-table
    :headers="headers"
    :items="params"
    hide-actions
    class="elevation-0"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{props.item.id}}</td>
        <td class="text-xs-center">{{props.item.name}}</td>
        <td class="text-xs-center">{{props.item.numeric ? "是" : "否"}}</td>
        <td class="text-xs-center">{{props.item.unit ? props.item.unit : '无'}}</td>
        <td class="text-xs-center">{{props.item.generic ? "是" : "否"}}</td>
        <td class="text-xs-center">{{props.item.searching ? "是" : "否"}}</td>
        <td class="text-xs-center">
          <!-- 编辑操作的按钮 -->
          <v-btn flat icon @click="editParam(props.item)">
            <v-icon>edit</v-icon>
          </v-btn>
          <!-- 删除操作的按钮 -->
          <v-btn flat icon @click="deleteParam(props.item)">
            <v-icon>delete</v-icon>
          </v-btn>
        </td>
      </template>

    </v-data-table>
    <v-divider/>
    <!-- 新增参数按钮 -->
    <v-btn color="primary" @click="addParam">新增参数</v-btn>

    <!-- 新增和编辑参数的弹窗 -->
    <v-dialog v-model="show" max-width="500" persistent>
      <!-- 中间做成卡片样式 -->
      <v-card>
        <!-- 头部标题 -->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>
            {{isEdit ? "编辑" : "新增"}}参数
          </v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>

        <!-- 对话框的正文，form表单 -->
        <v-card-text class="px-5">
          <spec-param-form @close="closeWindow"
                           :isEdit="isEdit"
                           :oldParam="oldParam"
                           :cid="group.cid"
                           :groupId="group.id"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>

</template>

<script>
  import specParamForm from "./SpecParamForm"
    export default {
      name: "spec-param",
      components: {specParamForm},
      // 接收父组件传递过来的group信息
      props:{
        group:{
          type:Object,
          default:{}
        }
      },
      data(){
          return {
            headers:[
              {text:"id",align:"center",value:"id"},
              {text:"参数名",align:"center",value:"name",sortable:false},
              {text:"是否为数值",align:"center",value:"numeric",sortable:false},
              {text:"单位",align:"center",value:"unit",sortable:false},
              {text:"是否通用",align:"center",value:"generic",sortable:false},
              {text:"是否可搜索",align:"center",value:"searching",sortable:false},
              {text:"操作",align:"center",sortable:false},
            ],
            params:[],      // 当前表格的规格参数
            show:false,     // 是否显示编辑窗口
            isEdit:false,      // 是否是编辑
            oldParam:{},    // 编辑前的参数
          }
      },
      // 监视group值变化，如果group不为空，则调用loadParams方法
      watch:{
        group:{
          deep:true,
          handler(val){
            if (val && val.id) {
              this.loadParams();
            }
          }
        }
      },

      methods:{
        // 加载规格参数的数据
        loadParams(){
          this.$http.get("item/spec/params",{
            params:{
              gid:this.group.id
            }
          })
            .then(({data}) => {
              data.forEach(p => {p.segments = p.segments ? p.segments.split(",") : []})
              this.params = data;
            })
            .catch(() => {
              this.params = []
            })
        },

        // 新增参数
        addParam(){
          this.show = true;
          this.oldParam = {
            cid:this.group.cid,
            groupId:this.group.id,
            name:"",
            numeric:false,
            unit:"",
            generic:false,
            searching:false,
            segments:[]
          };
          this.isEdit = false;
          console.log(this.group)
        },

        // 编辑按钮
        editParam(oldParam){
          this.show = true;
          this.isEdit = true;
          this.oldParam = oldParam;
        },

        // 删除按钮
        deleteParam(oldParam){
          this.$message.confirm("此操作将永久删除该参数，是否继续？")
            .then(() => {
              this.$http.delete("/item/spec/param",{
                params:{
                  id:oldParam.id
                }
              })
                .then(() => {
                  this.$message.success("删除成功");
                  this.loadParams();
                }).catch(() => {
                  this.$message.error("删除失败");
                  this.loadParams();
                })
            })
            .catch(() => {
              this.$message.info("删除已取消");
              this.loadParams();
          })
        },

        // 关闭编辑弹窗
        closeWindow(){
          this.show = false;
          this.dialog = false;
          this.loadParams()
        }
      }
    }
</script>

<style scoped>

</style>
