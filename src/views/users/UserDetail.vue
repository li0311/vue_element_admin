<template>
  <el-dialog title="用户详情" 
             :visible.sync="dialogFormVisible"
             width="80%"
             @close="closeDia">
    <div class="detail-con" v-if="Object.keys(detailData).length != 0">
      <p>ID：{{detailData.id}}</p>
      <p>用户名：{{detailData.user.username}}</p>
      <p>昵称：{{detailData.user.nikename ? detailData.user.nikename : '未设置'}}</p>
      <p>手机号：{{detailData.user.phone}}</p>
      <p>锁定：{{detailData.user.status ? '是' : '否'}}</p>
      <p>会员：{{detailData.user.user_level}}</p>
      <p>会员到期事件：{{detailData.user.user_level_expire}}</p>
      <p>注册时间：{{detailData.user.created_time}}</p>
    </div>
    <el-menu
      :default-active="activeIndex2"
      class="el-menu-demo"
      mode="horizontal"
      @select="handleSelect"
      text-color="#000"
      active-text-color="#1890ff">
      <el-menu-item index="1">课程订阅</el-menu-item>
      <el-menu-item index="2">专栏订阅</el-menu-item>
      <el-menu-item index="3">订单记录</el-menu-item>
      <el-menu-item index="4">观看历史</el-menu-item>
      <el-menu-item index="5">用户评论</el-menu-item>
    </el-menu>
    <!-- 课程订阅 -->
    <el-table
            :data="crouseData"
            border
            v-loading="listLoading"
            align="center"
            style="width: 100%"
            max-height="250"
            v-show="showType === 1">
      <el-table-column
              align="center"
              label="课程标题"
              width="200">
              <template slot-scope="scope">{{scope.row.title}}</template>
      </el-table-column>
      <el-table-column
              prop="crouseData.price"
              align="center"
              label="购买价格"
              width="150">
              <template slot-scope="scope">{{scope.row.price}}</template>
      </el-table-column>
      <el-table-column
              prop="crouseData.created_time"
              align="center"
              label="购买时间">
          <template slot-scope="scope">{{scope.row.created_time}}</template>
      </el-table-column>
    </el-table>
    <!-- 专栏订阅 -->
    <el-table
            :data="columnData"
            border
            v-loading="listLoading"
            align="center"
            style="width: 100%"
            max-height="250"
            v-show="showType === 2">
      <el-table-column
              align="center"
              label="专栏标题"
              width="200">
              <template slot-scope="scope">{{scope.row.title}}</template>
      </el-table-column>
      <el-table-column
              prop="crouseData.price"
              align="center"
              label="购买价格"
              width="150">
              <template slot-scope="scope">{{scope.row.price}}</template>
      </el-table-column>
      <el-table-column
              prop="crouseData.created_time"
              align="center"
              label="购买时间">
          <template slot-scope="scope">{{scope.row.created_time}}</template>
      </el-table-column>
    </el-table>
    <!-- 订单 -->
    <el-table
            :data="orderData"
            border
            v-loading="listLoading"
            align="center"
            style="width: 100%"
            max-height="250"
            v-show="showType === 3">
      <el-table-column
                  prop="orderData.id"
                  align="center"
                  label="ID"
                  width="100">
                  <template slot-scope="scope">{{scope.row.id}}</template>
          </el-table-column>
      <el-table-column
              align="center"
              label="订单号"
              width="200">
              <template slot-scope="scope">{{scope.row.no}}</template>
      </el-table-column>
      <el-table-column
              prop="orderData.price"
              align="center"
              label="购买价格"
              width="150">
              <template slot-scope="scope">{{scope.row.price}}</template>
      </el-table-column>
      <el-table-column
              align="center"
              label="状态"
              width="150">
              <template slot-scope="scope">{{scope.row.status}}</template>
      </el-table-column>
      <el-table-column
              align="center"
              label="商品"
              width="150">
              <template slot-scope="scope">{{scope.row.title}}</template>
      </el-table-column>
      <el-table-column
              prop="orderData.created_time"
              align="center"
              label="购买时间">
          <template slot-scope="scope">{{scope.row.created_time}}</template>
      </el-table-column>
    </el-table>
    <!-- 历史 -->
    <el-table
            :data="historyData"
            border
            v-loading="listLoading"
            align="center"
            style="width: 100%"
            max-height="250"
            v-show="showType === 4">
      <el-table-column
              align="center"
              label="课程标题"
              width="200">
              <template slot-scope="scope">{{scope.row.title}}</template>
      </el-table-column>
      <el-table-column
              prop="historyData.price"
              align="center"
              label="课程类型"
              width="150">
              <template slot-scope="scope">
                <div v-show="scope.row.type == 'media'">图文</div>
                <div v-show="scope.row.type == 'audio'">音频</div>
                <div v-show="scope.row.type == 'video'">视频</div>
                <div v-show="scope.row.type == 'column'">专栏</div>
                </template>
      </el-table-column>
      <el-table-column
              prop="historyData.total_time"
              align="center"
              label="学习时长">
          <template slot-scope="scope">{{scope.row.total_time}}</template>
      </el-table-column>
    </el-table>
    <!-- 评论 -->
    <el-table
            :data="commentData"
            border
            v-loading="listLoading"
            align="center"
            style="width: 100%"
            max-height="250"
            v-show="showType === 5">
      <el-table-column
              align="center"
              label="评论内容"
              width="200">
              <template slot-scope="scope">{{scope.row.content}}</template>
      </el-table-column>
      <el-table-column
              prop="commentData.created_time"
              align="center"
              label="评论时间">
          <template slot-scope="scope">{{scope.row.created_time}}</template>
      </el-table-column>
      <el-table-column
              align="center"
              label="课程标题"
              width="200">
              <template slot-scope="scope">{{scope.row.title}}</template>
      </el-table-column>
      <el-table-column
              prop="commentData.price"
              align="center"
              label="课程类型"
              width="150">
              <template slot-scope="scope">
                <div v-show="scope.row.type == 'media'">图文</div>
                <div v-show="scope.row.type == 'audio'">音频</div>
                <div v-show="scope.row.type == 'video'">视频</div>
                <div v-show="scope.row.type == 'column'">专栏</div>
                </template>
      </el-table-column>
    </el-table>
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
  </el-dialog>
