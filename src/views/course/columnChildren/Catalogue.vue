<template>
  <div>
    <div class="detail">
      <div class="detail-img">
        <img :src="titleList.cover" alt="">
      </div>
      <div class="detail-title">
        <div class="detail-status">
          <h2>{{titleList.title}}</h2>
          <span class="status-end" v-show="titleList.isend === 1">已完结</span>
          <span class="status-loading" v-show="titleList.isend !== 1">连载中</span>
        </div>
        <p>{{titleList.content}}</p>
        <span>￥{{titleList.price}}</span>
        <div class="detail-btn">
          <el-button type="warning" 
                     class="btn-on" 
                     v-show="titleList.isend === 1"
                     @click="titleList.isend = 0">上架</el-button>
          <el-button type="warning" 
                     class="btn-on" 
                     v-show="titleList.isend !== 1"
                     @click="titleList.isend = 1">下架</el-button>
          <el-button class="btn-loading" 
                     v-show="titleList.isend === 1"
                     @click="titleList.isend = 0">设为连载中</el-button>
          <el-button class="btn-loading" 
                     v-show="titleList.isend !== 1"
                     @click="titleList.isend = 1">设为已完结</el-button>
        </div>
      </div>
    </div>
    <el-card class="box-card" shadow="never">
      <div slot="header">
        <el-button class="btn-add" @click="isShowAdd">
          <svg-icon icon-class="edit" />
          新增目录
        </el-button>
        <div class="right-con">
          <el-dropdown>
            <el-button class="select-type" type="primary">
              商品状态<i class="el-icon-arrow-down el-icon--right"></i>
            </el-button>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item>已下架</el-dropdown-item>
              <el-dropdown-item>已上架</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <el-input class="search-title" v-model="meTitle" placeholder="请输入内容"></el-input>
          <el-button class="btn-add" @click="searchTitle">搜索</el-button>
        </div>
      </div>
      <div class="table-container">
        <el-table
                :data="mediaList"
                border
                v-loading="listLoading"
                align="center"
                style="width: 100%">
          <el-table-column
                  prop="mediaList.id"
                  align="center"
                  sortable
                  width="80"
                  fixed="left">
                  <template slot-scope="scope">{{scope.$index+1}}</template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="单品内容">
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
          <el-table-column
                  prop="mediaList.sub_count"
                  align="center"
                  label="订阅量"
                  width="100"
                  fixed="right">
                  <template slot-scope="scope">{{scope.row.sub_count}}</template>
          </el-table-column>
          <el-table-column
                  prop="mediaList.status"
                  align="center"
                  label="状态"
                  width="100"
                  fixed="right">
                  <template slot-scope="scope">
                    <div v-show="scope.row.status === 1" class="sold-out">
                      已下架
                    </div>
                    <div  v-show="scope.row.status !== 1" class="sold-on">
                      已上架
                    </div>
                  </template>
          </el-table-column>
          <el-table-column
                  align="center"
                  label="操作"
                  width="240"
                  fixed="right">
            <template slot-scope="scope">
              <el-button size="mini" type="primary" @click="handleUpdate(scope.$index,scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index,scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-card>
    <add-catalogue :dialogFormVisible="addShowType" 
                   @changeShowType="changeShowType" />
  </div>
</template>

<script>
import AddCatalogue from './AddCatalogue'
// fetchDetailCourse
import { fetchList, deleteMedia } from '@/api/media.js'

export default {
  name: 'Media',
  data() {
    return {
      meTitle: '', //输入框标题
      mediaList: [],
      titleList: {},
      listLoading: false,
      totals: 10,
      addShowType: false,
      updateType: false,
      updatePropsData: {}
    }
  },
  components: {
    AddCatalogue
  },
  created() {
    this._getMediaList()
  },
  mounted() {
    this.titleList = this.$route.params.data
  },
  methods: {
    // 请求数据
    _getMediaList() {
      fetchList().then(res => {
      this.mediaList = res.data.items
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
    // 展示模态框
    isShowAdd() {
      this.addShowType = !this.addShowType
    },
    changeShowType(type) {
      this.addShowType = type
    },
    // 操作
    handleUpdate(index, row) {
      this.updateType = !this.updateType
      this.updatePropsData = row
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
        deleteMedia(row.id).then(res => {
          if (res.code === 20000) {
            this.$message.success('删除成功')
            const currentIndex = this.mediaList.findIndex(item => item.id === row.id)
            this.mediaList.splice(currentIndex, 1)
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
        this._getMediaList()
        return
      }
      this.mediaList =  this.mediaList.filter(item => item.title === title)
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
/* 头部 */
  .detail{
    display: flex;
    width: 100%;
    align-items: center;
    padding: 0 10px;
  }
  .detail-title{
    margin-left: 16px;
  }
  .detail-status{
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .detail-status h2{
    font-size: 18px;
  }
  .detail-status .status-end{
    color: #bbb;
    font-size: 13px;
    margin-bottom: 10px;
  }
  .detail-status .status-loading{
    color: #f00;
    font-size: 13px;
    margin-bottom: 10px;
  }
  .detail-title p{
    color: #bbb;
    margin: 8px 0;
  }
  .detail-title span{
    color: #f00;
    font-size: 13px;
    margin-bottom: 10px;
  }
  .detail-btn .btn-on{
    margin: 0;
  }



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
    color: #f00;
  }
  .sold-on{
    border: 1px solid #eee;
    padding: 5px;
    color: #0f0;
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
