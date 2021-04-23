<template>
  <el-dialog title="选择课程" 
             :visible.sync="dialogFormVisible" 
             width="70%"
             @close="$emit('changeShowType', false)">
    <el-card class="box-card" shadow="never">
      <div slot="header" class="add-title">
        <div></div>
        <div class="right-con">
          <el-input class="search-title" v-model="meTitle" placeholder="标题"></el-input>
          <el-button class="btn-add" type="primary" @click="searchTitle">搜索</el-button>
        </div>
      </div>

      <!-- 主题内容 -->
      <div class="table-container">
        <template>
          <el-tabs :tab-position="tabPosition" style="height: 430px;">
            <el-tab-pane label="图文">
                <column-table @relevanceData="relevanceData"></column-table>
            </el-tab-pane>
            <el-tab-pane label="音频">
              <column-table></column-table>
              </el-tab-pane>
            <el-tab-pane label="视频">
              <column-table></column-table>
            </el-tab-pane>
          </el-tabs>
        </template>

        <!-- 底部 -->
        <div slot="footer" class="dialog-footer">
          <el-button @click="cancleAdd">取 消</el-button>
          <el-button type="primary" @click="subAdd">确 定</el-button>
        </div>
      </div>
    </el-card>
  </el-dialog>
</template>

<script>
import ColumnTable from './ColumnTable'

export default {
  name: 'AddCatalogue',
  data() {
    return {
      tabPosition: 'left',
      meTitle: '',
      data: {}
    }
  },
  props: {
    dialogFormVisible: {
      type: Boolean,
      default: false
    }
  },
  components: {
    ColumnTable
  },
  methods: {
    cancleAdd() {
      this.$emit('changeShowType', false)
    },
    subAdd() {
      this.$emit('changeShowType', false)
      this.$emit('relevanceData', this.data)
    },
    searchTitle() {

    },
    // 表格传递的数据
    relevanceData(data) {
      this.data = data
    }
  }
}
</script>

<style scoped>
  .ovflow-auto{
    overflow: auto;
  }
  .add-title{
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .right-con{
    display: flex;
    justify-content: center;
  }
  .search-title{
    margin: 0 8px;
    width: 120px;
  }
</style>