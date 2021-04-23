<template>
  <div class="assets">
    <div class="table-con">
      <div class="table-con-item">
        <div class="balance-title">
          <span class="mr-btn">可用余额(元)</span>
          <el-button type="primary" @click="embody">申请体现</el-button>
        </div>
        <div class="table-con">20.00</div>
      </div>
      <div class="table-con-item">
        <div class="table-title">
          <span>累计收入(元)</span>
        </div>
        <div class="table-con">20.00</div>
      </div>
      <div class="table-con-item">
        <div class="table-title">
          <span>待结算金额(元)</span>
        </div>
        <div class="table-con">21</div>
      </div>
      <div class="table-con-item">
        <div class="table-title">
          <span>冻结余额(元)</span>
        </div>
        <div class="table-con">20.00</div>
      </div>
    </div>
    <div class="table-user">
      <el-table
                :data="assetsList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  prop="assetsList.created_time"
                  align="center"
                  label="交易时间">
              <template slot-scope="scope">
                <div class="time-setting">
                  <p>{{scope.row.created_time}}</p>
                </div>
              </template>
          </el-table-column>
          <el-table-column
                  prop="assetsList.account"
                  align="center"
                  label="提款账号">
                  <template slot-scope="scope">{{scope.row.account}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="开户人"
                  width="200">
                  <template slot-scope="scope">{{scope.row.name}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="体现金额"
                  width="150">
                  <template slot-scope="scope">{{scope.row.price | priceFilter}}</template>
          </el-table-column>
          <el-table-column
                  prop="assetsList.status"
                  align="center"
                  label="状态"
                  width="150">
                  <template slot-scope="scope">
                    <div v-show="scope.row.status === 'success'" class="sold-on">
                      提现成功
                    </div>
                    <div  v-show="scope.row.status !== 'success'" class="sold-out">
                      提现失败
                    </div>
                  </template>
          </el-table-column>
        </el-table>
    </div>
    <!-- 分页 -->
    <div class="pagination-container">
      <el-pagination
              :current-page="pageNum"
              :page-sizes="[1]"
              :page-size="pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="totals">
      </el-pagination>
    </div>
    <!-- 弹出对话框 -->
    <el-dialog title="体现" :visible.sync="dialogFormVisible">
      <el-form :model="assetsForm">
        <el-form-item label="提现金额" :label-width="formLabelWidth">
          <el-input-number v-model="assetsForm.price" @change="changePrice" :min="1" :max="10"></el-input-number>
        </el-form-item>
        <el-form-item label="提现账户" required :label-width="formLabelWidth">
          <el-select v-model="assetsForm.account" placeholder="请选择">
            <el-option label="请选择" :value="assetsForm.account">中国银行 {{assetsForm.account}}</el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { fetchAssets } from '@/api/pay.js'

export default {
  name: 'Assets',
  data() {
    return {
      assetsList: [],
      pageSize: 1,
      pageNum: 1,
      totals: 1,
      dialogFormVisible: false,
      listLoading: false,
      assetsForm: {
        price: 0,
        account: ''
      },
      formLabelWidth: '120px'
    }
  },
  created() {
    this._getAssetsList(this.pageNum, this.pageSize)
  },
  methods: {
    /**
     * 数据请求方法
     */
    _getAssetsList(page, limit) {
      fetchAssets({page: page, limit: limit}).then(res => {
        // 只给响应一条数据
        this.assetsList = res.data.items
        this.assetsForm.account = res.data.items[0].account
      })
    },

    // 普通事件方法
    // 申请体现
    embody() {
      this.dialogFormVisible = !this.dialogFormVisible
    },
    // 金额
    changePrice(value) {
      this.assetsForm.price = value
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
  /* 盒子 */
  .assets{
    padding: 20px;
  }
  
  /* 上边的四个表格 */
  .table-con{
    width: 100%;
    display: flex;
  }
  .table-con .table-con-item{
    flex: 1;
    margin: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  .table-con-item .balance-title{
    border-bottom: 1px solid #ccc;
    padding: 12px; 
  }
  .mr-btn{
    margin-right: 20px;
  }
  .table-con-item .table-title{
    border-bottom: 1px solid #ccc;
    padding: 20px;
  }
  .table-con-item .table-con{
    font-size: 25px;
    font-weight: bold;
    padding: 26px;
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
</style>