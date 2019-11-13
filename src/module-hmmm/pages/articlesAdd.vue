<template>
  <div style="margin:20px">
    <el-card style="padding:10px">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="标题:" prop="title">
          <el-input v-model="addForm.title"></el-input>
        </el-form-item>
        <el-form-item prop="articleBody">
          <quill-editor v-model="addForm.articleBody"></quill-editor>
        </el-form-item>
        <el-form-item label="视频地址:" style="margin-top:15px" prop="videoURL">
          <el-input v-model="addForm.videoURL"></el-input>
        </el-form-item>
        <div style="text-align:center">
          <el-button type="primary" @click="add" class="btnAdd">提交</el-button>
          <el-button @click="cancle" class="btnAdd">取消</el-button>
        </div>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import { add as articleAdd } from '@/api/hmmm/articles.js'
export default {
  data() {
    return {
      addForm: {
        title: '',
        articleBody: '',
        videoURL: ''
      },
      addFormRules: {
        title: [{ required: true, message: '请输入标题', trigger: 'blur' }],
        articleBody: [{ required: true, message: '请输入内容', trigger: 'blur' }],
        videoURL: [{ required: true, message: '请输入地址', trigger: 'blur' }]
      }
    }
  },
  methods: {
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await articleAdd(this.addForm)
        this.$router.push({ path: '/articles/list' })
      })
    },
    cancle() {
      this.$router.push({ path: '/articles/list' })
    }
  }
}
</script>

<style>
.ql-editor {
  min-height: 300px !important;
}
</style>
