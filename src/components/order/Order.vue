<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append"
                       icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <el-table :data="orderlist"
                border>
        <el-table-column type="index"
                         label="#"></el-table-column>
        <el-table-column label="订单编号"
                         prop="order_number"
                         width="200px"></el-table-column>
        <el-table-column label="订单价格"
                         prop="order_price"></el-table-column>
        <el-table-column label="是否付款">
          <template v-slot="scope">
            <el-tag v-if="scope.row.pay_status === '1'"
                    type="success">已付款</el-tag>
            <el-tag v-else
                    type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货"
                         prop="is_send"></el-table-column>
        <el-table-column label="下单时间"
                         prop="create_time">
          <template v-slot="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini">
            </el-button>
            <el-button type="success"
                       icon="el-icon-location"
                       size="mini"
                       @click="showProgress">
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[5, 10,15,20]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>

    <!-- 展示物流信息对话框 -->
    <el-dialog title="物流信息"
               :visible.sync="ProgressDialogVisible"
               width="50%">
      <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressInfo"
                          :key="index"
                          :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      ProgressDialogVisible: false,
      // 物流信息
      progressInfo: []
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败！')
      }
      console.log(res);
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    // 监听 pageSize
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // 监听 pagenum
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 展示物流信息 点击事件
    async showProgress () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')

      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败！')
      }
      this.progressInfo = res.data
      this.ProgressDialogVisible = true
      console.log(this.progressInfo);
    }
  }
}
</script>

<style lang="less" scoped>
</style>