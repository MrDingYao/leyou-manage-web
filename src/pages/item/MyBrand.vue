<template>
  <div>
    <!-- 使用卡片样式,将表格和添加按钮做在一个卡片上 -->
    <v-card>
      <!-- 卡片头部 -->
      <v-card-title>
        <v-btn color="primary" @click="addBrand">新增品牌</v-btn>
        <!-- 空间隔离组件 -->
        <v-spacer/>
        <!-- 搜索框,与search属性关联,hide-details取消输入框的默认提示 -->
        <v-text-field label="输入关键字搜索" v-model.lazy="search" append-icon="search" hide-details/>
      </v-card-title>
      <!-- 分割线 -->
      <v-divider/>

    <!-- 表格内容 -->
      <v-data-table
        :headers="headers"
        :items="brands"
        :pagination.sync="pagination"
        :total-items="totalBrands"
        :loading="loading"
        class="elevation-1"
        no-data-text="你想要的，并不存在"
      >
        <template slot="items" slot-scope="props">
          <td>{{ props.item.id }}</td>
          <td class="text-xs-center">{{ props.item.name }}</td>
          <td class="text-xs-center">
            <img :src="props.item.image" width="105px" height="35" v-show="props.item.image">
          </td>
          <td class="text-xs-center">{{ props.item.letter }}</td>
          <td class="justify-center">
            <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
            <v-btn color="warning" @click="deleteBrand(props.item)">删除</v-btn>
          </td>
        </template>
      </v-data-table>
    </v-card>

    <!-- 添加品牌的弹窗 -->
    <v-dialog v-model="show" max-width="500" persistent>
      <!-- 内部做成卡片 -->
      <v-card>
        <!-- 对话框的标题:dense:紧凑排布; dark:黑暗主题;  -->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? "编辑" : "新增"}}品牌</v-toolbar-title>
          <v-spacer/>
          <!-- 关闭窗口的按钮 -->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>

        <!-- 对话框的正文,表单 -->
        <v-card-text class="px-5">
          <!-- 自定义的表单局部组件,绑定一个close方法,子组件提交保存成功后,关闭窗口 -->
          <my-brand-form @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
  import MyBrandForm from "./MyBrandForm"
  export default {
    name:"my-brand",
    components:{
      MyBrandForm
    },

    data(){
      return{
        search: "",       // 搜索过滤字段
        totalBrands: 0,   // 总条数
        brands:[],        // 当前页品牌数据
        loading: true,    // 是否在加载中
        pagination:{},    // 分页信息
        headers:[         // 头信息
          {text:"id",align:"center",value:"id"},
          {text:"名称",align:"center",value:"name",sortable:false},
          {text:"LOGO",align:"center",value:"image",sortable:false},
          {text:"首字母",align:"center",value:"letter",sortable:true},
          {text:"操作",align:"center",value:"id",sortable:false}
        ],
        show:false,       // 用来控制添加品牌的弹出窗的显示
        oldBrand:{},       // 即将被编辑的品牌数据
        isEdit:false      // 是否是编辑,用来判断弹窗是用来编辑还是新增的
      }
    },
    methods:{
      getDataFromServer(){
        this.$http.get("/item/brand/page",{
          params:{
            key:this.search,    // 搜索条件
            page:this.pagination.page,   // 当前页码
            rows:this.pagination.rowsPerPage,    // 每页显示的信息数量
            sortBy:this.pagination.sortBy,       // 根据什么进行排序
            desc:this.pagination.descending      // 是否进行降序
          }
        })
          .then(({data}) => {       // 使用箭头函数
            // 将得到的数据赋值给本地属性
            this.brands = data.items;
            this.totalBrands = data.total;
            // 完成赋值后,将加载状态改为false
            this.loading = false;
          })

      },
      // 新增品牌方法
      addBrand(){
        this.show = true;
        // 将弹窗中的数据清空
        this.oldBrand = {
          name:"",        // 品牌的名字
          letter:"",      // 品牌的首字母
          image:"",       // 品牌的LOGO
          categories:[]   // 品牌所属的分类对象数组
        };
        this.isEdit = false;
      },

      // 编辑品牌方法
      editBrand(oldBrand){
        // 发送Ajax从后台获取分类信息,发送参数为当前brand的id
        this.$http.get("/item/category/bid/" + oldBrand.id)
        // 请求成功后的回调函数
          .then(({data}) => {
            // 修改标记
            this.isEdit = true;
            // 显示窗口
            this.show = true;
            // 获取要编辑的brand
            this.oldBrand = oldBrand;
            this.oldBrand.categories = data;
          });

      },

      // 删除品牌方法
      deleteBrand(oldBrand){
        // 弹出确认框
        this.$message.confirm("此操作将永久删除该品牌,是否继续?")
          .then(() => {
            // 发送异步请求去后台执行删除
            this.$http.delete("/item/brand?bid=" + oldBrand.id)
              .then(() => {
                // 删除成功,然后刷新页面
                this.$message.success("删除成功!");
                this.getDataFromServer()
              })
          })
          .catch(() => {
            this.$message.info("删除已取消!");
            this.getDataFromServer()
          })

      },

      // 关闭弹窗方法
      closeWindow(){
        this.show = false;
        this.dialog = false;
        // 优化一下,关闭窗口的同时刷新页面数据
        this.getDataFromServer();
      }

    },

    // vue实例挂载即会加载
    mounted(){
      this.getDataFromServer();
    },

    // 监视功能,当pagination发生改变的时候调用回调函数,进行数据的查询
    watch:{
      // 监视pagination的属性的变化
      pagination:{
        // deep为true时,会监视pagination中的属性及属性中的对象的属性的变化
        deep:true,
        // 变化后的回调函数,这里再次调用查询数据的getDataFromServer方法
        handler(){
          this.getDataFromServer()
        }
      },
      // 监视搜索字段
      search:{
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
