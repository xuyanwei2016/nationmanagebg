<template>
  <el-container>

    <el-aside width="200px" class="tree-aside">
      <el-tree :data="navTree" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
    </el-aside>

    <el-main>
    <div class="nav-buttons-container">
      <el-button v-if="authBtns.map(i => i.idName).indexOf('uploadBtn') >= 0" @click="uploadBtn"> {{authBtns.filter(i => i.idName=='uploadBtn')[0].name}}<i v-if="authBtns.filter(i => i.idName=='uploadBtn').length>0 && authBtns.filter(i => i.idName=='uploadBtn')[0].icon" :class="[authBtns[authBtns.map(i => i.idName).indexOf('uploadBtn')].icon, 'el-icon--right']"></i></el-button>
      <el-button v-if="authBtns.map(i => i.idName).indexOf('batchDeleteBtn') >= 0" @click="batchDeleteBtn"> {{authBtns.filter(i => i.idName=='batchDeleteBtn')[0].name}}<i v-if="authBtns.filter(i => i.idName=='batchDeleteBtn').length>0 && authBtns.filter(i => i.idName=='batchDeleteBtn')[0].icon" :class="[authBtns[authBtns.map(i => i.idName).indexOf('batchDeleteBtn')].icon, 'el-icon--right']"></i></el-button>
    </div>

    <el-form inline class="form-inline">
      <el-form-item label="资源名称">
        <el-input @keyup.enter.native="handleFilter" v-model="listQuery.name" clearable></el-input>
      </el-form-item>
      <el-form-item label="选择时间">
        <el-date-picker v-model="timeLimit" value-format="yyyy-MM-dd" type="daterange" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" @keyup.enter.native="handleFilter">
        </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="handleFilter">检索</el-button><el-button @click="resetFilter">重置</el-button>
      </el-form-item>
    </el-form>
    <el-table :key='tableKey' :data="list" v-loading="listLoading" element-loading-text="给我一点时间" border highlight-current-row stripe @selection-change="chioceList">
      <el-table-column type="selection" align="center"></el-table-column>
      <el-table-column type="index" align="center"></el-table-column>
      <el-table-column label="资源名称" min-width="160" align="center" :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <span>{{scope.row.name}}</span>
        </template>
      </el-table-column>
      <el-table-column label="作者" min-width="180" align="center">
        <template slot-scope="scope">
          <span>{{scope.row.source}}</span>
        </template>
      </el-table-column>
      <el-table-column label="上传时间" min-width="120" align="center">
        <template slot-scope="scope">
          <span>{{scope.row.uploadTime}}</span>
        </template>
      </el-table-column>
      <el-table-column class-name="small-padding fixed-width" label="操作" width="80" align="center">
        <template slot-scope="scope">
          <router-link :to="{ path:`pdf/details`, query:{detailsId:scope.row.id}}" v-if="authBtns.filter(i => i.idName=='detailsBtn').length > 0"><i :class="authBtns.filter(i => i.idName=='detailsBtn')[0].icon" :title="authBtns.filter(i => i.idName=='detailsBtn')[0].name"></i></router-link>
          <router-link :to="{ path:`pdf/details`, query:{id:scope.row.id}}" v-if="authBtns.filter(i => i.idName=='editBtn').length > 0"><i :class="authBtns.filter(i => i.idName=='editBtn')[0].icon" :title="authBtns.filter(i => i.idName=='editBtn')[0].name"></i></router-link>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog title="导入课件" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="resourceForm" ref="resourceForm">
        <el-form-item>
          <span>操作提示:单次上传最多10个文件,全部上传完成前请勿关闭窗口,避免数据丢失!</span>
          <el-upload v-model="resourceForm.pdf" class="upload-demo" ref="my-upload" multiple :limit="10" :action="`${uploadUrl}${uploadPath}/info`" :on-exceed="handleExceed" :file-list="fileList"  :on-success="handlePDFSuccess" :before-upload="beforePDFUpload">
            <el-button size="small" type="primary">选择文件(可多选)</el-button>
            <div slot="tip" class="el-upload__tip">只能上传pdf文件</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <el-form inline :model="resourceForm">
          <el-form-item label="来源" >
            <el-input v-model="resourceForm.source" placeholder="来源" style="width: 200px;"></el-input>
          </el-form-item>
          <el-form-item label="关键词" >
            <el-input v-model="resourceForm.keywords" placeholder="关键词" style="width: 200px;"></el-input>
          </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="allUpload">确 定</el-button>
        <el-button @click="clearFiles">取 消</el-button>
      </div>
    </el-dialog>

    <div class="pagination-container">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="listQuery.pageNum" :page-sizes="[30,50,100,200,500]" :page-size="listQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total" align="center">
      </el-pagination>
    </div>
    </el-main>

  </el-container>
</template>

<style scoped>
  .el-row {
  width: 800px;
  margin: 0 auto;
}
.el-button {
  border-radius: 5px;
}
</style>

<script>
import { getNavAPI } from '@/api/database/theory'
import { uploadUrl, uploadPath, requestPath } from '@/utils/global.js'
import { getListAPI, batchDeleteAPI, importAPI } from '@/api/database/pdf'
import { requestMsg, confirm } from '@/utils/publicFunctions'
import { getMultipleSelectionIDs } from '@/filters/getIds'
import store from '@/store'
import { mapGetters } from 'vuex'

