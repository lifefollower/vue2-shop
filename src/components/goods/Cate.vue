<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>

      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>

        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            type="success"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>

        <template slot="opt" slot-scope="scope">
          <el-button
            icon="el-icon-edit"
            type="primary"
            size="mini"
            @click="showCateDialog(scope.row)"
            >编辑</el-button
          >
          <el-button
            icon="el-icon-delete"
            type="danger"
            size="mini"
            @click="removeCateById(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        ref="addCateFormRef"
        :model="addCateForm"
        :rules="addCateFormRules"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ expandTrigger: 'hover', ...cascaderProps }"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类信息 -->
    <el-dialog
      title="修改分类"
      :visible.sync="editDialogVisible"
      width="50%"
    >
      <el-form
        :model="editCate"
        :rules="editCateRules"
        ref="editCateRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCate.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisbel = false">取 消</el-button>
        <el-button type="primary" @click="editCateInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      catelist: [],
      total: 0,
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          type: "template",
          template: "isok",
        },
        {
          label: "排序",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "opt",
        },
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: "",
        cat_pid: 0,
        cat_level: 0,
      },
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: "请输入分类名称",
            trigger: "blur",
          },
        ],
      },
      parentCateList: [],
      cascaderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      selectedKeys:[],
      editDialogVisible:false,
       editCate: {},
      editCateRules: {
        cat_name: [{ required: true, message: '请输入要修改的信息', trigger: 'blur' }]
      },
      editCateId:''
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败!");
      }
      this.catelist = res.data.result;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类数据失败!");
      }
      this.parentCateList = res.data;
    },
    parentCateChanged(){
        if(this.selectedKeys.length>0){
            this.addCateForm.cat_pid=this.selectedKeys[this.selectedKeys.length-1]
            this.addCateForm.cat_level=this.selectedKeys.length
            return
        }else{
             this.addCateForm.cat_pid=0
            this.addCateForm.cat_level=0
        }
    },
    addCateDialogClosed(){
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys=[]
        this.addCateForm.cat_level=0
        this.addCateForm.cat_pid=0
    },
    addCate(){
        this.$refs.addCateFormRef.validate(async valid=>{
            if(!valid)return
            const {data:res}=await this.$http.post('categories',this.addCateForm)
            if(res.meta.status!==201){
                return this.$message.error('添加分类失败!')
            }
            this.$message.success('添加分类成功!')
            this.getCateList()
            this.addCateDialogVisible=false
        })
    },
    async showCateDialog(cateInfo) {
        this.editCateId=cateInfo.cat_id
      const { data: res } = await this.$http.get(
        'categories/'+cateInfo.cat_id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询分类失败！");
      }
      this.editCate = res.data;
      this.editDialogVisible = true;
    },
    async editCateInfo(){
        const { data: res } = await this.$http.put('categories/' + this.editCate.cat_id, { cat_name: this.editCate.cat_name })
      if (res.meta.status !== 200) {
        return this.$message.error('更新分类数据失败!')
      }
      this.$message.success('更新分类数据成功!')
      this.getCateList()
      this.editDialogVisbel = false
    },
     async removeCateById(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该分类, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      const { data: res } = await this.$http.delete(`categories/${id}`);
      if (res.meta.status !== 200) return this.$message.error("删除分类失败！");
      this.$message.success("删除分类成功！");
      this.getCateList()
    },
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
</style>