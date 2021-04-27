<template>
  <div style="padding: 20px">
    <el-form :model="addForm" :rules="rules" ref="addForm" :label-width="formLabelWidth">
      <el-form-item label="试卷名称" prop="title">
        <el-input v-model="addForm.title" style="width: 200px"></el-input>
      </el-form-item>
      <el-form-item label="是否公开" prop="status" :label-width="formLabelWidth">
        <el-radio-group v-model="addForm.status">
          <el-radio :label="1">是</el-radio>
          <el-radio :label="0">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="总分" prop="total_score" :label-width="formLabelWidth">
        <el-input-number v-model="addForm.total_score" @change="totalChange"></el-input-number>分
      </el-form-item>
      <el-form-item label="及格分" prop="pass_score" :label-width="formLabelWidth">
        <el-input-number v-model="addForm.pass_score" @change="passChange"></el-input-number>分
      </el-form-item>
      <el-form-item label="时间限制" prop="expire" :label-width="formLabelWidth">
        <el-input-number v-model="addForm.expire" @change="expireChange"></el-input-number>分钟
      </el-form-item>
      <el-form-item label="题目列表" prop="selectionListShow" :label-width="formLabelWidth">
        <h3>当前已有题目分数{{score}}</h3>
        <div class="title" 
             v-show="getShow"
             v-for="(item ,index) in addForm.selectionListShow" :key="index">
          <div class="title-num">
            <div>第{{index + 1}}题</div>
            <i class="el-icon-delete" @click="delOneQuestion(index)" />
          </div>
          <div class="title-con">
            <div class="title-con-left">
              <div class="con-type">{{getType(item.type)}}</div>
              <div class="con-question">
                <p>{{item.title}}</p>
                <ul v-if="item.value.value">
                  <li v-for="(item2, indey) in item.value.options" :key="indey">
                    <span :class="{'answer-red': indey === item.value.value[indey]}">
                      {{answer[indey]}}.{{item2}}</span>
                  </li>
                </ul>
              </div>
              <div class="answer">
                答案: <span v-show="item.value.value"
                            v-for="(item3, indez) in item.value.value" :key="indez + 'only'">
                        {{answer[item3]}}
                      </span>
                      <span v-show="!item.value.value"
                            v-for="(item4, indez) in item.value.options" :key="indez + 'once'">
                        {{item4}}
                      </span>
              </div>
            </div>
            <div class="title-con-ri">
              分值
              <el-input-number style="width: 150px" 
                               v-model="num" 
                               @change="handleChange" 
                               :min="1" :max="20"></el-input-number>
            </div>
          </div>
        </div>
        <el-button type="primary" @click="addTitle">添加题目</el-button>
      </el-form-item>
      <el-form-item style="margin-left: 380px">
        <el-button type="primary" @click="submitForm('addForm')">保存</el-button>
        <el-button @click="resetForm('addForm')">重置</el-button>
      </el-form-item>
    </el-form>
    <!-- 选择题目表格 -->
    <el-dialog title="选择题目" 
               :visible.sync="selectCrouse" 
               width="60%"
               @close="selectCrouse = false">
      <el-card class="box-card" shadow="never">
        <div slot="header" class="add-title">
          <div></div>
          <div class="right-con">
            <div></div>
            <div>
              <el-input class="search-title" v-model="meTitle" placeholder="标题"></el-input>
              <el-button class="btn-add" type="primary" @click="searchTitle">搜索</el-button>
            </div>
          </div>
        </div>
        <!-- 表格 -->
        <div class="table-container">
          <el-table
                  :data="questionList"
                  border
                  v-loading="listLoading"
                  align="center"
                  style="width: 100%"
                  height="380px"
                  @select="userSelect">
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>
            <el-table-column
                    prop="questionList.type"
                    align="center"
                    label="状态"
                    width="100">
                    <template slot-scope="scope">
                      <div>{{getType(scope.row.type)}}</div>
                    </template>
            </el-table-column>
            <el-table-column
                    prop="questionList.title"
                    label="题目">
                <template slot-scope="scope">{{scope.row.title}}</template>
            </el-table-column>
          </el-table>
          <div class="pagination-container">
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="pageNum"
                    :page-sizes="[5,10,15]"
                    :page-size="pageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="totals">
            </el-pagination>
          </div>

          <!-- 底部 -->
          <div slot="footer" class="dialog-footer">
            <div></div>
            <div>
              <el-button @click="selectCrouse = false">取 消</el-button>
              <el-button type="primary" @click="submitSelect">确 定</el-button>
            </div>
          </div>
        </div>
      </el-card>
    </el-dialog>
  </div>
