<template>
  <v-stepper v-model="step">
    <v-stepper-header>
      <v-stepper-step :complete="step > 1" step="1">基本信息</v-stepper-step>
      <v-divider/>
      <v-stepper-step :complete="step > 2" step="2">商品描述</v-stepper-step>
      <v-divider/>
      <v-stepper-step :complete="step > 3" step="3">规格参数</v-stepper-step>
      <v-divider/>
      <v-stepper-step step="4">SKU属性</v-stepper-step>
    </v-stepper-header>
    <v-stepper-items>
      <!--1、基本信息-->
      <v-stepper-content step="1">   
        <v-flex class="xs10 mx-auto">
          <v-form v-model="valid" ref="basic">
            <v-layout row>
              <v-flex xs5>
                <!--商品分类-->
                <v-cascader
                  url="/item/category/list"
                  required
                  showAllLevels
                  v-model="goods.categories"
                  label="请选择商品分类"/>
              </v-flex>
              <v-spacer/>
              <v-flex xs5>
                <!--品牌-->
                <v-select
                  :items="brandOptions"
                  item-text="name"
                  item-value="id"
                  label="所属品牌"
                  v-model="goods.brandId"
                  required
                  autocomplete
                  clearable
                  dense chips
                  :rules="[v => !!v || '品牌不能为空']"
                />
              </v-flex>
            </v-layout>
            <v-text-field label="商品标题" v-model="goods.title" :counter="200" required :rules="[v => !!v || '商品标题不能为空']"/>
            <v-text-field label="商品卖点" v-model="goods.subTitle" :counter="200"/>
            <v-text-field label="包装清单" v-model="goods.spuDetail.packingList" :counter="1000" multi-line :rows="3"/>
            <v-text-field label="售后服务" v-model="goods.spuDetail.afterService" :counter="1000" multi-line :rows="3"/>
          </v-form>       
        </v-flex>
      </v-stepper-content>
      <!--2、商品描述-->
      <v-stepper-content step="2">
        <v-editor v-model="goods.spuDetail.description" upload-url="/upload/image"/>
      </v-stepper-content>
      <!--3、规格参数-->
      <v-stepper-content step="3">
        <v-flex class="xs10 mx-auto px-3">
          <!--遍历整个规格参数，获取每一组-->
          <v-card v-for="spec in specifications" :key="spec.group" class="my-2">
            <!--组名称-->
            <v-card-title class="subheading">{{spec.group}}</v-card-title>
            <!--遍历组中的每个属性，并判断是否是全局属性，不是则不显示-->
            <v-card-text v-for="param in spec.params" :key="param.k" v-if="param.global" class="px-5">
              <!--判断是否有可选项，如果没有，则显示文本框。还要判断是否是数值类型，如果是把unit显示到后缀-->
              <v-text-field v-if="param.options.length <= 0" :label="param.k" v-model="param.v"
                            :suffix="param.unit || ''"/>
              <!--否则，显示下拉选项-->
              <v-select v-else :label="param.k" v-model="param.v" :items="param.options"/>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-stepper-content>
      <!--4、SKU属性-->
      <v-stepper-content step="4">
        <v-flex class="mx-auto">
          <!--遍历特有规格参数-->
          <v-card flat v-for="spec in specialSpecs" :key="spec.k">
            <!--特有参数的标题-->
            <v-card-title class="subheading">{{spec.k}}:</v-card-title>
            <!--特有参数的待选项，需要判断是否有options，如果没有，展示文本框，让用户自己输入-->
            <v-card-text v-if="spec.options.length <= 0" class="px-5">
              <div v-for="i in spec.selected.length+1" :key="i" class="layout row">
                <v-text-field :label="'新的' + spec.k + ':'" class="flex xs10" auto-grow
                              v-model="spec.selected[i-1]" v-bind:value="i"/>
                <v-spacer/>
                <v-btn @click="spec.selected.splice(i-1,1)" v-if="i <= spec.selected.length" icon>
                  <i class="el-icon-delete"/>
                </v-btn>
              </div>
            </v-card-text>
            <!--如果有options，需要展示成多个checkbox-->
            <v-card-text v-else class="container fluid grid-list-xs">
              <v-layout row wrap class="px-5">
                <v-checkbox color="primary" v-for="o in spec.options" :key="o" class="flex xs3"
                            :label="o" v-model="spec.selected" :value="o"/>
              </v-layout>
            </v-card-text>
          </v-card>
          <v-card>
            <!--标题-->
            <v-card-title class="subheading">SKU列表</v-card-title>
            <!--SKU表格，hide-actions因此分页等工具条-->
            <v-data-table :items="skus" :headers="headers" hide-actions item-key="indexes">
              <template slot="items" slot-scope="props">
                <tr @click="props.expanded = !props.expanded">
                  <!--价格和库存展示为文本框-->
                  <td v-for="(v,k) in props.item" :key="k" v-if="['price', 'stock'].includes(k)"
                      class="text-xs-center">
                    <v-text-field single-line v-model="props.item[k]" @click.stop=""/>
                  </td>
                  <!--enable展示为checkbox-->
                  <td class="text-xs-center" v-else-if="k === 'enable'">
                    <v-checkbox v-model="props.item[k]"/>
                  </td>
                  <!--indexes和images不展示，其它展示为普通文本-->
                  <td class="text-xs-center" v-else-if="!['indexes','images'].includes(k)">{{v}}</td>
                </tr>
              </template>
              <!--点击表格后展开-->
              <template slot="expand" slot-scope="props">
                <v-card class="elevation-2 flex xs11 mx-auto my-2">
                  <!--图片上传组件-->
                  <v-upload v-model="props.item.images" url="/upload/image"/>
                </v-card>
              </template>
            </v-data-table>
          </v-card>
        </v-flex>
        <!--提交按钮-->
        <v-flex xs3 offset-xs9>
          <v-btn color="info" @click="submit">保存商品信息</v-btn>
        </v-flex>
      </v-stepper-content>
    </v-stepper-items>
  </v-stepper>
