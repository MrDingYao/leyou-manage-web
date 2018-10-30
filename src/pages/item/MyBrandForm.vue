<template>
  <!-- 表单 -->
  <v-form v-model="valid" ref="myBrandForm">
    <v-text-field v-model="brand.name" label="请输入品牌名" required :rules="nameRules"/>
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules"/>
    <!-- 多级联动选择下拉框,选择所属分类 -->
    <v-cascader url="/item/category/list" multiple required v-model="brand.categories" label="请选择分类"/>
    <!-- 图片上传组件,显示在一行 -->
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px;color: #444">品牌LOGO:</span>
      </v-flex>
      <v-flex>
        <v-upload v-model="brand.image"
                  url="/upload/image"
                  :multiple="false"
                  :pic-width="250"
                  :pic-height="90"/>
      </v-flex>
    </v-layout>

    <!-- 提交和清空按钮 ,my-4:增大上下边距 -->
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  export default {
    name:"my-brand-form",
    // 子组件中通过props定义oldBrand属性
    props:{
      oldBrand:{
        type:Object
      },
      isEdit:{
        type:Boolean,
        default:false
      }
    },
    data(){
      return {
        valid:true,      // 表单校验结果的标记
        brand:{           // 品牌对象包含的初始信息
          name:"",        // 品牌的名字
          letter:"",      // 品牌的首字母
          image:"",       // 品牌的LOGO
          categories:[]   // 品牌所属的分类对象数组
        },
        // 表单校验规则
        // 校验规则是一个数组,里面是函数,
        nameRules:[
          v => !!v || "品牌名不能为空",
          v => v.length > 1 || "品牌名长度至少2位"
        ],
        letterRules:[
          v => !!v || "",
          v => /^[A-Z]{1}$/.test(v) || "品牌只能是A-Z的大写字母"
        ]

      }
    },
    methods:{
      submit(){
        // 提交,表单校验
        if (this.$refs.myBrandForm.validate()) {
          // 使用解构表达式获取brand对象中的属性
          const {categories,letter,...params} = this.brand;
          // 取出品牌对应的分类的数组中的所有id并转化为字符串
          params.cids = categories.map(c => c.id).join(",");
          // 将输入的首字母转为大写
          params.letter = letter.toUpperCase();
          // 发送post请求将数据发送至后台,进行保存
          //this.$http.post("/item/brand",this.$qs.stringify(params))
          this.$http({
            method:this.isEdit?"put":"post",    // 动态判断是编辑还是新增
            url:"/item/brand",
            data:this.$qs.stringify(params)
          })
            .then(() => {
              // 关闭窗口,通过$emit()方法来调用父组件的close方法
              this.$emit("close");
              this.$message.success("保存成功")}
              )
            .catch(() => {this.$message.error("保存失败")})
        }
      },
      clear(){
        // 清空
        this.$refs.myBrandForm.reset();
        this.categories = [];
        console.log(this.$qs)
      }
    },

    watch:{
      // 监控oldBrand的属性
      oldBrand:{
        handler(val){
          this.clear();
          if (val) {
            // 选择复制传递过来的brand,直接修改会影响父组件的值
            this.brand = Object.deepCopy(val);
          }else {
            // 为空,则初始化brand
            this.brand = {
              name:"",
              letter:"",
              image:"",
              categories:[]
            }
          }
        },
        deep:true
      }
    }

  }
</script>
