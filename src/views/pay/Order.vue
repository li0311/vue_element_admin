<template>
  <div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" type="primary" @click="derive">
          导出选中
        </el-button>
        <div class="right-con">
          <el-input class="search-title" v-model="meTitle" placeholder="请输入内容"></el-input>
          <el-button class="btn-add" @click="searchTitle">搜索</el-button>
        </div>
      </div>
      <div class="table-container">
        <el-table
                :data="orderList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
            type="selection"
            width="55">
          </el-table-column>
          <el-table-column
                  prop="orderList.id"
                  align="center"
                  label="订单号"
                  width="100"
                  fixed="left">
                  <template slot-scope="scope">{{scope.row.id}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="商品名称">
                  <template slot-scope="scope">{{scope.row.goods[0].title}}</template>
          </el-table-column>
          <el-table-column
                  prop="orderList.sub_count"
                  align="center"
                  label="订单类型"
                  width="100"
                  fixed="right">
                  <template slot-scope="scope">
                    <div v-show="scope.row.type == 'group'">
                      拼团
                    </div>
                    <div  v-show="scope.row.type == 'default'">
                      普通
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  prop="orderList.status"
                  align="center"
                  label="订单状态状态"
                  width="120"
                  fixed="right">
                  <template slot-scope="scope">
                    <div v-show="scope.row.status === 'success'" class="sold-out">
                      成功
                    </div>
                    <div  v-show="scope.row.status !== 'success'" class="sold-on">
                      待支付
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="原价/实付(元)">
                  <template slot-scope="scope">
                    {{scope.row.total_price}}/<span class="price-red">{{scope.row.price}}</span>
                    </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="支付方式">
                  <template slot-scope="scope">
                    {{scope.row.pay_method === 'wechat' ? '微信支付' : '支付宝支付'}}
                  </template>
          </el-table-column>
          <el-table-column
                  prop="orderList.created_time"
                  align="center"
                  label="创建/支付时间"
                  width="200"
                  fixed="right">
              <template slot-scope="scope">
                <div class="time-setting">
                  <p>{{scope.row.created_time}}</p>
                  <p>{{scope.row.updated_time}}</p>
                </div>
              </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="120"
                  fixed="right">
            <template slot-scope="scope">
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
  </div>
</template>

<script>
import { fetchOrder, deleteOrder } from '@/api/pay.js'

export default {
  name: 'Order',
  data() {
    return {
      meTitle: '', //输入框标题
      orderList: [],
      listLoading: false,
      totals: 100,
      updatePropsData: {},
      pageSize: 10,
      pageNum: 1
    }
  },
  created() {
    this._getOrderList(this.pageNum, this.pageSize)
  },
  methods: {
    /**
     * 请求订单数据
     */
    _getOrderList(page, limit) {
      fetchOrder({page: page, limit: limit}).then(res => {
        this.orderList = res.data.items
      })
    },
    // 搜索
    searchTitle() {
      const title = this.meTitle
      if (title === '') {
      this._getOrderList()
        return
      }
      this.orderList =  this.orderList.filter(item => item.goods[0].title === title)
    },
    // 分页
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this._getOrderList(this.pageNum, this.pageSize)
    },
    // 导出
    derive() {

    },
    // 删除
    handleDelete(_, row) {
        this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteOrder(row.id).then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.orderList.findIndex(item => item.id === row.id)
            this.orderList.splice(currentIndex, 1)
          }
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

  /* 价格红色数字 */
  .price-red{
    color: #f00;
  }

  /* 时间的样式 */
  .time-setting{
    display: flex;
    flex-direction: column;
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
</style>