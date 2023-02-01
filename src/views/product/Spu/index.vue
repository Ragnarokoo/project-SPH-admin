<template>
  <div>
    <el-card style="margin: 20px 0">
      <!-- 三级联动已经是全局组件了 -->
      <CategorySelect @getCategoryId="getCategoryId" :show="scene != 0" />
    </el-card>
    <el-card>
      <!-- 底部这里将来是有三部分进行切换 -->
      <div v-show="scene == 0">
        <!-- 展示SPU列表的结构 -->
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          @click="addSpu"
          >添加SPU</el-button
        >
        <el-table style="width: 100%" border :data="records">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="spuName" label="SPU名称" width="width">
          </el-table-column>
          <el-table-column prop="description" label="SPU描述" width="width">
          </el-table-column>
          <el-table-column prop="prop" label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <!-- 这里的按钮将来用hintButton替换 -->
              <hint-button
                title="添加sku"
                type="success"
                icon="el-icon-plus"
                size="mini"
                @click="addSku(row)"
              ></hint-button>
              <hint-button
                title="修改spu"
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                title="查看当前spu全部sku列表"
                type="info"
                icon="el-icon-info"
                size="mini"
                @click="handler(row)"
              ></hint-button>

              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  title="删除spu"
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  slot="reference"
                ></hint-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>

        <!--  分页器 -->
        <el-pagination
          style="text-align: center"
          :current-page="page"
          :page-sizes="[3, 5, 10]"
          :page-size="limit"
          layout="prev, pager, next, jumper, -> , sizes , total"
          :total="total"
          @current-change="getSpuList"
          @size-change="handleSizeChange"
        >
        </el-pagination>
      </div>
      <SpuForm v-show="scene == 1" @changeScene="changeScene" ref="spu" />
      <SkuForm v-show="scene == 2" ref="sku" @changeScenes="changeScenes" />
    </el-card>
    <el-dialog
      :title="`${spu.spuName}的sku列表`"
      :visible.sync="dialogTableVisible"
      :before-close="close"
    >
      <!-- table展示sku列表 -->
      <el-table :data="skuList" style="width: 100%" border v-loading="loading">
        <el-table-column prop="skuName" label="名称" width="width">
        </el-table-column>
        <el-table-column prop="price" label="价格" width="width">
        </el-table-column>
        <el-table-column prop="weight" label="重量" width="width">
        </el-table-column>
        <el-table-column label="默认图片" width="width">
          <template slot-scope="{ row, $index }">
            <img
              :src="row.skuDefaultImg"
              alt=""
              style="width: 100px; height: 100px"
            />
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import SpuForm from "./SpuForm";
import SkuForm from "./SkuForm";
export default {
  name: "Spu",
  data() {
    return {
      // 分别是分类的id
      category1Id: "",
      category2Id: "",
      category3Id: "",
      // 分页器当前第几页
      page: 1,
      // 每一页需要展示多少条数据
      limit: 3,
      // spu列表的数据
      records: [],
      // 分页器一共需要展示数据的条数
      total: 0,
      // 0:代表展示spi列表的数据  1：代表添加spu|修改spu  2：代表添加sku
      scene: 0,
      // 控制对话框的显示与隐藏
      dialogTableVisible: false,
      spu: {},
      // 存储的是SKU列表的数据
      skuList: [],
      loading: true,
    };
  },
  methods: {
    // 三级联动的自动以事件，可以把子组件的id传递给父组件
    getCategoryId({ categoryId, level }) {
      // categoryId：获取到一、二、三级分类的id，level：区分是几级的id
      if (level === 1) {
        this.category1Id = categoryId;
        // 清除2、3级分类的id
        this.category2Id = "";
        this.category3Id = "";
      } else if (level === 2) {
        this.category2Id = categoryId;
        // 清除3级分类的id
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        // 获取spu列表数据进行展示
        this.getSpuList();
      }
    },
    // 获取spu列表数据的方法
    async getSpuList(pages = 1) {
      this.page = pages;
      const { page, limit, category3Id } = this;
      // 携带三个参数：page 第几页，limit每一页显示多少条数据，三级分类的id
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      if (result.code === 200) {
        this.total = result.data.total;
        this.records = result.data.records;
      }
    },
    // 当分页器的某一个展示数据条数发生变化的回调
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit;
      // 再次发请求
      this.getSpuList();
    },
    // 添加spu按钮的回调
    addSpu() {
      this.scene = 1;
      // 通知子组件SpuForm发请求--两个
      this.$refs.spu.addSpuData(this.category3Id);
    },
    // 修改某一个SPU
    updateSpu(row) {
      this.scene = 1;
      // 获取子组件SpuForm子组件的
      // 在父组件当中可以通过$refs获取子组件等
      this.$refs.spu.initSpuData(row);
    },
    // 自定义事件回调（SpuForm）
    changeScene({ scene, flag }) {
      // flag 为了区分是保存还是添加
      // 切换结构（场景）
      this.scene = scene;
      // 子组件通知父组件切换场景，需要再次获取spu列表数据进行展示
      if (flag == "修改") {
        this.getSpuList(this.page);
      } else {
        this.getSpuList();
      }
    },
    // 删除spu按钮的回调
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      if (result.code == 200) {
        this.$message({
          type: "success",
          message: "删除成功",
        });
        // 代表spu个数大于1删除的时候停留在当前页，如果spu个数小于1则回到上一页
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1);
      }
    },
    // 添加sku按钮的回调
    addSku(row) {
      // 切换场景为2
      this.scene = 2;
      // 让父组件调用子组件的方法，让子组件发请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row);
    },
    // skuForm通知父组件修改场景
    changeScenes(scene) {
      this.scene = scene;
    },
    // 查看SKU的按钮的回调
    async handler(spu) {
      // 点击这个按钮的时候对话框可见
      this.dialogTableVisible = true;
      // 保存spu的信息
      this.spu = spu;
      // 获取sku列表的数据进行展示
      let result = await this.$API.spu.reqSkuList(spu.id);
      console.log(result);
      if (result.code == 200) {
        this.skuList = result.data;
        // loading隐藏
        this.loading = false;
      }
    },
    // 关闭对话框的回调
    close(done) {
      // loading属性再次变为真
      this.loading = true;
      // 清除sku列表的数据
      this.skuList = [];
      // 管理对话框
      done();
    },
    // 点击分页器第几页的按钮的回调
    /*     handleCurrentChange(page) {
      this.page = page;
      this.getSpuList();
    } */
  },
  components: {
    SpuForm,
    SkuForm,
  },
};
</script>

<style scoped></style>
