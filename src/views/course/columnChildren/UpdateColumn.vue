<template>
  <div>
    <el-dialog class="add-content" 
               title="修改" 
               :visible.sync="updateVisible"
               fullscreen
               @close="$emit('changeUpdateType', false)">
      <el-form class="add-form" :model="updateForm">
        <el-form-item label="标题" :label-width="formLabelWidth">
          <el-input v-model="updateForm.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="封面" :label-width="formLabelWidth">
          <el-upload
            class="avatar-uploader"
            action="https://jsonplaceholder.typicode.com/posts/"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="updateForm.imageUrl" :src="updateForm.imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="课程介绍" style="margin-bottom: 30px;">
          <el-input type="textarea" v-model="updateForm.try" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="课程内容" style="margin-bottom: 30px;">
          <Tinymce ref="editor" v-model="updateForm.content" :height="400" />
        </el-form-item>
        <el-form-item label="课程价格" style="margin-bottom: 30px;">
          <el-input-number v-model="updateForm.price" @change="handleChange" :min="1" :max="100"></el-input-number>
        </el-form-item>
        <el-form-item label="状态" style="margin-bottom: 30px;">
          <el-radio-group v-model="updateForm.status">
            <el-radio :label="num1">下架</el-radio>
            <el-radio :label="num2">上架</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="更新状态" style="margin-bottom: 30px;">
          <el-radio-group v-model="updateForm.isend">
            <el-radio :label="isend1">连载中</el-radio>
            <el-radio :label="isend2">已完结</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancleAdd">取消</el-button>
        <el-button type="primary" @click="subAdd">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'

import { updateVideo } from '@/api/video.js'

export default {
  name: 'AddList',
  data() {
    return {
      updateForm: {
        title: '',
        imageUrl: '',
        content: '',
        try: '',
        price: '',
        status: 0,
        isend: 0
      },
      formLabelWidth: '120px',
      num1: 1,
      num2: 0,
      isend1: 1,
      isend2: 0,
      fileList: [{
        name: 'food.jpeg',
        url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
      }]
    }
  },
  props: {
    updateVisible: {
      type: Boolean,
      default: false
    },
    updateData: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  mounted() {
    console.log(this.updateData);
    this.updateForm = Object.assign({}, this.updateData)
  },
  components: {
    Tinymce
  },
  methods: {
    cancleAdd() {
      this.$emit('changeUpdateType', false)
    },
    subAdd() {
      updateVideo(this.updateForm).then(res => {
        this.$message.success('更新数据成功')
        this.$emit('editData', {type: false, data: this.updateForm})
      })
    },
    // 上传
    handleAvatarSuccess(res, file) {
      this.addForm.imageUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg';
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!');
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!');
      }
      return isJPG && isLt2M;
    },
    // 计数器
    handleChange(value) {
      this.updateForm.price = value
    },
    // 视频上传
    handleChangeVideo(file, fileList) {
      this.fileList = fileList.slice(-3);
    }
  }
}
</script>

<style scoped>
  .add-content{
    width: 100%;
    height: 100vh;
  }
  .add-form{
    width: 100%;
    height: 100%;
    background-color: #fff;
  }

  /* 图片上传 */
  .avatar-uploader {
    border: 1px dashed #d9d9d9;
    width: 178px;
    height: 178px;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>