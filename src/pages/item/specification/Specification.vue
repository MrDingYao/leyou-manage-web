<!-- 规格页面的组件 -->
<template>
  <!-- 整个页面为一个card,分成两部分,左边为分类树,右边为具体的规格表格 -->
  <v-card>
    <v-layout row>
      <!-- 左边的分类树 -->
      <v-flex xs3>
        <v-card>
          <v-card-title>选择分类，查看规格参数模板：</v-card-title>
          <v-tree url="/item/category/list" :isEdit="false" @handleClick="handleClick"/>
        </v-card>
      </v-flex>

      <!-- 右边的参数规格表格 -->
      <v-flex xs9 class="px-1">
        <v-card>
          <v-card-title v-if="!currentNode.path">
            请选择一个分类
          </v-card-title>
          <!-- 导航面包屑 -->
          <v-breadcrumbs divider=">" dense v-else>
            <v-breadcrumbs-item v-for="(item,i) in currentNode.path" :key="i">
              {{item}}
            </v-breadcrumbs-item>
          </v-breadcrumbs>

          <v-divider/>

          <v-card-text>
            <!-- 主体内容,具体的规格内容展示,使用两个独立的自定义组件 -->
            <spec-group v-show="showGroup" :cid="currentNode.id" @select="selectGroup"/>
            <spec-param v-show="!showGroup" :group="group"/>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>

  </v-card>

</template>

<script>
  import SpecGroup from "./SpecGroup";
  import SpecParam from "./SpecParam";
    export default {
      name: "v-spec",
      components: {SpecParam, SpecGroup},
      data(){
        return{
          currentNode:{},   // 当前被选中的节点
          group:{},         // 被选中的分组
          showGroup:true,    // 是否展示分组

        }
      },
      methods:{
        // 点击分类后的调用方法
        handleClick(node){
          this.currentNode = Object.deepCopy(node);
          // 如果当前节点不是父节点,才会显示具体的规格表格
          if (!node.isParent) {
            // 显示规格表格
            this.showGroup = true;
            // 初始化表格内容,默认为空
            this.group = {};

          } else {
            // 否则是父节点的话，初始化数据
            //debugger;
            //this.group = {};
            this.showGroup = true;
          }
        },
        selectGroup(group){
          // 记录选中的分组
          this.group = group;
          // 不再显示分组表格
          this.showGroup = false;
          this.currentNode.path.push(group.name)
        }


      }
    }
</script>

<style scoped>

</style>
