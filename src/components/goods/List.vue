<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <el-table :data="goodsList" stripe border>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column
          prop="goods_price"
          label="商品价格（元）"
          width="120px"
        >
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="90px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="180px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.goods_id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>

    <el-dialog
      title="修改商品信息"
      :visible.sync="editDialogVisible"
      width="50%"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-form-item label="商品名称" prop="goods_name">
          <el-input v-model="editForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price">
          <el-input v-model="editForm.goods_price" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight">
          <el-input v-model="editForm.goods_weight" type="number"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number">
          <el-input v-model="editForm.goods_number" type="number"></el-input>
        </el-form-item>
      </el-form>
      <el-upload
        action="http://127.0.0.1:8888/api/private/v1/upload"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        list-type="picture"
        :headers="headerObj"
        :on-success="handleSuccess"
      >
        <el-button size="small" type="primary">点击上传</el-button>
        <div slot="tip" class="el-upload__tip">
          只能上传jpg/png文件，且不超过500kb
        </div>
      </el-upload>
      <!-- 富文本编辑组件 -->
      <quill-editor v-model="editForm.goods_introduce"></quill-editor>
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="edit">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      goodsList: [],
      total: 0,
      editDialogVisible: false,
      editForm: {},
      headerObj: {
        Authorization: window.sessionStorage.getItem("token"),
      },
      previewPath: "",
      previewVisible: false,
      editFormRules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "请输入商品重量", trigger: "blur" },
        ],
        goods_number: [
          { required: true, message: "请输入商品数量", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getGoodsList();
  },
  methods: {
    //  根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        this.$message.error("获取商品列表失败");
        return;
      }
      this.goodsList = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    async removeById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该商品，是否继续？",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (confirmResult !== "confirm") {
        this.$message.info("已取消本次操作");
        return;
      }
      const { data: res } = await this.$http.delete(`goods/${id}`);
      if (res.meta.status !== 200) {
        this.$message.error("删除商品失败！");
        return;
      }
      this.$message.success("删除商品成功！");
      this.getGoodsList();
    },
    goAddPage() {
      this.$router.push("/goods/add");
    },
    async showEditDialog(id) {
      this.editDialogVisible = true;
      const { data: res } = await this.$http.get(`goods/${id}`);
      if (res.meta.status !== 200) {
        this.$message.error("获取当前商品信息失败！");
        return;
      }
      this.editForm = res.data;
    },
    // 处理图片预览效果
    handlePreview(file) {
      this.previewPath = file.response.data.url;
      this.previewVisible = true;
    },
    // 处理移除图片的操作
    handleRemove(file) {
      // 获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path;
      // 从pics数组中，找到这个图片对应的索引值
      const i = this.editForm.pics.findIndex((x) => x.pic === filePath);
      // 调用数组的splice方法，把图片信息对象，从pics数组中移除
      this.editForm.pics.splice(i, 1);
    },
    handleSuccess(response) {
      // 先拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path };
      // 将图片信息对象push到pics数组中
      this.editForm.pics.push(picInfo);
    },
    async edit() {
      const { data: res } = await this.$http.put(`goods/${this.editForm.goods_id}`, this.editForm);
      if (res.meta.status !== 200) {
        this.$message.error("修改商品信息失败");
        return;
      }
      this.$message.success("修改商品信息成功！");
      this.getGoodsList();
      this.editDialogVisible = false
    },
  },
};
</script>

<style lang="less" scoped>
</style>