export default {
  data() {
    return {
      batchList: [],
      authBtns: [],
      tableKey: 0,
      list: null,
      total: null,
      listLoading: true,
      listQuery: {
        pageNum: 1,
        pageSize: 100,
        nationsType: this.$store.getters.getNation
      },
      timeLimit: null,
      navTree: [],
      defaultProps: {
        children: 'list',
        label: 'showName'
      },
      dialogFormVisible: false,
      resourceForm: {
        names: [],
        pdfs: [],
        size: '',
        timeLength: '',
        nationsType: this.$store.getters.getNation
      },
      fileList: []
    }
  },
  computed: {
    ...mapGetters([
      'getNation'
    ])
  },
  watch: {
    getNation: function (val) {
      this.listQuery.nationsType = val
      this.resourceForm.nationsType = val
      this.getList()
    },
  },
  created(){
    this.authBtns = store.getters.pageBtn[this.$route.meta.routerIds[0]]
    this.uploadUrl = uploadUrl
    this.uploadPath = uploadPath
  },
  mounted() {
    this.getList()
    this.getNav()
  },
  methods: {
    getNav() {
      getNavAPI('PDF').then(res => {
        // res.data.data = [{showName:'全部'}].concat(res.data.data)
        // res.data.code === 0 ? this.navTree = res.data.data : this.$message.error('获取nav数据失败')
        if (res.data.code === 0) {
          let treeList = [];
          res.data.data.map(el => {
            treeList = treeList.concat(el.list);
          });
          this.navTreeDia = treeList;
          this.navTree = [{ showName: "全部" }].concat(treeList);
        }
      })
    },
    getList() {
      this.listLoading = true
      getListAPI(this.listQuery).then(res => {
        if(res.data.code === 0){
          this.list = res.data.data.list
          this.total = res.data.data.total
        } else {
          this.$message.error('获取数据失败');
        }
        this.listLoading = false
      })
    },
    handleNodeClick(data){ //点击树节点获取table数据
      this.listQuery.diyTypeId = data.id
      this.listQuery.diyTypeCode = data.code
      this.getList()
    },
    batchDeleteBtn(){//批量删除--不能删除已上架的数据
      //批量删除
      if (this.batchList.length == 0) {
        this.$message.error("请选择至少一条记录进行操作");
      } else {
        const dataType = this.batchList.every(data => data.status != 1)
        // console.log(dataType)
          if(dataType) {
            const ids = getMultipleSelectionIDs(this.batchList);
            confirm.apply(this,['确认删除吗？']).then(() => {
              batchDeleteAPI(ids).then(res => {
                requestMsg.apply(this,[res.data,'删除'])
                this.getList()
              })
            })
          } else {
            this.$message.error('已上架数据不能删除')
          }
      }
    },
    chioceList(val) {
      this.batchList = val;
    },
    uploadBtn(){//导入pdf
      const id = this.listQuery.diyTypeId
      this.resourceForm.source = ''
      this.resourceForm.keywords = ''
      if(id) {
        this.dialogFormVisible = true
      } else {
        this.$message.error('请选择自定义分类!')
      }
    },
    handleExceed(files, fileList) {
      this.$message.warning(`单次上传最多10个文件`);
    },
    // 全部上传
    allUpload() {
      this.resourceForm.diyType = {
        code: this.listQuery.diyTypeCode,
        dtId: this.listQuery.diyTypeId
      }
      console.log(this.resourceForm.names.length,'aaaaaaa');

      // this.resourceForm.divTypeCode = this.listQuery.diyTypeCode
      // this.resourceForm.divTypeId = this.listQuery.diyTypeId
      if(this.resourceForm.names.length > 0) {
        importAPI(this.resourceForm).then(res => {
          if(res.data.code === 0) {
            this.$message.success('导入成功')
            this.$refs['resourceForm'].resetFields()
            this.$refs['my-upload'].clearFiles()
            this.dialogFormVisible = false
            this.getList()
          } else {
            this.$message.error('导入失败')
          }
        })
      }else {
        this.$message.error('请先选择pdf资源!')
      }
    },
    // 全部取消
    clearFiles() {
      this.$refs['my-upload'].clearFiles();
      this.resourceForm = {}
      this.dialogFormVisible = false
    },
    handleFilter(){ //检索
      if(this.timeLimit){
          this.listQuery.beginTime = this.timeLimit[0]
          this.listQuery.endTime = this.timeLimit[1]
        } else {
          this.listQuery.beginTime = null
          this.listQuery.endTime = null
        }
      store.dispatch('SetSupplierSerKeys',this.listQuery).then(res => {
        console.log(res,'保存查询成功')
      })
      this.listQuery.pageNum = 1
      this.getList()
    },
    resetFilter(){ //重置
      this.timeLimit = []
      this.listQuery = {
        pageNum: 1,
        pageSize: this.listQuery.pageSize,
        diyTypeId: this.listQuery.diyTypeId,
        diyTypeCode: this.listQuery.diyTypeCode,
        nationsType: this.listQuery.nationsType
      }
      this.getList()
    },
    beforePDFUpload(file){ //beforePDFUpload、handlePDFSuccess上传视频相关
      let type = file.name.slice(file.name.lastIndexOf('.')+1).toLowerCase()
      const isPDF = type === 'pdf';
      if (!isPDF) {
        this.$message.error('上传pdf只能是 pdf 格式!');
        return isPDF
      }
    },
    handlePDFSuccess(response, file, fileList){
      this.$message.success('上传成功')
      this.resourceForm.names.push(response.originalFileName)
      this.resourceForm.pdfs.push(response.fileName)
    },
    handleSizeChange(val) {
      this.listQuery.pageSize = val
      this.getList()
    },
    handleCurrentChange(val) {
      this.listQuery.pageNum = val
      this.getList()
    }
  }
}
</script>
