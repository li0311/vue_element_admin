<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" @click="addAcount">
          <svg-icon icon-class="edit" />
          新增收款账号
        </el-button>
      </div>
      <div class="table-container">
        <el-table
                :data="paymentList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  prop="paymentList.account"
                  align="center"
                  label="账号">
                  <template slot-scope="scope">{{scope.row.account}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="开户人">
                  <template slot-scope="scope">{{scope.row.name}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="开户银行">
                  <template slot-scope="scope">{{scope.row.bank}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="240"
                  fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" @click="handleUpdate(scope.$index,scope.row)">编辑</el-button>
              <el-button type="danger" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="pagination-container">
        <el-pagination
                :current-page="pageNum"
                :page-sizes="[1]"
                :page-size="pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="totals">
        </el-pagination>
      </div>
    </el-card>
    <!-- 弹出对话框 -->
    <el-dialog :title="isDidEdit ? '修改' : '新增'" :visible.sync="dialogFormVisible">
      <el-form :model="addForm" :rules="rules" ref="payment">
        <el-form-item label="账号" prop="account" :label-width="formLabelWidth">
          <el-input v-model="addForm.account"></el-input>
        </el-form-item>
        <el-form-item label="省市区" :label-width="formLabelWidth">
          <el-cascader :options="areaSelectData"
              @change="handleChange" class="full-width" size="large" 
              v-model="selectedOptions" placeholder="请选择您的地址" />
        </el-form-item>
        <el-form-item label="详细地址" prop="path" :label-width="formLabelWidth">
          <el-input v-model="addForm.path"></el-input>
        </el-form-item>
        <el-form-item label="所属银行" prop="bank" :label-width="formLabelWidth">
          <el-select v-model="addForm.bank" placeholder="请选择">
            <el-option v-for="(item, index) in banks" :key="index" 
                       :label="item" :value="item"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="姓名" prop="name" :label-width="formLabelWidth">
          <el-input v-model="addForm.name"></el-input>
        </el-form-item>
        <el-form-item style="margin-left: 420px">
          <el-button @click="resetForm">取消</el-button>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
        <!-- <el-form-item ></el-form-item> -->
      </el-form>
    </el-dialog>    
  </div>
</template>

<script>
// 接口 fetchPayments
import { fetchPayments, deleteOrder, createPayment } from '@/api/pay.js'

// 省市区三级选择器
import {regionData, CodeToText} from 'element-china-area-data';

export default {
  name: 'Media',
  data() {
    return {
      paymentList: [],
      addForm: {
        id: '',
        account: '',
        province: '',
        city: '',
        area: '',
        path: '',
        bank: '',
        name: ''
      },
      isDidEdit: false,
      listLoading: false,
      dialogFormVisible: false,
      formLabelWidth: '120px',
      areaSelectData: regionData,  //省市区数据
      selectedOptions: [],  //选择的内容
      totals: 1,
      pageSize: 1,
      pageNum: 1,
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
      banks: ['国家开发银行', '中国工商银行', '中国农业银行', '中国银行', '中国建设银行', '中国建设银行', '开封市商业银行']
    }
  },
  created() {
    this._getPaymentList(this.pageNum, this.pageSize)
  },
  methods: {
    // 请求数据
    _getPaymentList(page, limit) {
      fetchPayments({page: page, limit: limit}).then(res => {
        this.paymentList = res.data.items
      })
    },
    // 普通事件方法
    // 添加
    addAcount() {
      this.addForm = {}
      this.isDidEdit = false
      this.dialogFormVisible = !this.dialogFormVisible
    },
    // 省市区
    handleChange() {
      const provinceCode = this.selectedOptions[0];
      const cityCode = this.selectedOptions[1];
      const areaCode = this.selectedOptions[2];
      // CodeToText属性是区域码，属性值是汉字 CodeToText['110000']输出北京市
      this.addForm.province = CodeToText[provinceCode];
      this.addForm.city = CodeToText[cityCode];
      this.addForm.area = CodeToText[areaCode];
      console.log("选择的省市：", this.addForm.province, this.addForm.city, this.addForm.area);
    },
    // 表单提交
    submitForm() {
      this.$refs.payment.validate(async (valid) => {
        if (valid) {
          // 如果是编辑
          if (this.isDidEdit) {
            const id = this.addForm.id
            const currentIndex = this.paymentList.findIndex(item => item.id === id)
            this.paymentList[currentIndex] = this.addForm
            
            this.dialogFormVisible = false
            return
          }

          if (!this.isDidEdit) {
            console.log(22);
            // addForm2中没有数据 说明点击的是添加
            // 调接口增加数据
            await createPayment(this.addForm)
            // 获取id id为数组中最后一个元素的id加一 这样删除不会由bug
            const list = this.paymentList
            const currentId = list.lenght === 0 ? 0 : list[list.length -1].id
            const idForm = {...this.addForm, id: currentId + 1}
            this.paymentList = [...this.paymentList, idForm]
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

    /**
     * 操作
     */
    // 编辑
    handleUpdate(index, row) {
      console.log(index, row);
      this.dialogFormVisible = true
      // 是为了数据回显
      this.addForm = row
      // 是为了判断是编辑还是新增
      this.isDidEdit = true
      this.selectedOptions = [row.province, row.city, row.area]
      
    },
    // 删除
    handleDelete(index, row) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).then(() => {
        deleteOrder(row.id).then(res => {
          this.$message.success('删除成功')
          const currentIndex = this.paymentList.findIndex(item => item.id === row.id)
          this.paymentList.splice(currentIndex, 1)
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });          
      })
    },

  }
}
</script>

<style scoped>
  .btn-add{
    background-color: #1890ff;
    color: #fff;
    font-weight: bold;
  }
</style>
