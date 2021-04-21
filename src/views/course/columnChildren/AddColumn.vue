<template>
  <div v-if="Object.keys(addForm).length !== 0">
    <el-dialog class="add-content" 
               title="新增" 
               :visible.sync="dialogFormVisible"
               fullscreen
               @close="$emit('changeShowType', false)">
      <el-form class="add-form" :model="addForm">
        <el-form-item label="标题" :label-width="formLabelWidth">
          <el-input v-model="addForm.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="封面" :label-width="formLabelWidth">
          <el-upload
            class="avatar-uploader"
            action="https://jsonplaceholder.typicode.com/posts/"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="addForm.imageUrl" :src="addForm.imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="课程介绍" style="margin-bottom: 30px;">
          <el-input type="textarea" v-model="addForm.try" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="课程内容" style="margin-bottom: 30px;">
          <Tinymce ref="editor" v-model="addForm.content" :height="400" />
        </el-form-item>
        <el-form-item label="课程价格" style="margin-bottom: 30px;">
          <el-input-number v-model="addForm.price" @change="handleChange" :min="1" :max="100"></el-input-number>
        </el-form-item>
        <el-form-item label="状态" style="margin-bottom: 30px;">
          <el-radio-group v-model="addForm.status">
            <el-radio label="下架"></el-radio>
            <el-radio label="上架"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="更新状态" style="margin-bottom: 30px;">
          <el-radio-group v-model="addForm.isend">
            <el-radio label="连载中"></el-radio>
            <el-radio label="已完结"></el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancleAdd">取 消</el-button>
        <el-button type="primary" @click="subAdd">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'

export default {
  name: 'AddList',
  data() {
    return {
      addForm: {
        title: '',
        imageUrl: '',
        content: '',
        try: '',
        price: 0,
        status: 0,
        isend: 0
      },
      formLabelWidth: '120px',
      num1: 1,
      num2: 0,
      fileList: [{
        name: 'food.jpeg',
        url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
      }]
    }
  },
  props: {
    dialogFormVisible: {
      type: Boolean,
      default: false
    }
  },
  components: {
    Tinymce
  },
  methods: {
    cancleAdd() {
      this.$emit('changeShowType', false)
    },
    subAdd() {
      this.$emit('changeShowType', false)
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
      this.addForm.price = value
    },
    // 上传文件
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
