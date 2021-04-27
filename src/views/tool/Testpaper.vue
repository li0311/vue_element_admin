<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button  type="primary" @click="testpaperForm">
          手动试卷
        </el-button>
        <el-button type="success" @click="imQuestion">
          考试管理
        </el-button>
        <div class="right-con">
          <el-input class="search-title" v-model="meTitle" placeholder="题目标题"></el-input>
          <el-button class="btn-add" @click="searchTitle">搜索</el-button>
        </div>
      </div>
      <div class="table-container">
        <el-table
                :data="testpaperList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  prop="testpaperList.id"
                  align="center"
                  label="ID"
                  sortable
                  width="80">
                  <template slot-scope="scope">{{scope.row.id}}</template>
          </el-table-column>
          <el-table-column
                  prop="testpaperList.title"
                  label="试卷标题">
              <template slot-scope="scope">{{scope.row.title}}</template>
          </el-table-column>
          <el-table-column
                  prop="testpaperList.status"
                  align="center"
                  label="是否公开"
                  width="80">
                  <template slot-scope="scope">
                    <div v-show="scope.row.status === 1" class="sold-out">
                      是
                    </div>
                    <div  v-show="scope.row.status !== 1" class="sold-on">
                      否
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  prop="testpaperList.total_score"
                  align="center"
                  label="总分"
                  width="100">
              <template slot-scope="scope">{{scope.row.total_score}}</template>
          </el-table-column>
          <el-table-column
                  prop="testpaperList.pass_score"
                  align="center"
                  label="及格分"
                  width="100">
              <template slot-scope="scope">{{scope.row.pass_score}}</template>
          </el-table-column>
          <el-table-column
                  prop="testpaperList.expire"
                  align="center"
                  label="时长（分钟）"
                  width="100">
              <template slot-scope="scope">{{scope.row.expire}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="240">
            <template slot-scope="scope">
              <el-button type="primary" @click="handleUpdate(scope.$index,scope.row)">编辑</el-button>
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
import { fetchTestpaper, deleteTestpaper } from '@/api/testpaper.js'

export default {
  name: 'Media',
  data() {
    return {
      meTitle: '', //输入框标题
      testpaperList: [],
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
    this._getTestpaperList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getTestpaperList(page, limit) {
      fetchTestpaper({page: page, limit: limit}).then(res => {
        this.testpaperList = res.data.items
      })
    },
    // 普通事件方法
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getTestpaperList(this.pageNum, this.pageSize)
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
      // this.updateType = !this.updateType
      // this.updatePropsData = row
      this.$router.push({
        path: '/tool/testpaper_form',
        name: 'TestpaperForm',
        params: {
          data: row
        }
      })
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
        deleteTestpaper(row.id).then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.testpaperList.findIndex(item => item.id === row.id)
            this.testpaperList.splice(currentIndex, 1)
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
        this._getTestpaperList(this.pageNum, this.pageSize)
        return
      }
      this.testpaperList =  this.testpaperList.filter(item => item.title === title)
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
