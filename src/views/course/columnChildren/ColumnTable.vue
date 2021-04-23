<template>
  <div class="table-content">
    <el-table
      ref="multipleTable"
      :data="tableData"
      tooltip-effect="dark"
      height="500"
      style="width: 100%"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
              align="center"
              label="专栏内容">
          <template slot-scope="scope">
            <div class="content">
              <img :src="scope.row.cover" width="150px" alt="">
              <div class="content-text">
                <p>{{scope.row.title}}</p>
                <span>{{scope.row.price | priceFilter}}</span>
              </div>
            </div>
          </template>
      </el-table-column>
    </el-table>
    <div class="pagination-container">
      <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="1"
              :page-sizes="[5,10,15]"
              :page-size="100"
              layout="total, sizes, prev, pager, next, jumper"
              :total="totals">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import { fetchList } from '@/api/column.js'

export default {
  name: 'ColumnTable',
  data() {
    return {
      tableData: [],
      totals: 10
    }
  },
  created() {
    this._getMediaList()
  },
  methods: {
    // 请求数据
    _getMediaList() {
      fetchList().then(res => {
        this.tableData = res.data.items
      })
    },
    // 普通事件方法
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this._getMediaList()
    },

    // 表格
    handleSelectionChange(val) {
      if (val.length >= 2) {
        this.$message.error('最多只能选中一个')
        return
      }
      const obj = {
        cover: val[0].cover,
        title: val[0].title,
        price: val[0].price
      }

      this.$emit('relevanceData', obj)
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
  .table-content{
    overflow: auto;
  }
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
</style>