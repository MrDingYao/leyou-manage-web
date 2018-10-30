<!-- 参数规格编辑和新增的表单 -->
<template>
  <v-form v-model="valid" ref="specParamForm">
    <v-text-field v-model="param.name" label="请输入参数名" required :rules="[v => !!v || '参数名不能为空']"/>
    <v-switch v-model="param.numeric" :label="`${param.numeric ? '是' : '不是'}数值`"/>
    <v-text-field v-model="param.unit" label="请输入单位"/>
    <v-switch v-model="param.generic" :label="`${param.generic ? '' : '不'}通用`"/>
    <v-switch v-model="param.searching" :label="`${param.searching ? '可' : '不可'}搜索`"/>

    <div v-show="param.numeric && param.searching && param.generic">
    <!--特有参数的标题-->
    <div class="subheading">搜索范围:</div>
    <!--特有参数的待选项，需要判断是否有options，如果没有，展示文本框，让用户自己输入-->
    <v-card-text class="px-5">
      <div v-for="i in param.segments.length+1" :key="i" class="layout row px-5">
        <v-text-field placeholder="新的搜索范围" class="flex xs10" auto-grow
                      v-model="param.segments[i-1]" v-bind:value="i" single-line hide-details/>

        <v-btn @click="param.segments.splice(i-1,1)" v-if="i <= param.segments.length" icon>
          <i class="el-icon-delete"/>
        </v-btn>
      </div>
    </v-card-text>
    </div>

    <!-- 提交和清空按钮 ,my-4:增大上下边距 -->
    <v-layout class="my-4" row>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
    export default {
      name: "spec-param-form",
      props:{
        oldParam:{
          type:Object
        },
        isEdit:{
          type:Boolean,
          default:false
        },
        cid:{
          type:Number,
          default: 0
        },
        groupId:{
          type:Number,
          default:0
        }
      },

      data(){
        return{
          valid:true,
          param:{
            id:0,
            cid:this.cid,
            groupId:this.groupId,
            name:"",
            numeric:false,
            unit:"",
            generic:false,
            searching:false,
            segments:[]
          },
          show:false,
        }
      },
      watch:{
        oldParam:{
          deep:true,
          handler(val){
            if (val) {

              this.param = Object.deepCopy(val);

            } else {
              this.param = {
                cid:this.cid,
                groupId:this.groupId,
                name:"",
                numeric:false,
                unit:"",
                generic:false,
                searching:false,
                segments:[]
              }
            }
          }
        }
      },
      methods:{
        submit(){
          if (this.$refs.specParamForm.validate()) {
            // 将segments专程字符串格式
            const {segments,...newParam} = this.param;
            newParam.segments = segments.join(",");
            this.$http({
              method:this.isEdit ? 'put' : 'post',
              url:'/item/spec/param',
              data:this.$qs.stringify(newParam)
            }).then(() => {
              this.$message.success("保存成功");
              this.$emit("close");
            }).catch(() => {
              this.$message.error("保存失败");
              this.$emit("close");
            })
          }
        },
        clear(){
          this.$refs.specParamForm.reset();
          console.log(this.param)
        }
      }
    }
</script>

<style scoped>

</style>