</template>

<script>
export default {
  name: "goods-form",
  props: {
    oldGoods: {
      type: Object
    },
    isEdit: {
      type: Boolean,
      default: false
    },
    step: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      valid:false,
      goods: {
        categories: [], // 商品分类信息
        brandId: 0, // 品牌id信息
        title: "", // 标题
        subTitle: "", // 子标题
        spuDetail: {
          packingList: "", // 包装列表
          afterService: "", // 售后服务
          description: "" // 商品描述
        }
      },
      brandOptions: [], // 品牌列表
      specifications: [], // 规格参数的模板
      specialSpecs: [] // 特有规格参数模板
    };
  },
  methods: {
    submit() {
      // 表单校验。
      if(!this.$refs.basic.validate){
        this.$message.error("请先完成表单内容！");
      }
      // 先处理goods，用结构表达式接收,除了categories外，都接收到goodsParams中
      const {
        categories: [{ id: cid1 }, { id: cid2 }, { id: cid3 }],
        ...goodsParams
      } = this.goods;
      // 处理规格参数
      const specs = this.specifications.map(({ group, params }) => {
        const newParams = params.map(({ options, ...rest }) => {
          return rest;
        });
        return { group, params: newParams };
      });
      // 处理特有规格参数模板
      const specTemplate = {};
      this.specialSpecs.forEach(({ k, selected }) => {
        specTemplate[k] = selected;
      });
      // 处理sku
      const skus = this.skus
        .filter(s => s.enable)
        .map(({ price, stock, enable, images, indexes, ...rest }) => {
          // 标题，在spu的title基础上，拼接特有规格属性值
          const title = goodsParams.title + " " + Object.values(rest).join(" ");
          return {
            price: this.$format(price), // 价格需要格式化
            stock,
            enable,
            indexes,
            title, // 基本属性
            images: !images ? "" : images.join(","), // 图片
            ownSpec: JSON.stringify(rest) // 特有规格参数
          };
        });
      Object.assign(goodsParams, {
        cid1,
        cid2,
        cid3, // 商品分类
        skus // sku列表
      });
      goodsParams.spuDetail.specifications = JSON.stringify(specs);
      goodsParams.spuDetail.specTemplate = JSON.stringify(specTemplate);

      this.$http({
        method: this.isEdit ? "put" : "post",
        url: "/item/goods",
        data: goodsParams
      })
        .then(() => {
          // 成功，关闭窗口
          this.$emit("close");
          // 提示成功
          this.$message.success("保存成功了");
        })
        .catch(() => {
          this.$message.error("保存失败！");
        });
    }
  },
  watch: {
    oldGoods: {
      deep: true,
      handler(val) {
        if (!this.isEdit) {
          Object.assign(this.goods, {
            categories: null, // 商品分类信息
            brandId: 0, // 品牌id信息
            title: "", // 标题
            subTitle: "", // 子标题
            spuDetail: {
              packingList: "", // 包装列表
              afterService: "", // 售后服务
              description: "" // 商品描述
            }
          });
          this.specifications = [];
        } else {
          this.goods = Object.deepCopy(val);

          // 先得到分类名称
          const names = val.cname.split("/");
          // 组织商品分类数据
          this.goods.categories = [
            { id: val.cid1, name: names[0] },
            { id: val.cid2, name: names[1] },
            { id: val.cid3, name: names[2] }
          ];

          // 将skus处理成map
          const skuMap = new Map();
          val.skus.forEach(sku => {
            skuMap.set(sku.indexes, sku);
          });
          this.goods.skus = skuMap;
        }
      }
    },
    "goods.categories": {
      deep: true,
      handler(val) {
        // 判断商品分类是否存在，存在才查询
        if (val && val.length > 0) {
          // 根据分类查询品牌
          this.$http
            .get("/item/brand/cid/" + this.goods.categories[2].id)
            .then(({ data }) => {
              this.brandOptions = data;
            });
          // 根据分类查询规格参数
          this.$http
            .get("/item/spec/" + this.goods.categories[2].id)
            .then(({ data }) => {
              // 保存全部规格
              this.specifications = data;

              // 判断当前是否是编辑状态，如果是，开始对规格参数进行回显
              if (this.isEdit) {
                // 得到回显数据中的规格模板, 需要进行json反序列化
                const specs = JSON.parse(this.goods.spuDetail.specifications);
                // 遍历规格参数，并且回显
                this.specifications.forEach((spec, i) => {
                  spec.params.forEach((param, j) => {
                    param.v = specs[i].params[j].v;
                  });
                });
              }

              // 对特有规格进行筛选
              const temp = [];
              data.forEach(({ params }) => {
                params.forEach(({ k, options, global }) => {
                  if (!global) {
                    temp.push({
                      k,
                      options,
                      selected: []
                    });
                  }
                });
              });
              this.specialSpecs = temp;

              // 判断是否是编辑，如果是，对特有规格属性进行回显
              if (this.isEdit) {
                // 得到回显数据中的特有规格属性值:
                const template = JSON.parse(this.goods.spuDetail.specTemplate);
                // 遍历特有规格参数，回显数据
                this.specialSpecs.forEach(s => {
                  s.selected = template[s.k];
                });
              }
            });
        }
      }
    }
  },
  computed: {
    skus() {
      // 过滤掉用户没有填写数据的规格参数
      const arr = this.specialSpecs.filter(s => s.selected.length > 0);
      // 通过reduce进行累加笛卡尔积
      return arr.reduce(
        (last, spec, index) => {
          const result = [];
          last.forEach(o => {
            spec.selected.forEach((option, i) => {
              const obj = {};
              Object.assign(obj, o);
              obj[spec.k] = option;
              // 拼接当前这个特有属性的索引
              obj.indexes = (o.indexes || "") + "_" + i;
              if (index === arr.length - 1) {
                // 如果发现是最后一组，则添加价格、库存等字段
                Object.assign(obj, {
                  price: 0,
                  stock: 0,
                  enable: false,
                  images: []
                });
                // 去掉索引字符串开头的下划线
                obj.indexes = obj.indexes.substring(1);
                if (this.isEdit) {
                  // 如果是编辑，则回填sku信息
                  const sku = this.goods.skus.get(obj.indexes);
                  if (sku != null) {
                    const { price, stock, enable, images } = sku;
                    Object.assign(obj, {
                      price: this.$format(price),
                      stock,
                      enable,
                      images: images ? images.split(",") : []
                    });
                  }
                }
              }
              result.push(obj);
            });
          });
          return result;
        },
        [{}]
      );
    },
    headers() {
      if (this.skus.length <= 0) {
        return [];
      }
      const headers = [];
      Object.keys(this.skus[0]).forEach(k => {
        let value = k;
        if (k === "price") {
          // enable，表头要翻译成“价格”
          k = "价格";
        } else if (k === "stock") {
          // enable，表头要翻译成“库存”
          k = "库存";
        } else if (k === "enable") {
          // enable，表头要翻译成“是否启用”
          k = "是否启用";
        } else if (k === "indexes" || k === "images") {
          // 图片和索引不在表格中展示
          return;
        }
        headers.push({
          text: k,
          align: "center",
          sortable: false,
          value
        });
      });
      return headers;
    }
  }
};
</script>

<style scoped>
</style>
