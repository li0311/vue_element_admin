<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" @click="addGroup">
          <svg-icon icon-class="edit" />
          创建优惠券
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
                  prop="groupList.id"
                  align="center"
                  label="创建优惠券ID"
                  width="80">
                  <template slot-scope="scope">{{scope.row.id}}</template>
          </el-table-column>
          <el-table-column
                  prop="groupList.price"
                  align="center"
                  label="面值"
                  width="100">
                  <template slot-scope="scope">{{scope.row.price}}</template>
          </el-table-column>
          <el-table-column
                  prop="groupList.price"
                  align="center"
                  label="适用课程">
                  <template slot-scope="scope">{{scope.row.value.title}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="使用期限"
                  width="200">
              <template slot-scope="scope">
                <div class="time-setting">
                  <p>{{scope.row.start_time}}至</p>
                  <p>{{scope.row.end_time}}</p>
                </div>
              </template>
          </el-table-column>
          <el-table-column
                  prop="groupList.c_num"
                  align="center"
                  label="发行量"
                  width="80">
                  <template slot-scope="scope">{{scope.row.c_num}}</template>
          </el-table-column>
          <el-table-column
                  prop="groupList.c_num"
                  align="center"
                  label="已领取"
                  width="80">
                  <template slot-scope="scope">{{scope.row.received_num}}</template>
          </el-table-column>
          <el-table-column
                  prop="groupList.used_num"
                  align="center"
                  label="已使用"
                  width="80">
                  <template slot-scope="scope">{{scope.row.used_num}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="拼团状态"
                  width="120">
                  <template slot-scope="scope">
                    <div class="text-ccc">
                      {{gropType(scope.row.start_time, scope.row.end_time)}}
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="240"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" @click="handleUpdate(scope.$index,scope.row)">编辑</el-button>
              <el-button type="danger" 
                         :disabled="gropType(scope.row.start_time, scope.row.end_time) == '已下架'" 
                         @click="handleDelete(scope.$index,scope.row)">下架</el-button>
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
    <el-dialog :title="isDidEdit ? '修改' : '新增'" :visible.sync="dialogFormVisible">
      <el-form :model="addForm" :rules="rules" ref="payment">
        <el-form-item label="类型" prop="type" :label-width="formLabelWidth">
          <el-select v-model="addForm.type" placeholder="请选择">
            <el-option label="课程" value="course"></el-option>
            <el-option label="专栏" value="column"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="关联课程" :label-width="formLabelWidth">
          <el-button type="primary" @click="relevance">关联</el-button>
          <div class="rele-con">
            <img :src="releData.cover" alt="">
            <p>{{releData.title}}</p>
            <span v-show="releData.price">{{releData.price | priceFilter}}</span>
          </div>
        </el-form-item>
        <el-form-item label="拼团价" :label-width="formLabelWidth">
          <el-input-number v-model="addForm.price" @change="handleChange1"></el-input-number>
        </el-form-item>
        <el-form-item label="拼团人数" :label-width="formLabelWidth">
          <el-input-number v-model="addForm.person" @change="handleChange2"></el-input-number>
        </el-form-item>
        <el-form-item label="是否开启拼团" prop="isGroup" :label-width="formLabelWidth">
          <el-radio-group v-model="addForm.isGroup">
            <el-radio label="1">是</el-radio>
            <el-radio label="2">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="拼团时限" prop="isGroup" :label-width="formLabelWidth">
          <el-radio-group v-model="addForm.isGroup">
            <el-radio :label="time1">24小时</el-radio>
            <el-radio :label="time2">48小时</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="拼团时限" prop="time" :label-width="formLabelWidth">
          <el-date-picker
            v-model="addForm.time"
            type="datetimerange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item style="margin-left: 380px">
          <el-button @click="resetForm">取消</el-button>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <add-catalogue :dialogFormVisible="addShowType" 
                   @changeShowType="changeShowType"
                   @relevanceData="getrelevanceData" />
  </div>
</template>

<script>
import AddCatalogue from '../course/columnChildren/AddCatalogue.vue'
// 接口
import { fetchCoupon, updateGroup, createGroup } from '@/api/marketing.js'

export default {
  name: 'Media',
  data() {
    return {
      groupList: [],
      listLoading: false,
      dialogFormVisible: false,
      isChecked: false,
      isDidEdit: false,  //修改还是新增
      addShowType: false,  //是否显示关联
      releData: {},  //显示的关联数据
      totals: 100,
      pageSize: 10,
      pageNum: 1,
      time1: 24,
      time2: 48,
      addForm: {
        type: '',
        price: '',
        person: '',
        isGroup: '',
        expire: '',
        time: []  //包括开始时间和结束时间
      },
      formLabelWidth: '120px',
      rules: {
        account: [
          { required: true, message: '请输入账号', trigger: 'blur' },
        ],
        path: [
          { required: true, message: '请输入详细地址', trigger: 'blur' },
        ],
        name: [
          { required: true, message: '开户人不能为空', trigger: 'blur' },
        ]
      },
    }
  },
  components: {
    AddCatalogue
  },
  created() {
    this._getCouponList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getCouponList(page, limit) {
      fetchCoupon({page: page, limit: limit}).then(res => {
        console.log(res);
        this.groupList = res.data.items
      })
    },
    // 普通事件方法
    // 获取拼团状态
    gropType(start_time, end_time) {
      // 0 未开始  1 已结束 2 已下架

      if (start_time == '2029-01-15 10:00:00') return '未开始'
      if (end_time == '2020-12-15 10:00:00') return '已结束'
      if (start_time == '2021-01-15 10:00:00' && end_time == '2031-01-15 10:00:00') return '已下架'
    },
    // 添加
    addGroup() {
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
      this._getCouponList(this.pageNum, this.pageSize)
    },
    // 操作
    // 编辑
    handleUpdate(index, row) {
      this.dialogFormVisible = true
      // 是为了数据回显
      this.addForm = row
      // 是为了判断是编辑还是新增
      this.isDidEdit = true

      // 回显关联数据
      this.releData = row.value
    },
    // 删除
    handleDelete(index, row) {
      this.$confirm('是否要将选中的课程下架?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        updateGroup().then(res => {
          if (res.code === 20000) {
            this.$message.success('下架成功')
            console.log(index, row);
            const currentIndex = this.groupList.findIndex(item => item.id === row.id)
            this.groupList[currentIndex].start_time = '2021-01-15 10:00:00'
            this.groupList[currentIndex].end_time = '2031-01-15 10:00:00'
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消下架'
        });          
      })
    },
    /**
     * 关联课程
     */
    relevance() {
      this.addShowType = !this.addShowType
    },
    // 改变显示模态框显示
    changeShowType(type) {
      this.addShowType = type
    },
    // 获取关联的数据
    getrelevanceData(data) {
      console.log(data);
      this.releData = data
    },
    // 计数器
    handleChange1(value) {
      console.log(value);
    },
    handleChange2(value) {
      console.log(value);
    },
    // 表单提交
    submitForm() {
      this.$refs.payment.validate(async (valid) => {
        if (valid) {
          console.log(this.addForm);
          // 如果是编辑
          if (this.isDidEdit) {
            const id = this.addForm.id
            const currentIndex = this.groupList.findIndex(item => item.id === id)
            this.groupList[currentIndex] = this.addForm
            this.groupList[currentIndex].value = this.releData
            
            this.dialogFormVisible = false
            this.$message.success('编辑成功')
            return
          }

          if (!this.isDidEdit) {
            // 调接口增加数据
            await createGroup(this.addForm)
            // 获取id id为数组中最后一个元素的id加一 这样删除不会由bug
            const list = this.groupList
            const currentId = list.lenght === 0 ? 0 : list[list.length -1].id
            const idForm = {...this.addForm, id: currentId + 1}
            this.groupList.push(idForm)
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
  }
  .content-text p:first-child{
    font-size: 18px;
  }
  .price-left{
    text-align: left;
  }
  .content-text span{
    color: #f00;
    text-align: left;
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