</template>

<script>
// 选择题目接口
import { fetchQuestion } from '@/api/question.js'

export default {
  name: 'TestpaperForm',
  data() {
    return {
      questionList: [],  //题目表格列表
      selectionList: [],  //选中题目的数组
      answer: ['A', 'B', 'C'],
      num: 0,
      selectCrouse: false,
      listLoading: false,
      meTitle: '',  //搜索
      pageSize: 10,
      pageNum: 1,
      totals: 100,
      addForm: {
        title: '试卷标题',
        status: 1,
        total_score: 100,
        pass_score: 60,
        expire: 60,
        selectionListShow: [],  //用户展示选中题目的数组
      },
      score: 0,  //已有题目分数
      formLabelWidth: '120px',
      rules: {
        title: [
          { required: true, message: '请输入试卷名称', trigger: 'blur' },
        ],
        total_score: [
          { required: true, message: '请输入总分', trigger: 'blur' },
        ],
        pass_score: [
          { required: true, message: '请输入及格分', trigger: 'blur' },
        ],
        expire: [
          { required: true, message: '请输入时间限制', trigger: 'blur' },
        ]
      },
    }
  },
  computed: {
    getShow() {
      return this.addForm.selectionListShow.length > 0
    }
  },
  created() {
    this._getQuestionList(this.pageNum, this.pageSize)
  },
  mounted() {
    console.log(this.$route.params.data);
    if (this.$route.params.data) {
      this.addForm = this.$route.params.data
    }
  },
  methods: {
    // 请求数据
    _getQuestionList(page, limit) {
      fetchQuestion({page: page, limit: limit}).then(res => {
        this.questionList = res.data.items
      })
    },
    // 计数器
    totalChange(value) {
      console.log(value);
    },
    passChange(value) {
      console.log(value);
    },
    expireChange(value) {
      console.log(value);
    },
    // 添加题目
    addTitle() {
      this.selectCrouse = !this.selectCrouse
    },
    // 分页
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getQuestionList(this.pageNum, this.pageSize)
    },
    // 获取类型状态
    getType(type) {
      // 'radio' 单选题  'checkbox' 多选题 'trueOrfalse' 判断题 'answer' 问答题 'completion' 填空题

      if (type == 'radio') return '单选题'
      if (type == 'checkbox') return '多选题'
      if (type == 'trueOrfalse') return '判断题'
      if (type == 'answer') return '问答题'
      if (type == 'completion') return '填空题'
    },
    // 保存或重置
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.$router.go(-1)
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    // 选中的题目
    userSelect(seletions, row) {
      this.selectionList = seletions
    },
    // 点击提交展示数据
    submitSelect() {
      this.selectCrouse = false
      this.addForm.selectionListShow = this.selectionList
    },
    // 分值
    handleChange(value) {
      this.score = value
    },
    // 删除一个题目
    delOneQuestion(index) {
      // 直接删除中间存储的这个数组中的数据 this.addForm.selectionListShow 中的也会没有 为啥
      this.selectionList.splice(index, 1)
      this.addForm.selectionListShow.splice(index, 1)
    },
    // 搜索
    searchTitle() {
      const title = this.meTitle
      if (title === '') {
        this._getQuestionList(this.pageNum, this.pageSize)
        return
      }
      this.questionList =  this.questionList.filter(item => item.title === title)
    },
  }
}
</script>

<style scoped>
  /* 去li默认样式 */
  ul li{
    list-style: none;
  }
  .right-con{
    display: flex;
    justify-content: space-between;
  }
  .search-title{
    margin: 0 8px;
    width: 120px;
  }
  /* 选择题目底部确定与取消 */
  .dialog-footer{
    display: flex;
    justify-content: space-between;
  }

  /* 选择的题目的样式 */
  .title{
    width: 100%;
    border: 1px solid #eee;
    margin-bottom: 8px;
  }
  .title .title-num{
    height: 50px;
    color: #0f0;
    font-size: 18px;
    background-color: #e7faf0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 10px;
  }
  .title .title-con{
    margin-top: 8px;
    display: flex;
  }
  .title-con .title-con-left{
    flex: 1;
  }
  .title-con-left .con-type,.answer{
    height: 50px;
    line-height: 50px;
    color: #000;
    font-size: 18px;
    background-color: #f4f4f5;
    padding: 0 10px;
  }
  .answer-red{
    color: #f00;
  }
  .title-con .title-con-ri{
    width: 200px;
  }
</style>