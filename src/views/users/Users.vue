<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-dropdown @command="handleCommand">
          <el-button class="select-type" type="danger">
            黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="1">禁止评论</el-dropdown-item>
            <el-dropdown-item command="2">禁止访问</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <div class="right-con">
          <el-input class="search-title" v-model="meTitle" placeholder="请输入内容"></el-input>
          <el-button class="btn-add" @click="searchTitle">搜索</el-button>
        </div>
      </div>
      <div class="table-container">
        <el-table
                :data="userList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%"
                @select="userSelect">
          <el-table-column
            type="selection"
            width="55">
          </el-table-column>
          <el-table-column
                  align="center"
                  label="用户">
                  <template slot-scope="scope">
                    <div class="content">
                      <img :src="scope.row.user.avatar" alt="">
                      <div class="content-text">
                        <p>{{scope.row.user.username}}</p>
                      </div>
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  prop="userList.total_consume"
                  align="center"
                  label="消费总额"
                  width="100"
                  fixed="right">
                  <template slot-scope="scope">{{scope.row.total_consume}}</template>
          </el-table-column>
          <el-table-column
                  prop="userList.updated_time"
                  align="center"
                  label="创建时间"
                  width="160"
                  fixed="right">
              <template slot-scope="scope">{{scope.row.updated_time}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="350"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" @click="isShowAdd(scope.$index,scope.row)">详情</el-button>
              <el-button class="mr-btn" type="success" @click="handleUpdate(scope.$index,scope.row)">联系用户</el-button>
              <el-dropdown @command="handleCommand2">
                <el-button class="select-type" type="danger">
                  黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="1">禁止评论</el-dropdown-item>
                  <el-dropdown-item command="2">禁止访问</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
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
    <user-detail :dialogFormVisible="addShowType" 
              @changeShowType="changeShowType"
              :detailData="detailData" />
  </div>
</template>

<script>
import UserDetail from './UserDetail.vue'

// 接口
import { fetchList, fetchUserDetail } from '@/api/user.js'

export default {
  name: 'Media',
  data() {
    return {
      meTitle: '', //输入框标题
      userList: [],
      listLoading: false,
      totals: 100,
      addShowType: false,
      updateType: false,
      updatePropsData: {},
      pageSize: 10,
      pageNum: 1,
      selectUser: [],
      detailData: {}
    }
  },
  components: {
    UserDetail
  },
  created() {
    this._getUserList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getUserList(page, limit) {
      fetchList({page: page, limit: limit}).then(res => {
        this.userList = res.data.items
      })
    },
    // 普通事件方法
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getUserList(this.pageNum, this.pageSize)
    },
    // 展示模态框
    isShowAdd(_, row) {
      this.addShowType = !this.addShowType
      const data = this.userList.find(item => item.id == row.id)
      this.detailData = data

      // 接口传id会响应数据
      fetchUserDetail({id: row.id}).then(res => {
        // console.log(res);
      })
    },
    changeShowType(type) {
      this.addShowType = type
    },
    /**
     * 操作
     */
    handleUpdate(index, row) {
      this.updateType = !this.updateType
      this.updatePropsData = row
    },
    changeUpdateType(type) {
      this.updateType = type
    },
    // 目录
    handleCatalog(index, row) {
      console.log(index, row);
      this.$router.push({
        path: '/course/column_detail',
        name: 'Catalogue',
        params: {
          id: row.id,
          data: row
        }
      })
    },
    // 设置单个用户黑名单
    handleCommand2(command) {
      // 禁止评论 1 禁止访问 2
      if (command == 1) {
        this.$confirm('是否要禁止用户评论?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 这里好像不能写代码 写了取消弹窗也会执行
          // const currentIndex = this.userList.findIndex(item => item.id === row.id)
          // this.userList[currentIndex].no_comment = 0
          this.$message({
            type: 'success',
            message: '操作成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });          
        });
        return
      }
      if (command == 2) {
        this.$confirm('是否要禁止用户访问?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // const currentIndex = this.userList.findIndex(item => item.id === row.id)
          // this.userList[currentIndex].no_access = 0
          this.$message({
            type: 'success',
            message: '操作成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });          
        });
      }
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
    // 选中一个用户
    userSelect(selection, row) {
      // console.log(row);
      // row为选中那一行的信息
      this.selectUser = selection
    },
    // 禁止评论
    handleCommand(command) {
      if (!this.selectUser.length) {
        this.$message.error('请先选中需要操作的用户')
        return
      }
      
      // 禁止评论 1 禁止访问 2
      if (command == 1) {
        this.$confirm('是否要禁止用户评论?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.selectUser.map(item => item.no_comment = 0)
          this.$message({
            type: 'success',
            message: '操作成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });          
        });
        return
      }
      if (command == 2) {
        this.$confirm('是否要禁止用户访问?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.selectUser.map(item => item.no_access = 0)
          this.$message({
            type: 'success',
            message: '操作成功!'
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消'
          });          
        });
      }

    }

  },
  filters: {
    priceFilter(value) {
      return '￥' + value
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

  /* 图文内容样式 */
  .content{
    display: flex;
    align-items: center;
  }
  .content img{
    width: 80px;
    border-radius: 50%;
    overflow: hidden;
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
  .sold-end{
    color: #666;
  }
  .loading{
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
    background-color: #f00;
    border-color: #ccc;
    color: #fff;
  }

  /* 联系用户与黑名单之间的间距 */
  .mr-btn{
    margin-right: 8px;
  }
</style>
