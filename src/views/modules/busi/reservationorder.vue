<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('busi:reversationorder:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('busi:reversationorder:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <!--<el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>-->
      <!--<el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="ID">
      </el-table-column>-->
      <el-table-column
        prop="reservationNo"
        header-align="center"
        align="center"
        width="160"
        label="预约单号">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === '1'" size="small" type="warning">待审核</el-tag>
          <el-tag v-if="scope.row.status === '2'" size="small" type="danger">审核不通过</el-tag>
          <el-tag v-if="scope.row.status === '3'" size="small" type="info">审核通过</el-tag>
          <el-tag v-if="scope.row.status === '4'" size="small" type="success">已完成</el-tag>
          <el-tag v-if="scope.row.status === '5'" size="small" type="danger">已过期</el-tag>
          <!--<el-tag v-else size="small">正常</el-tag>-->
        </template>
      </el-table-column>
      <el-table-column
        prop="visitorName"
        header-align="center"
        align="center"
        label="访客姓名">
      </el-table-column>
      <el-table-column
        prop="visitorMobile"
        header-align="center"
        align="center"
        label="访客手机号">
      </el-table-column>
      <el-table-column
        prop="idcardNo"
        header-align="center"
        align="center"
        width="180"
        label="访客身份证">
      </el-table-column>
      <el-table-column
        prop="remark"
        header-align="center"
        align="center"
        label="来访事由">
      </el-table-column>
      <el-table-column
        prop="appointStartTime"
        header-align="center"
        align="center"
        label="预约开始时间">
      </el-table-column>
      <el-table-column
        prop="appointEndTime"
        header-align="center"
        align="center"
        label="预约结束时间">
      </el-table-column>
      <el-table-column
        prop="staffName"
        header-align="center"
        align="center"
        label="员工姓名">
      </el-table-column>
      <el-table-column
        prop="staffMobile"
        header-align="center"
        align="center"
        label="员工手机号">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="60"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/app/visitorreservation/searchReservationOrder'),
          method: 'post',
          data: this.$http.adornData({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'keyword': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 200) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/busi/reversationorder/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