</template>

<script>
// 接口
import { 
  fetchUserCourse, 
  fetchUserColumn, 
  fetchUserOrder, 
  fetchUserHistory, 
  fetchUserComment
} from '@/api/user.js'

export default {
  name: 'UserDetail',
  data() {
    return {
      gridData: [],
      activeIndex2: '1',
      crouseData: [],  //课程
      columnData: [],  //专栏
      orderData: [],  //订单
      historyData: [],  //历史
      commentData: [],  //评论
      pageNum: 1,
      pageSize: 10,
      listLoading: false,
      totals: 100,
      showType: 1
    }
  },
  props: {
    dialogFormVisible: {
      type: Boolean,
      default: false
    },
    detailData: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  created() {
    this.getCrouse(this.pageNum, this.pageSize)
    this.getColumn(this.pageNum, this.pageSize)
    this.getOrder(this.pageNum, this.pageSize)
    this.getHistory(this.pageNum, this.pageSize)
    this.getComment(this.pageNum, this.pageSize)
  },
  methods: {
    /**
     * 请求表格数据
     */
    getCrouse(page, limit) {
      fetchUserCourse({page: page, limit: limit}).then(res => {
        this.crouseData = res.data.items
      })
    },
    getColumn(page, limit) {
      fetchUserColumn({page: page, limit: limit}).then(res => {
        this.columnData = res.data.items
      })
    },
    getOrder(page, limit) {
      fetchUserOrder({page: page, limit: limit}).then(res => {
        this.orderData = res.data.items
      })
    },
    getHistory(page, limit) {
      fetchUserHistory({page: page, limit: limit}).then(res => {
        this.historyData = res.data.items
      })
    },
    getComment(page, limit) {
      fetchUserComment({page: page, limit: limit}).then(res => {
        console.log(res);
        this.commentData = res.data.items
      })
    },

    /**
     * 普通事件方法
     */
    closeDia() {
      this.$emit('changeShowType', false)
    },
    handleSelect(key, keyPath) {
      this.showType = +key
    },
    // 分页
    handleSizeChange(val) {
      this.pageNum = val
    },
    handleCurrentChange(val) {
      this.pageNum = val
      this.getCrouse(this.pageNum, this.pageSize)
      this.getColumn(this.pageNum, this.pageSize)
      this.getOrder(this.pageNum, this.pageSize)
      this.getHistory(this.pageNum, this.pageSize)
      this.getComment(this.pageNum, this.pageSize)
    },
  }
}
</script>

<style scoped>
  .detail-con{
    display: flex;
    flex-wrap: wrap;
  }
  .detail-con p{
    width: 25%;
    /* text-align: center; */
  }

</style>