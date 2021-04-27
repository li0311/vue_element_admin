<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-select class="select-mr" v-model="answerValue" placeholder="答题状态">
          <el-option
            v-for="(item, index) in answerType" :key="index"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        <el-select class="select-mr" v-model="questionValue" placeholder="问卷状态">
          <el-option
            v-for="(item, index) in questionType" :key="index"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        <el-button type="primary" class="btn-add" @click="searchTitle">搜索</el-button>
      </div>
      <div class="table-container">
        <el-table
                :data="userTestList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column label="试卷名称">
              <template slot-scope="scope">{{scope.row.testpaper.title}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="用户名"
                  width="100">
              <template slot-scope="scope">{{scope.row.user.username}}</template>
          </el-table-column>
          <el-table-column
                  prop="userTestList.answer_status"
                  align="center"
                  label="答题状态"
                  width="100">
                  <template slot-scope="scope">
                    <div v-show="scope.row.answer_status === 1">
                      答题完成
                    </div>
                    <div  v-show="scope.row.answer_status !== 1">
                      答题未完成
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  prop="userTestList.read_status"
                  align="center"
                  label="是否阅卷"
                  width="80">
                  <template slot-scope="scope">
                    <div v-show="scope.row.answer_status === 1" class="sold-out">
                      是
                    </div>
                    <div  v-show="scope.row.answer_status === 0" class="sold-on">
                      否
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  prop="userTestList.created_time"
                  align="center"
                  label="开始时间"
                  width="180">
              <template slot-scope="scope">{{scope.row.created_time}}</template>
          </el-table-column>
          <el-table-column
                  prop="userTestList.score"
                  align="center"
                  label="分数"
                  width="100">
              <template slot-scope="scope">{{scope.row.score}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="220">
            <template slot-scope="scope">
              <el-button type="primary" @click="handleUpdate(scope.$index,scope.row)">阅卷</el-button>
              <el-button type="danger" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
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
    </el-card>
    <!-- 阅卷 -->
    <el-dialog title="阅卷" 
               :visible.sync="dialogFormVisible" 
               fullscreen
               v-if="Object.keys(readUser).length !== 0">
      <h3><span>试卷标题</span><span class="read-title">{{readUser.testpaper.title}}</span></h3>
      <h3>题目</h3>
      <div class="title"
            v-show="dialogFormVisible"
            v-for="(item ,index) in readUser.questions" :key="index">
        <div class="title-num">
          <div>第{{index + 1}}题</div>
          <span>本题分值：{{item.score}}</span>
        </div>
        <div class="title-con">
          <div class="title-con-left">
            <div class="con-type">{{getType(item.question.type)}}</div>
            <div class="con-question">
              <p>{{item.question.title}}</p>
              <ul v-if="item.question.value.value">
                <li v-for="(item2, indey) in item.question.value.options" :key="indey">
                  <span :class="{'answer-red': indey === item.question.value.value[indey]}">
                    {{answer[indey]}}.{{item2}}</span>
                </li>
              </ul>
            </div>
            <div class="answer">
              答案: <span v-show="item.question.value.value"
                          v-for="(item3, indez) in item.question.value.value" 
                          :key="indez + 'only'">
                      {{answer[item3]}}
                    </span>
                    <span v-show="!item.question.value.value"
                          v-for="(item4, indez) in item.question.value.options" 
                          :key="indez + 'once'">
                      {{item4}}
                    </span>
            </div>
          </div>
          <div class="title-con-ri">
            <div class="user-answer">
              <h3>用户答案</h3>
              <p>
                {{getAnswer(readUser.values[index].answer)}}  
              </p>
            </div>
            <div class="user-score">
              得分
              <el-input-number style="width: 150px" 
                              v-model="item.num" 
                              @change="handleChange" 
                              :min="1" :max="20"></el-input-number>
            </div>
          </div> 
        </div>
      </div>
      <div style="float: right; font-size: 18px">最终得分: <span style="color: #f00">{{score}}</span>分</div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// 接口
import { fetchUserTest, deleteUserTest, readUserTest } from '@/api/testpaper.js'

export default {
  name: 'Media',
  data() {
    return {
      userTestList: [],
      newUserTestList: [],
      readUser: {},  //阅卷对象
      answer: ['A', 'B', 'C'],
      score: 5,
      answerType: [{  //答题状态
        value: '1',
        label: '答题中'
      },{
        value: '2',
        label: '答题完成'
      }],  
      questionType: [{  // 问卷状态
          value: '1',
          label: '是'
        },{
          value: '2',
          label: '否'
        }],
      answerValue: '', 
      questionValue: '', 
      listLoading: false,
      totals: 100,
      pageSize: 10,
      pageNum: 1,
      dialogFormVisible: false,
    }
  },
  created() {
    this._getUserTestList(this.pageNum, this.pageSize)
    this._getReadUserList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getUserTestList(page, limit) {
      fetchUserTest({page: page, limit: limit}).then(res => {
        this.userTestList = res.data.items
        this.newUserTestList = res.data.items
      })
    },
    _getReadUserList(page, limit) {
      readUserTest({page: page, limit: limit}).then(res => {
        console.log(res);
        this.readUser = res.data
        this.readUser.questions.map(item => {
          this.$set(item, 'num', 0)
        })
      })
    },
    // 普通事件方法
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getUserTestList(this.pageNum, this.pageSize)
    },
    // 手动试卷
    testpaperForm() {
      this.$router.push('/tool/testpaper_form')
    },
    // 导入题目
    imQuestion() {
      this.$router.push('/tool/testpaper_test')
    },
    changeShowType(type) {
      this.addShowType = type
    },
    // 操作
    handleUpdate(index, row) {
      this.dialogFormVisible = !this.dialogFormVisible
      
    },
    changeUpdateType(type) {
      this.updateType = type
    },
    handleDelete(index, row) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        deleteUserTest(row.id).then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.userTestList.findIndex(item => item.id === row.id)
            this.userTestList.splice(currentIndex, 1)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });          
      })
    },
    // 搜索
    searchTitle() {
      // 1 答题中 2答题完成 
      // 1 是 2 否
      const answerVal = +this.answerValue
      const questionVal = +this.questionValue
      // 1 1
      if (answerVal === 1 && questionVal === 1) {
        this.$message.error('选择的状态不正确，答题未完成不能阅卷')
        return
      }
      // 1 2
      if (answerVal === 1 && questionVal === 2) {
        this.userTestList = []
        return
      }

      if (answerVal === 1 && questionVal === 0) {
        this.userTestList = this.newUserTestList.filter(item => item.answer_status === 0)
        return
      }
      if (answerVal === 2 && questionVal === 0) {
        console.log('jin');
        this.userTestList = this.newUserTestList.filter(item => item.answer_status === 1)
        return
      }

      if (questionVal === 1 && answerVal === 2) {
        this.userTestList = this.newUserTestList.filter(item => item.read_status === 0)
        return
      }
      if (questionVal === 2 && answerVal === 2) {
        this.userTestList = this.newUserTestList.filter(item => item.read_status === 1)
        return
      }
    },
    // 分值
    handleChange() {
      // 先重置分数 再累加
      this.score = 0
      this.readUser.questions.forEach(item => {
        this.score += item.num
      })
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
    // 获取用户答案
    getAnswer(answer) {
      if (Array.isArray(answer)) {
        // 元素是数字
        if (typeof answer[0] === 'number') {
          return answer.map(item => this.answer[item]).join('，')
        }

        // 元素是直接的答案
        if (typeof answer[0] === 'string') return answer[0]
      } else {
        return this.answer[answer]
      }
    },

  },
  filters: {
    priceFilter(value) {
      return '￥' + value
    }
  }
}
</script>

