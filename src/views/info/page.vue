<template>
  <div class="app-container">
    <div class="filter-container">
      <!--标题检索输入框-->
      <el-input v-model="listQuery.title" :placeholder="$t('table.title')" style="width: 200px;" class="filter-item" @keyup.enter.native="getList" />
      <!--新闻类型检索区-->
      <el-select v-model="listQuery.type" :placeholder="$t('table.type')" clearable class="filter-item" style="width: 130px" @change="getList">
        <el-option v-for="item in infoType" :key="item.typeName" :label="item.typeName" :value="item.id" />
      </el-select>
      <!--搜索按钮-->
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="getList">
        {{ $t('table.search') }}
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        {{ $t('table.add') }}
      </el-button>
      <!--<el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload">-->
      <!--{{ $t('table.export') }}-->
      <!--</el-button>-->
    </div>
    <el-table v-loading="listLoading" :data="pageData" border style="width: 100%;text-align: center" @row-click="checkInfo">
      <el-table-column prop="" label="图片" width="200" align="center">
        <template slot-scope="scope">
          <el-popover placement="right" title="" trigger="hover">
            <img :src="scope.row.imgUrl">
            <img slot="reference" :src="scope.row.imgUrl" style="max-width: 100%">
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column prop="title" sortable label="标题" width="120" align="center" />
      <el-table-column prop="infoType.typeName" sortable label="类型" width="80" align="center" />
      <el-table-column prop="previewContent" label="正文预览" />
      <el-table-column prop="read" sortable label="浏览量" width="100" align="center" :filters="[{text:'火爆',value:200},{text:'热点',value:100}]" :filter-method="filterTag">
        <template slot-scope="scope">
          <el-tag :type="scope.row.read | typeFilter">{{ scope.row.read }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="date" sortable label="发布时间" width="120" align="center" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button size="mini" type="danger" @click="handleDelete(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

    <el-dialog title="资讯编辑" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="类型" prop="type">
          <el-select v-model="temp.infoType" value-key="id" class="filter-item">
            <el-option v-for="item in infoType" :key="item.id" :label="item.typeName" :value="item" />
          </el-select>
        </el-form-item>
        <el-form-item label="正文">
          <el-input v-model="temp.content" tabindex="2" :autosize="{ minRows: 5, maxRows: 10}" type="textarea" style="width: 500px" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          取消
        </el-button>
        <el-button type="primary" @click="updateData()">
          保存
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { fetchInfoList, updateInfo } from '@/api/info'
import Pagination from '@/components/Pagination'
import waves from '@/directive/waves' // waves directive
// 不变化的业务字典数据，可以定义为全局常量
const infoType = [
  { id: 1, typeName: '中标信息' }, { id: 2, typeName: '行业热点' }, { id: 3, typeName: '公司政策' }
]

export default {
  components: {
    Pagination
  },
  directives: { waves },
  filters: {
    // 对热度数据进行过滤，生成不同颜色的标签
    typeFilter(read) {
      return read >= 200 ? 'danger' : (read >= 100 ? 'warning' : 'success')
    }
  },
  data() {
    return {
      // 从后台获取的总数居
      newInfo: {},
      tableData: [],
      infoType,
      total: 0,
      listQuery: {
        page: 1,
        limit: 20,
        title: undefined,
        type: undefined
      },
      // 前台每页要显示的数据
      pageData: [],
      dialogFormVisible: false, // 控制编辑表格是否显示
      //        tempInfoType:"", //待编辑的数据
      //        tempTitle:"",
      //        tempContent:"",
      //        tempId:-1,
      listLoading: false, // 控制loading加载特效
      temp: {
        id: undefined,
        title: '',
        content: '',
        previewContent: '',
        date: '',
        infoType: { id: 0, typeName: '' }
      }
    }
  },
  mounted() {
    this.listLoading = true

    if (this.$route.query) { this.newInfo = this.$route.query }

    // 首次挂载新闻列表组件时获得所有新闻数据
    fetchInfoList({}).then(response => {
      console.log(response)
      this.tableData = response.data.items
      // 获得数据总量
      this.total = response.data.total

      this.tableData.forEach((news) => {
        news.previewContent = news.content.substring(0, 100)
      })

      if (this.newInfo.title) { this.tableData.unshift(this.newInfo) }
      // 获取第一页
      this.getList()

      this.listLoading = false// 关闭加载框
    })
  },
  methods: {
    // 获得每页要显示的数据
    getList() {
      const { page, limit, title, type } = this.listQuery

      // 过滤查询结果集（先过滤，再分页）
      const filterData = this.tableData.filter(item => {
        if (title && item.title.indexOf(title) < 0) return false
        if (type && item.infoType.id !== type) return false
        return true
      })
      this.total = filterData.length

      // 从总数据中过滤出当前页要显示的数据集
      this.pageData = filterData.filter((item, index) =>
        index < page * limit && index >= limit * (page - 1)
      )
    },
    filterTag(value, row) {
      if (value === 200) { return row.read >= value } else { return row.read > value && row.read < 200 }
    },
    handleCreate() {
      this.$router.push({ name: 'infoCreate', params: { tableData: this.tableData }})
    },
    checkInfo(row) { // 查看资讯详情
      this.tableData.forEach((news, idx) => {
        if (news.id === row.id) {
          this.tempId = idx
        }
      })
      this.tableData[this.tempId].read += 1
      this.getList()
      this.$router.push({ path: 'infoDetail', query: row })
    },
    handleUpdate(row) {
      //        this.tableData.forEach((news, idx) => {
      //          if (news.id == row.id) {
      //            this.tempId = idx;
      //          }
      //        })
      //        this.tempInfoType=this.tableData[this.tempId].infoType;
      //        this.tempTitle=this.tableData[this.tempId].title;
      //        this.tempContent=this.tableData[this.tempId].content;
      //        this.dialogFormVisible = true;
      //        this.$nextTick(() => {
      //          this.$refs['dataForm'].clearValidate()
      //        })
      this.temp = Object.assign({}, row) // copy obj
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })

      event.stopPropagation()
    },
    updateData() {
      //        this.$confirm('是否保存修改的资讯, 是否继续?', '提示', {
      //          confirmButtonText: '确定',
      //          cancelButtonText: '取消',
      //          type: 'warning'
      //        }).then(() => {
      //          this.tableData[this.tempId].title=this.tempTitle;
      //          this.tableData[this.tempId].content=this.tempContent;
      //          this.tableData[this.tempId].previewContent=this.tempContent.substring(0,100);
      //          this.tableData[this.tempId].infoType=this.tempInfoType;
      //          let nowTime= new Date();
      //          this.tableData[this.tempId].date=nowTime.getFullYear()+'-'+(nowTime.getMonth()+1)+'-'+nowTime.getDate();
      //          this.dialogFormVisible = false;
      //          this.getList();
      //
      //          this.$message({
      //            type: 'success',
      //            message: '保存成功!'
      //          });
      //        }).catch(() => {
      //          this.$message({
      //            type: 'info',
      //            message: '已取消保存'
      //          });
      //        });
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.previewContent = this.temp.content.substring(0, 100)
          const nowTime = new Date()
          this.temp.date = nowTime.getFullYear() + '-' + (nowTime.getMonth() + 1) + '-' + nowTime.getDate()
          const tempData = Object.assign({}, this.temp)
          updateInfo(tempData).then(() => {
            for (const v of this.tableData) {
              if (v.id === this.temp.id) {
                const index = this.tableData.indexOf(v)
                console.log(this.temp)
                this.tableData.splice(index, 1, this.temp)
                this.getList()
                break
              }
            }
            this.dialogFormVisible = false
            this.$notify({
              title: '成功',
              message: '更新成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    // 删除数据
    handleDelete(row) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let index = -1
        this.tableData.forEach((news, idx) => {
          if (news.id === row.id) {
            index = idx
          }
        })
        this.tableData.splice(index, 1)
        this.getList()

        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      event.stopPropagation()
    }
  }
}

</script>
<style>
</style>
