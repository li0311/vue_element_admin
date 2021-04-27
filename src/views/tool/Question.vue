<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button  type="primary" @click="addQuestion">
          新增题目
        </el-button>
        <el-button type="success" @click="imQuestion">
          导入题目
        </el-button>
        <el-button type="danger" @click="delSome">
          批量删除
        </el-button>
        <div class="right-con">
            <el-select v-model="value" placeholder="题目类型">
              <el-option
                v-for="(item, index) in questionType" :key="index"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          <el-input class="search-title" v-model="meTitle" placeholder="题目标题"></el-input>
          <el-button class="btn-add" @click="searchTitle">搜索</el-button>
        </div>
      </div>
      <div class="table-container">
        <el-table
                :data="questionList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
            type="selection"
            width="55">
          </el-table-column>
          <el-table-column
                  prop="questionList.status"
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
          <el-table-column
                  align="center"
                  label="操作"
                  width="240">
            <template slot-scope="scope">
              <el-button size="mini" type="primary" @click="handleUpdate(scope.$index,scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
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
    <!-- <add-list :dialogFormVisible="addShowType" 
              @changeShowType="changeShowType" />
    <update-list v-if="updateType" 
                 :updateData="updatePropsData"
                 :updateVisible="updateType"
                 @changeUpdateType="changeUpdateType"
                 @editData="editData" /> -->
  </div>
</template>

<script>
// import AddList from './AddList.vue'
// import UpdateList from './UpdateList.vue'
// 接口
import { fetchQuestion, deleteQuestion } from '@/api/question.js'

// 'radio', 'checkbox', 'trueOrfalse', 'answer', 'completion'

export default {
  name: 'Media',
  data() {
    return {
      meTitle: '', //输入框标题
      questionList: [],
      questionType: [{  // 题目类型
          value: '1',
          label: '单选题'
        },{
          value: '2',
          label: '多选题'
        },{
          value: '3',
          label: '判断题'
        },{
          value: '4',
          label: '问答题'
        },{
          value: '5',
          label: '填空题'
        }],
      value: '',  
      listLoading: false,
      totals: 100,
      addShowType: false,
      updateType: false,
      updatePropsData: {},
      pageSize: 10,
      pageNum: 1
    }
  },
  components: {
    // AddList,
    // UpdateList
  },
  created() {
    this._getQuestionList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getQuestionList(page, limit) {
      fetchQuestion({page: page, limit: limit}).then(res => {
        console.log(res);
        this.questionList = res.data.items
      })
    },
    // 普通事件方法
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getQuestionList(this.pageNum, this.pageSize)
    },
    // 新增题目
    addQuestion() {
      
    },
    // 导入题目
    imQuestion() {

    },
    // 批量删除
    delSome() {

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
    changeShowType(type) {
      this.addShowType = type
    },
    // 操作
    handleUpdate(index, row) {
      this.updateType = !this.updateType
      this.updatePropsData = row
    },
    changeUpdateType(type) {
      this.updateType = type
    },
    // 更新
    editData(payload) {
      this.updateType = payload.type
      const list = payload.data
      const currentIndex = this.mediaList.findIndex(item => item.id === list.id)
      this.mediaList[currentIndex].title = list.title
      this.mediaList[currentIndex].price = list.price
      this.mediaList[currentIndex].status = list.status
      this.mediaList[currentIndex].content = list.content
      this.mediaList[currentIndex].try = list.try
    },
    handleDelete(index, row) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        deleteQuestion(row.id).then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.questionList.findIndex(item => item.id === row.id)
            this.questionList.splice(currentIndex, 1)
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
      const title = this.meTitle
      if (title === '') {
        this._getMediaList()
        return
      }
      this.mediaList =  this.mediaList.filter(item => item.title === title)
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
    color: #f00;
  }
  .sold-on{
    border: 1px solid #eee;
    padding: 5px;
    color: #0f0;
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
