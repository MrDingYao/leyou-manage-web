<template>
  <div>
    <!-- 整体是个卡片 -->
    <v-card>
      <!-- 卡片头部 -->
      <v-toolbar class="elevation-0">
        <!-- 新增按钮 -->
        <v-btn color="primary" @click="addGood">新增商品</v-btn>
        <v-spacer/>
        <!-- 状态 -->
        <v-flex xs3>
          状态：
          <v-btn-toggle v-model="toggle_exclusive">
            <v-btn flat>全部</v-btn>
            <v-btn flat @click="filter.saleable=true">上架</v-btn>
            <v-btn flat @click="filter.saleable=false">下架</v-btn>
          </v-btn-toggle>
        </v-flex>
        <!-- 搜索框 -->
        <v-flex xs3>
          <v-text-field label="搜索"
                        hide-details
                        append-icon="search"
                        v-model.lazy="filter.search"
                        clearable/>
        </v-flex>
      </v-toolbar>
      <v-divider/>
      <!-- 主题商品信息表格 -->
      <v-data-table
      :headers="headers"
      :items="goods"
      :pagination.sync="pagination"
      :total-items="totalGoods"
      :loading="loading"
      class="elevation-1"
      no-data-text="你想要的，并不存在"
      >
        <template slot="items" slot-scope="props">
          <td class="text-xs-center">{{props.item.id}}</td>
          <td class="text-xs-center">{{props.item.title}}</td>
          <td class="text-xs-center">{{props.item.cname}}</td>
          <td class="text-xs-center">{{props.item.bname}}</td>
          <td class="text-xs-center">
            <v-btn flat icon @click="editGood(props.item)">
              <v-icon>edit</v-icon>
            </v-btn>
            <v-btn flat icon @click="deleteGood(props.item.id)">
              <v-icon>delete</v-icon>
            </v-btn>
            <v-btn depressed flat small fab @click="shelfGoods(props.item.id)">
              {{filter.saleable ? "下架" : "上架"}}
            </v-btn>
          </td>
        </template>

      </v-data-table>
    </v-card>

    <!-- 新增商品的弹窗 -->
      <v-dialog v-model="show" max-width="800" persistent scrollable>
        <v-card>
          <v-toolbar dense dark color="primary">
            <v-toolbar-title>{{isEdit ? "编辑" : "新增"}}商品</v-toolbar-title>
            <v-spacer/>
            <v-btn icon @click="closeWindow">
              <v-icon>close</v-icon>
            </v-btn>
          </v-toolbar>

          <!-- 对话框正文 -->
          <v-card-text class="px-5">
            <!-- 自定义表单组件 -->
            <my-goods-form :is-edit="isEdit" :oldGoods="oldGoods" @close="closeWindow" :step="step" ref="goodsForm"/>
          </v-card-text>

          <!-- 底部按钮，操作步骤线 -->
          <v-card-actions>
            <v-flex class="xs3 mx-auto">
              <v-btn @click="previous" :disabled="step===1">上一步</v-btn>
              <v-btn @click="next" color="primary" :disabled="step===4">下一步</v-btn>
            </v-flex>
          </v-card-actions>
        </v-card>
      </v-dialog>
  </div>
    
</template>

<script>
    import myGoodsForm from "./MyGoodsForm"
    export default {
      name: "MyGoods",
      components: {myGoodsForm},
      data(){
        return{
          step:1,         // 步骤计数
          filter:{
            search:"",        // 搜索字段
            saleable:true     // 是否上架
          },
          headers:[
            {text:"id",align:"center",value:"id",sortable:false},
            {text:"标题",align:"center",value:"title",sortable:false},
            {text:"商品分类",align:"center",value:"cname",sortable:false},
            {text:"品牌",align:"center",value:"bname",sortable:false},
            {text:"操作",align:"center",value:"id",sortable:false},
          ],
          goods:[],      // 表格中显示的商品的信息数组
          totalGoods:0,  // 总共的信息条数
          loading:true,   // 是否是在加载
          pagination:{},  // 分页信息
          show:false,     // 添加商品的弹窗是否显示
          isEdit:false,   // 是否是编辑，用来区分是添加商品还是编辑商品
          oldGoods:{},     // 即将被编辑的商品
          toggle_exclusive:1,
          noDataText:"你想要的，并不存在"    // 没查询到数据时，表格显示的信息
        }
      },

      mounted(){
        this.getDataFromServer()
      },
      methods:{
        getDataFromServer(){
          this.$http.get("/item/spu/page",{
            params:{
              key:this.filter.search,
              saleable:this.filter.saleable,
              page:this.pagination.page,        // 当前页
              rows:this.pagination.rowsPerPage  // 每页信息数
            }
          }).then((resp) => {
              this.goods = resp.data.items;
              this.totalGoods = resp.data.total;
              this.loading = false;
            }).catch(() => {
              this.$message.error("服务器异常")
          })
        },

        // 关闭窗口的方法
        closeWindow(){
          this.show = false;
          this.dialog = false;
          this.isEdit=false;
          this.getDataFromServer();
          this.step = 1;
        },

        // 新增商品的方法
        addGood(){
          this.show = true;
          this.oldGoods = {};

        },

        // 编辑商品
        async editGood(oldGoods){
          // 异步请求，查询商品的详情和sku
          oldGoods.spuDetail = await this.$http.loadData("/item/spu/detail/" + oldGoods.id);
          oldGoods.skus = await this.$http.loadData("/item/sku/list?id=" + oldGoods.id);
          this.show = true;
          this.isEdit = true;
          this.oldGoods = oldGoods;
        },

        // 删除商品,暂未实现
        deleteGood(id){
          this.$message.confirm("此操作将永久删除改商品，是否继续？").then(() => {
            console.log(id);
            this.$message.success("删除成功");
          }).catch(() => {
            this.$message.info("删除操作已取消")
          })
        },

        // 下架和上架商品
        shelfGoods(id){
          const option = (this.filter.saleable ? "下架" : "上架");
          this.$message.confirm("是否确定" + option + "该商品？").then(() => {
            this.$http({
              method: "put",
              url: "/item/shelfGoods",
              data: this.$qs.stringify({id: id, flag: !this.filter.saleable})
            }).then(() => {
              this.$message.success("商品" + option + "成功");
              this.getDataFromServer();
            }).catch(() => {
              this.$message.error("服务器异常，商品" + option + "失败");
            });
          }).catch(() => {
            this.$message.info(option + "操作已取消");
          })
        },

        // 上一步按钮的方法
        previous(){
          if (this.step > 1) {
            this.step--;
          }
        },

        // 下一步的按钮的方法
        next(){
          if (this.step < 4 && this.$refs.goodsForm.$refs.basic.validate()) {
            this.step++;
          }
        }
      },

      // 监视pagination的属性变化
      watch:{
        pagination:{
          deep:true,
          handler(){
            this.getDataFromServer()
          }
        },
        // 监视搜索框
        filter:{
          deep:true,
          handler(){
            this.getDataFromServer()
          }
        }
      }

    }
</script>

<style scoped>

</style>