<style scoped>
  /* 下拉选择框的样式 */
  .select-mr{
    margin-right: 8px;
    width: 150px;
  }

  /* 阅卷的样式 */
  .read-title{
    color: #999;
    margin-left: 6px;
  }
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
  .title-con-left .con-question p{
    margin-left: 8px;
    font-size: 17px;
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
    width: 420px;
    display: flex;
  }
  .title-con-ri .user-answer{
    width: 200px;
    border: 1px solid #ccc;
    height: 100px;
    margin: 0 15px;
  }
  .title-con-ri .user-answer h3{
    width: 100%;
    height: 40px;
    padding: 8px;
    border-bottom: 1px solid #ccc;
  }
  .title-con-ri .user-answer p{
    height: 40px;
    padding: 8px;
  }
  /* .user-score 分值的类名*/
  /* 去li默认样式 */
  ul{
    padding: 0;
    margin: 0;
  }
  ul li{
    list-style: none;
    margin: 10px;
    font-size: 17px;
  }

  /* 图文内容样式 */
  .content{
    display: flex;
  }
  .content-text{
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 8px;
  }
  .content-text p{
    margin: 0;
    padding: 0;
    font-size: 18px;
  }
  .content-text span{
    color: #f00;
    text-align: left;
  }

  /* 状态 */
  .sold-out{
    border: 1px solid #eee;
    padding: 5px;
    color: #0f0;
  }
  .sold-on{
    border: 1px solid #eee;
    padding: 5px;
    color: #f00;
  }

  .right-con{
    float: right;
    display: flex;
  }
  .search-title{
    margin: 0 8px;
  }

  .select-type{
    background-color: #fff;
    border-color: #ccc;
    color: #888;
  }
</style>
