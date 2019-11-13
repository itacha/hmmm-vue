<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card>
        <el-button type="primary" style="margin-bottom:10px" @click="push">新增面试技巧</el-button>
        <!-- 搜索表单 -->
        <el-row>
          <el-form ref="form" :model="queryInfo" label-width="80px">
            <el-col :span="6">
              <el-form-item label="关键字">
                <el-input v-model="queryInfo.keyword" placeholder="请输入题目编号/题干"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6" :offset="12">
              <el-form-item>
                <el-button size="mini" @click="clear">清除</el-button>
                <el-button size="mini" type="primary" @click="getArticleList">搜索</el-button>
              </el-form-item>
            </el-col>
          </el-form>
        </el-row>
        <!-- 表格区域 -->
        <el-table :data="articleList" border style="width: 100%;margin-bottom:15px">
          <el-table-column type="index"> </el-table-column>
          <el-table-column prop="title" label="标题" width="180"> </el-table-column>
          <el-table-column prop="reads" label="阅读数" width="180"> </el-table-column>
          <el-table-column prop="state" label="状态"> </el-table-column>
          <el-table-column prop="creator" label="录用人"> </el-table-column>
          <el-table-column label="操作">
            <template v-slot="scope">
              <el-link type="primary" @click="showArticle(scope.row)">预览</el-link>
              <el-link type="primary" @click="showEditDialog(scope.row)">修改</el-link>
              <el-link type="primary" @click="delArticle(scope.row)">删除</el-link>
              <el-link type="primary" @click="changeState(scope.row)">{{ scope.row.state === 1 ? '禁用' : '启用' }}</el-link>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页区域 -->
        <el-pagination
          background
          layout="total,sizes,prev, pager, next,jumper"
          :total="total"
          :page-size="queryInfo.pagesize"
          :page-sizes="[1, 2, 5, 10]"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
        >
        </el-pagination>
      </el-card>
      <!-- 预览对话框 -->
      <el-dialog title="文章预览" :visible.sync="DialogVisible" width="50%" @close="DialogClose">
        <h3>{{ article.title }}</h3>
        <p>{{ article.articleBody }}</p>
      </el-dialog>
      <!-- 修改对话框 -->
      <el-dialog title="修改文章" :visible.sync="editDialogVisible" width="70%" @close="editDialogClose">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
          <el-form-item label="标题:" prop="title">
            <el-input v-model="editForm.title"></el-input>
          </el-form-item>
          <el-form-item prop="articleBody">
            <quill-editor v-model="editForm.articleBody"></quill-editor>
          </el-form-item>
          <el-form-item label="视频地址:" style="margin-top:15px" prop="videoURL">
            <el-input v-model="editForm.videoURL"></el-input>
          </el-form-item>
          <div style="text-align:center">
            <el-button type="primary" @click="editArticle" class="btnAdd">提交</el-button>
            <el-button @click="editDialogClose">取消</el-button>
          </div>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { list as articleList, remove, state, update } from '@/api/hmmm/articles.js'
export default {
  name: 'ArticlesList',
  data() {
    return {
      queryInfo: {
        keyword: '',
        pagesize: 10,
        pagenum: 1
      },
      DialogVisible: false,
      editDialogVisible: false,
      articleList: [],
      total: 0,
      article: [],
      editForm: {
        id: '',
        title: '',
        articleBody: '',
        videoURL: ''
      },
      editFormRules: {
        title: [{ required: true, message: '请输入标题', trigger: 'blur' }],
        articleBody: [{ required: true, message: '请输入内容', trigger: 'blur' }],
        videoURL: [{ required: true, message: '请输入地址', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getArticleList()
  },
  methods: {
    async getArticleList() {
      const { data: res } = await articleList(this.queryInfo)
      this.articleList = res.items
      this.total = res.counts
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getArticleList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getArticleList()
    },
    clear() {
      this.queryInfo.keyword = ''
      this.getArticleList()
    },
    push() {
      this.$router.push({ path: '/articles/add' })
    },
    async delArticle(data) {
      const confirmResult = await this.$confirm('此操作将永久删除该项，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult === 'cancel') {
         return this.$message({
          message: '已取消删除',
          type: 'info'
        })
      }
      const { data: res } = await remove(data)
      this.$message({
        message: '删除成功！',
        type: 'success'
      })
      this.getArticleList()
    },
    async changeState(data) {
      const { data: res } = await state(data)
      this.getArticleList()
    },
    showArticle(data) {
      this.article = data
      this.DialogVisible = true
    },
    DialogClose() {
      this.article = []
      this.DialogVisible = false
    },
    showEditDialog(data) {
      this.editForm = {
        title: data.title,
        articleBody: data.articleBody,
        videoURL: data.videoURL,
        id: data.id
      }
      console.log(this.editForm)

      this.editDialogVisible = true
    },
    editArticle() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await update(this.editForm)
        this.editDialogVisible = false
        this.getArticleList()
      })
    },
    editDialogClose() {
      this.editDialogVisible = false
    }
  }
}
</script>

<style>
.ql-editor {
  min-height: 200px !important;
}
</style>
