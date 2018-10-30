<template>
  <div>

    <!-- 定义表格 -->
    <v-data-table
    :headers="headers"
    :items="groups"
    hide-actions
    class="elevation-0"
    v-show="groups.length"
    no-data-text="你想要的，并不存在"
    >
      <template slot="items" slot-scope="props">
        <tr @click="selectGroup(props.item)">
          <td class="text-xs-center" >{{props.item.id}}</td>
          <td class="text-xs-center">{{props.item.name}}</td>
          <td class="text-xs-center">
            <!-- 编辑操作的按钮 -->
            <v-btn flat icon @click.stop="editGroup(props.item)">
              <v-icon>edit</v-icon>
            </v-btn>
            <!-- 删除操作的按钮 -->
            <v-btn flat icon @click.stop="deleteGroup(props.item)">
              <v-icon>delete</v-icon>
            </v-btn>
          </td>
        </tr>
      </template>

    </v-data-table>
    <v-divider/>
    <!-- 新增分组按钮 -->
    <div>
      <v-btn color="primary" @click="addGroup" v-show="groups.length">新增分组</v-btn>
    </div>


    <!-- 编辑和删除的操作弹窗 -->

    <v-dialog v-model="show" max-width="500" persistent>
      <!-- 内部做成卡片 -->
      <v-card>
        <!-- 对话框的标题:dense:紧凑排布; dark:黑暗主题;  -->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? "编辑" : "新增"}}组名</v-toolbar-title>
          <v-spacer/>
          <!-- 关闭窗口的按钮 -->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>

        <!-- 对话框的正文,表单 -->
        <v-card-text class="px-5">
          <v-form v-model="valid" ref="specGroup">
            <v-text-field v-model="group.name" label="请输入组名" required :rules="nameRules"/>

            <!-- 提交和清空按钮 ,my-4:增大上下边距 -->
            <v-layout class="my-4" row>
              <v-btn @click="submit" color="primary">提交</v-btn>
              <v-btn @click="clear">重置</v-btn>
            </v-layout>
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>

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
          oldGroup:{},   // 编辑前的信息
          group:{       // 编辑时新的数据
            id:0,
            cid:this.cid,
            name:""
          },
          valid:true,
          isEdit:false,
          nameRules:[
            v => !!v || "组名不能为空"
          ]
        }
      },
      // 监视cid属性，
      watch:{
        cid(){
          this.loadData();
        },
        oldGroup:{
          handler(val){
            // 复制要编辑的group，防止编辑时候修改了老数据
            this.group = Object.deepCopy(val);
          }
        }
      },
      methods:{
        // 加载规格分组表格信息的方法
        loadData(){
          this.$http.get("item/spec/groups/" + this.cid)
            .then(({data}) => {
              if (data) {
                this.groups = data;
              } else {
                this.groups = [];
              }
            })
            .catch(() => {
              this.$message.error("服务器异常")
            });
        },
        // 点中一条规格组显示规格参数表格
        selectGroup(group){
          // 子组件触发父组件的select事件
          this.$emit("select",group);

        },

        // 编辑一条数据
        editGroup(oldGroup){
          // 回显当前要编辑的组名
          const {params,...old} = oldGroup;
          this.oldGroup = old;
          this.show = true;
          this.isEdit = true;
        },

        // 提交
        submit(){
          // 表单验证
          if (this.$refs.specGroup.validate()) {
            // 发送Ajax请求至后台进行编辑
            this.$http({
              method:this.isEdit ? "put" : "post",
              url:"item/spec/group",
              data:this.$qs.stringify(this.group)
            })
              .then(() => {
                this.closeWindow();
                this.$message.success("保存成功");
                this.closeWindow();
                this.loadData();
              })
              .catch(() => {
                this.$message.error("保存失败");
                this.closeWindow();
                this.loadData();
              });
          }
        },

        // 清空编辑的内容
        clear(){
          this.$refs.specGroup.reset();
        },

        // 删除一条数据
        deleteGroup(group){
          this.$message.confirm("此操作将永久删除该组名，是否继续？")
            .then(() => {
              // 发送Ajax请求去后台进行删除
              this.$http.delete("item/spec/group?gid="+group.id)
                .then(() => {
                  this.$message.success("删除成功");
                  this.loadData();
                })
                .catch(() => {
                  this.$message.error("删除失败");
                  this.loadData();
                })
            })
            .catch(() => {
              this.$message.info("删除已取消");
              this.loadData();
            })
        },

        // 新增分组方法
        addGroup(){
          this.show = true;
          this.isEdit = false;
          this.oldGroup = {
            cid:this.cid,
            name:""
          };
        },

        // 关闭弹窗
        closeWindow(){
          this.show = false;
          this.dialog = false;
          this.loadData();
        }
      }
    }
</script>

<style scoped>

</style>
