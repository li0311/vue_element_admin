<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" @click="addBbs">
          <svg-icon icon-class="edit" />
          新增社区
        </el-button>
      </div>
      <div class="table-container">
        <el-table
                :data="bbsList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  label="社区">
                  <template slot-scope="scope">{{scope.row.title}}</template>
          </el-table-column>
          <el-table-column
                  prop="bbsList.status"
                  align="center"
                  label="状态"
                  width="100"
                  fixed="right">
                  <template slot-scope="scope">
                    <div v-show="scope.row.status === 1" class="sold-out">
                      隐藏
                    </div>
                    <div  v-show="scope.row.status !== 1" class="sold-on">
                      显示
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="管理员"
                  width="100">
                  <template>admin</template>
          </el-table-column>
          <el-table-column
                  prop="bbsList.created_time"
                  align="center"
                  label="创建时间"
                  width="200">
                  <template slot-scope="scope">{{scope.row.created_time}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="350"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" @click="handleBbs(scope.$index,scope.row)">管理</el-button>
              <el-button type="success" 
                         v-show="scope.row.status === 1"
                         @click="scope.row.status = 0">
                显示
              </el-button>
              <el-button type="danger"
                         v-show="scope.row.status !== 1"
                         @click="scope.row.status = 1">
                隐藏
              </el-button>
              <el-dropdown @command="handleCommand(scope.$index, scope.row, $event)">
                <el-button class="select-type" type="primary">
                  更多<i class="el-icon-arrow-down el-icon--right"></i>
                </el-button>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="bj">编辑</el-dropdown-item>
                  <el-dropdown-item command="xg">修改管理员</el-dropdown-item>
                  <el-dropdown-item command="sc">删除</el-dropdown-item>
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
    <!-- 增加编辑对话框 -->
    <el-dialog :title="isDidEdit ? '修改' : '新增'" :visible.sync="dialogFormVisible">
      <el-form :model="addForm" :rules="rules" ref="payment">
        <el-form-item label="社区标题" prop="title" :label-width="formLabelWidth">
           <el-input v-model="addForm.title"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="status" :label-width="formLabelWidth">
          <el-radio-group v-model="addForm.status">
            <el-radio :label="1">隐藏</el-radio>
            <el-radio :label="0">显示</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item style="margin-left: 380px">
          <el-button @click="resetForm">取消</el-button>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!-- 修改管理者对话框 -->
    <el-dialog title="选择课程" 
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
                  :data="users"
                  border
                  v-loading="listLoading2"
                  align="center"
                  style="width: 100%"
                  height="380px"
                  @select="userSelect">
            <el-table-column
              type="selection"
              width="55">
            </el-table-column>
            <el-table-column
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
          </el-table>

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

          <!-- 底部 -->
          <div slot="footer" class="dialog-footer">
            <div></div>
            <div>
              <el-button @click="selectCrouse = false">取 消</el-button>
              <el-button type="primary" @click="selectCrouse = false">确 定</el-button>
            </div>
          </div>
        </div>
      </el-card>
    </el-dialog>
  </div>
</template>

<script>
import AddCatalogue from '../course/columnChildren/AddCatalogue.vue'
// 接口
import { fetchList, deleteBbs, fetchPostComment } from '@/api/bbs.js'

export default {
  name: 'Media',
  data() {
    return {
      meTitle: '',
      bbsList: [],
      users: [],
      listLoading: false,
      listLoading2: false,
      dialogFormVisible: false,  //新增编辑
      selectCrouse: false,  //选择课程
      isChecked: false,
      isDidEdit: false,  //修改还是新增
      totals: 100,
      pageSize: 10,
      pageNum: 1,
      userTotals: 100,  //用户 start
      userPageSize: 10,
      userPageNum: 1,  //用户 end
      addForm: {
        id: 0,
        title: '',  //标题
        status: '',  //状态
        created_time: '2021-05-20 5:20:21'
      },
      formLabelWidth: '120px',
      rules: {
        title: [
          { required: true, message: '请输入标题', trigger: 'blur' },
        ]
      },
    }
  },
  components: {
    AddCatalogue
  },
  created() {
    this._getBbsList(this.pageNum, this.pageSize)
    this._getUsersList(this.userPageNum, this.userPageSize)
  },
  methods: {
    // 请求数据
    _getBbsList(page, limit) {
      fetchList({page: page, limit: limit}).then(res => {
        this.bbsList = res.data.items
      })
    },
    _getUsersList(page, limit) {
      fetchPostComment({page: page, limit: limit}).then(res => {
        this.users = res.data.items
      })
    },
    // 普通事件方法
    // 添加
    addBbs() {
      this.addForm = {}
      this.isDidEdit = false
      this.dialogFormVisible = !this.dialogFormVisible
    },
    // 分页
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getGroupList(this.pageNum, this.pageSize)
    },
    // 分页2
    handleSizeChange2(val) {
      this.userPageNum= val
    },
    handleCurrentChange2(val) {
      this.userPageSize = val
      this._getUsersList(this.userPageNum, this.userPageSize)
    },
    // 操作
    // 管理 跳转
    handleBbs(index, row) {
      this.$router.push({
        path: '/tool/bbs_post',
        query: {
          id: row.id
        }
      })
    },
    // 更多
    handleCommand(index, row, e) {
      console.log(index, row, e);

      // 编辑
      if (e=== 'bj') {
        this.dialogFormVisible = true
        // 是为了数据回显
        this.addForm = row
        // 是为了判断是编辑还是新增
        this.isDidEdit = true
      }

      // 修改
      if (e === 'xg') {
        this.selectCrouse = !this.selectCrouse
      }

      // 删除
      if (e === 'sc') {
        this.$confirm('是否要删除这个社区?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          deleteBbs().then(res => {
            if (res.code === 20000) {
              this.$message.success('下架成功')
              console.log(index, row);
              const currentIndex = this.bbsList.findIndex(item => item.id === row.id)
              this.bbsList.splice(currentIndex, 1)
            }
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消下架'
          });          
        })
      }
    },
    // 表单提交
    submitForm() {
      this.$refs.payment.validate(async (valid) => {
        if (valid) {
          console.log(this.addForm);
          // 如果是编辑
          if (this.isDidEdit) {
            const id = this.addForm.id
            const currentIndex = this.bbsList.findIndex(item => item.id === id)
            this.bbsList[currentIndex] = this.addForm
            
            this.dialogFormVisible = false
            this.$message.success('编辑成功')
            return
          }

          if (!this.isDidEdit) {
            // 调接口增加数据
            // await createGroup(this.addForm)
            // 获取id id为数组中最后一个元素的id加一 这样删除不会由bug
            const list = this.bbsList
            const currentId = list.lenght === 0 ? 0 : list[list.length -1].id
            const idForm = {...this.addForm, id: currentId + 1, created_time: '2021-05-20 5:20:21'}
            this.bbsList.unshift(idForm)
            this.$message.success('提交成功')
            this.dialogFormVisible = false
          }
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    resetForm() {
      this.$refs.payment.resetFields();
      this.dialogFormVisible = false
    },
    // 搜索
    searchTitle() {
      const title = this.meTitle
      if (title === '') {
        this._getUsersList(this.userPageNum, this.userPageSize)
        return
      }
      this.users =  this.users.filter(item => item.user.title === title)
    },
    // 弹出的课程选择了哪个用户
    userSelect(selection, row) {
      // row为选中那一行的信息
      console.log(row);
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
    display: flex;
    justify-content: space-between;
  }
  .search-title{
    margin: 0 8px;
    width: 120px;
  }

  .select-type{
    background-color: #fff;
    border-color: #ccc;
    color: #888;
  }

  /* 更多 */
  .select-type{
    background-color: #fff;
    border-color: #ccc;
    color: #888;
    margin-left: 8px;
  }

  /* 选择课程底部确定与取消 */
  .dialog-footer{
    display: flex;
    justify-content: space-between;
  }
</style>
