<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" @click="addGroup">
          返回社区列表
        </el-button>
      </div>
      <div class="table-container">
        <el-table
                :data="groupList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  label="帖子内容">
                  <template slot-scope="scope">
                    <div class="content">
                      <div v-show="scope.row.is_top === 1" class="is-top">
                        <span class="is-top-con">置顶</span>
                      </div>
                      <div class="content-text">
                        <div v-for="(item, index) in scope.row.content" :key="index">
                          <p>{{item.text}}</p>
                          <div class="top-img">
                            <img :src="item.images[0]" width="150px" alt="">
                            <img :src="item.images[1]" width="150px" alt="">
                          </div>
                        </div>
                      </div>
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="用户"
                  width="100">
                  <template slot-scope="scope">{{scope.row.user.username}}</template>
          </el-table-column>
          <el-table-column
                  prop="groupList.created_time"
                  align="center"
                  label="创建时间"
                  width="160">
                  <template slot-scope="scope">{{scope.row.created_time}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="点赞数"
                  width="80">
                  <template slot-scope="scope">{{scope.row.support_count}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="评论数"
                  width="80">
                  <template slot-scope="scope">{{scope.row.comment_count}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="310"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" @click="lookReply(scope.$index,scope.row)">查看回复</el-button>
              <el-button type="success" 
                         v-show="scope.row.is_top === 1"
                         @click="scope.row.is_top = 0">
                置顶
              </el-button>
              <el-button v-show="scope.row.is_top !== 1"
                         @click="scope.row.is_top = 1">
                取消置顶
              </el-button>
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
    <!-- 弹出对话框 -->
    <el-dialog title="帖子回复" 
               :visible.sync="dialogFormVisible"
               @close="dialogFormVisible = false">
      <el-table
              :data="comment"
              border
              v-loading="listLoading2"
              align="center"
              style="width: 100%"
              height="380px">
        <el-table-column
                label="回复内容">
                <template slot-scope="scope">
                  <span class="con-blue">{{scope.row.reply_user.username | replyFilter}}</span>
                  <span>{{scope.row.content}}</span>
                </template>
        </el-table-column>
        <el-table-column
                align="center"
                label="用户"
                width="100">
                <template slot-scope="scope">{{scope.row.reply_user.username}}</template>
        </el-table-column>
        <el-table-column
                prop="comment.updated_time"
                align="center"
                label="回复时间"
                width="200">
                <template slot-scope="scope">{{scope.row.updated_time}}</template>
        </el-table-column>
        <el-table-column
                  align="center"
                  label="操作"
                  width="160"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="danger" @click="deleteComment(scope.$index,scope.row)">删除</el-button>
            </template>
          </el-table-column>
      </el-table>
      <!-- 评论的分页 -->
      <div class="pagination-container">
        <el-pagination
                @size-change="handleSizeChange2"
                @current-change="handleCurrentChange2"
                :current-page="userPageNum"
                :page-sizes="[5,10,15]"
                :page-size="userPageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="userTotals">
        </el-pagination>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// 接口
import { fetchPostList, deletePost, deletePostComment, fetchPostComment } from '@/api/bbs.js'

export default {
  name: 'Media',
  data() {
    return {
      groupList: [],
      comment: [],
      listLoading: false,
      listLoading2: false,
      dialogFormVisible: false,
      isDidEdit: false,  //修改还是新增
      totals: 100,
      pageSize: 10,
      pageNum: 1,
      userTotals: 100,  //评论 start
      userPageSize: 10,
      userPageNum: 1,  //评论 end
    }
  },
  created() {
    this._getPostList(this.pageNum, this.pageSize)
    this._getCommentList(this.userPageNum, this.userPageSize)
  },
  methods: {
    // 请求数据
    _getPostList(page, limit) {
      fetchPostList({page: page, limit: limit}).then(res => {
        this.groupList = res.data.items
      })
    },
    _getCommentList(page, limit) {
      fetchPostComment({page: page, limit: limit}).then(res => {
        this.comment = res.data.items
      })
    },
    // 普通事件方法
    // 返回社区列表
    addGroup() {
      this.$router.go(-1)
    },
    // 分页
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getPostList(this.pageNum, this.pageSize)
    },
    // 分页2
    handleSizeChange2(val) {
      this.userPageNum= val
    },
    handleCurrentChange2(val) {
      this.userPageSize = val
      this._getCommentList(this.userPageNum, this.userPageSize)
    },
    // 操作
    // 回复
    lookReply(_, row) {
      console.log(row);
      this.dialogFormVisible = true
    },
    // 删除
    handleDelete(_, row) {
      this.$confirm('是否要将选中的内容删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        deletePost().then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.groupList.findIndex(item => item.id === row.id)
            this.groupList.splice(currentIndex, 1)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });          
      })
    },
    // 删除一个评论
    deleteComment(index, row) {
      this.$confirm('是否要将选中的内容删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        deletePostComment().then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.comment.findIndex(item => item.id === row.id)
            this.comment.splice(currentIndex, 1)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });          
      })
    },

  },
  filters: {
    replyFilter(value) {
      return '@' + value
    }
  }
}
</script>

<style scoped>
  .btn-add{
    background-color: #1890ff;
    color: #fff;
    font-weight: bold;
  }

  /* 帖子内容样式 */
  .content{
    display: flex;
    flex-direction: column;
  }
  .is-top {
    padding-left: 10px;
  }
  .is-top .is-top-con{
    border: 1px solid #eee;
    padding: 8px;
    color: #f00;
  }
  .content-text{
    display: flex;
    flex-direction: column;
    padding: 8px;
  }
  .content-text p{
    margin: 10px 0;
    padding: 0;
    font-size: 17px;
  }
  .content-text .top-img img{
    width: 140px;
    margin-right: 8px;
  }

  /* 回复内容样式 */
  .con-blue{
    color: #00f;
  }

  /* 状态 */
  .text-ccc{
    padding: 5px;
    color: #ccc;
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

  /* 关联的样式 */
  .rele-con img {
    margin-top: 10px;
  }
  .rele-con span{
    color: #f00;
  }
</style>
