<template>
  <section>
    <div v-title :data-title="this.$route.name"></div>
    <el-row class="toolbar">
      <el-form :inline="true" :model="filter">
        <el-form-item label="商品编号">
          <el-input v-model="filter.code" placeholder="输入商品编号"></el-input>
        </el-form-item>
        <el-form-item label="商品名称">
          <el-input v-model="filter.name" placeholder="输入商品名称"></el-input>
        </el-form-item>
        <el-form-item label="">
          <el-button type="primary" @click="getWareList">搜索</el-button>
        </el-form-item>
      </el-form>
    </el-row>
    <el-table :data="wareList" v-loading="loading" border style="width: 100%">
      <el-table-column type="index" width="60"></el-table-column>
      <el-table-column prop="wareCode" label="商品编号" sortable width="200"></el-table-column>
      <el-table-column prop="wareName" label="商品名称" width="200"></el-table-column>
      <!-- <el-table-column prop="wareKind" label="商品类别" width="150"></el-table-column> -->
      <el-table-column prop="verifyStatus" label="审核状态" width="120" :formatter="formatStatus" >
      </el-table-column>
      <el-table-column prop="createTime" label="创建时间" sortable></el-table-column>
      <el-table-column label="操作" width="120">
        <template scope="scope">
          <el-button
            :plain="true"
            size="small"
            type="primary"
            @click="handleEdit(scope.row.wareId)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-row class="m-t">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currPage"
        :page-sizes="[10, 20, 30, 40]"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        class="pull-right">
      </el-pagination>
    </el-row>
  </section>
</template>
<script>
import { readWareList } from '@/api'
export default {
  data () {
    return {
      filter: {
        name: '',
        code: '',
      },
      currPage: 1,
      pageSize: 20,
      total: 0,
      loading: false,
      wareList: [],
      filteredWare: [
        {
          ware_code: '001',
          ware_name: '',
          provider_id: '',
          create_time: '',
          ware_kind: '',
          suggested_price: '',
          key_words: '',
          status: '',
          verify_status: ''
        }
      ]
    }
  },
  methods: {
    formatStatus (row, column) {
      return row.verifyStatus === 1 ? '审核通过' : row.verifyStatus === 0 ? '未审核' : row.verifyStatus === 2 ? '审核未通过' : '未知'
    },
    getWareList () {
      let data = {
        currPage: this.currPage,
        pageSize: this.pageSize,
        wareName: this.filter.name,
        wareCode: this.filter.code,
      }
      console.log(data)
      readWareList(data).then(res => {
        console.log(res)
        if (res.data.code === '0001') {
          let page = res.data.result.pageInfo;
          this.currPage = page.currPage;
          this.pageSize = page.pageSize;
          this.total = page.count;
          this.wareList = res.data.result.wareList
        } else {
          this.$message.error(res.data.message)
        }
      })
      .catch(err => {
        console.log(err)
        this.catchError(err.response)
      })
    },
    handleIconClick () {
      console.log(this.criteria)
    },
    handleSizeChange (val) {
    },
    handleCurrentChange (val) {
      this.currPage = val
    },
    handleEdit (wareId) {
      this.$router.push({
        path: '/provider/ware/edit/base?wareId=' + wareId
      })
    }
  },
  mounted () {
    this.getWareList()
  },
  computed: {
    // filteredWare () {
    //   return this.wareList.filter(ware => ware.ware_name.indexOf(this.criteria) !== -1)
    // }
  }
}
</script>
