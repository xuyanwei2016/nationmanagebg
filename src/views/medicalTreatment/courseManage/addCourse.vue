<template>
  <div class="app-container" @click="authorStatus = 1">
    <el-form :model="resourceForm" ref="resourceForm" :rules="rules" label-position="right" label-width="120px" class="ib">
      <div class="title">基本信息</div>
      <el-form-item label="课程名称：" prop="name">
        <el-input v-model="resourceForm.name" clearable placeholder="" style="width:300px;margin-right:10px"></el-input>
      </el-form-item>
      <el-form-item label="作者：" prop="memberId" >
        <el-select v-model="resourceForm.memberId" style="width:300px;margin-right:10px">
            <el-option v-for="item in memberList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="课程简介：">
        <tinymce :height=300 ref="editor" v-model="resourceForm.synopsis" style="width:800px"></tinymce>
      </el-form-item>
      <el-form-item label="详情封面：">
        <el-upload :disabled="$route.query.detailsId?true:false" class="avatar-uploader" :action="`${uploadUrl}${uploadPath}`" :show-file-list="false" :on-remove="handleRemove" :on-preview="handlePicPreview" :on-success="handlePicSuccess" :before-upload="beforePicUpload">
          <img :src="`${uploadUrl}?fileName=${resourceForm.cover}&isOnLine=true`" v-if="resourceForm.cover" class="avatar" style="width: 178px;height:178px;object:fit;border:dashed 1px #ccc" >
          <img src="../../../assets/img/bookDefault.png" v-else class="avatar-uploader-icon" style="width: 100%; height: 100%;">
        </el-upload>
        <span class="tip">建议尺寸600*420px或4:3，仅支持jpg、jpeg、png、png格式，不超过2M</span>
      </el-form-item>
      <el-form-item label="宣传图：">
        <el-upload :disabled="$route.query.detailsId?true:false" class="avatar-uploader" :action="`${uploadUrl}${uploadPath}`" :show-file-list="false" :on-remove="handleRemove" :on-preview="handlePicPreview" :on-success="handlePicSuccess" :before-upload="beforePicUpload">
          <img :src="`${uploadUrl}?fileName=${resourceForm.cover}&isOnLine=true`" v-if="resourceForm.cover" class="avatar" style="width: 178px;height:178px;object:fit;border:dashed 1px #ccc" >
          <img src="../../../assets/img/bookDefault.png" v-else class="avatar-uploader-icon" style="width: 100%; height: 100%;">
        </el-upload>
        <span class="tip">建议尺寸600*420px或4:3，仅支持jpg、jpeg、png、png格式，不超过2M</span>
      </el-form-item>
      <el-form-item label="总课时：" prop="name">
        <el-input v-model="resourceForm.name" clearable placeholder="" style="width:300px;margin-right:10px"></el-input>
      </el-form-item>
      <el-form-item label="排序号：" prop="name">
        <el-input v-model="resourceForm.name" clearable placeholder="" style="width:300px;margin-right:10px"></el-input>
      </el-form-item>
      <el-form-item label="状态：" prop="price">
        <el-radio-group v-model="resourceForm.realPriceType">
          <el-radio :label="0">上线</el-radio>
          <el-radio :label="1">下线</el-radio>
        </el-radio-group>
      </el-form-item>
      <div class="title">课程状态</div>
      <el-form-item label="是否显示：" prop="price">
        <el-radio-group v-model="resourceForm.realPriceType">
          <el-radio :label="0">上架</el-radio>
          <el-radio :label="1">下架</el-radio>
        </el-radio-group>
        <span class="tip">审核通过之后平台不显示</span>
      </el-form-item>
      <el-form-item label="售价：" prop="realPriceType">
        <el-radio v-model="resourceForm.realPriceType" :label=0>付费</el-radio>
        <el-radio v-model="resourceForm.realPriceType" :label=1>免费</el-radio>
      </el-form-item>
      <el-form-item prop="realPrice">
        <el-input v-model="resourceForm.realPrice" clearable placeholder="请输入售价" v-if="resourceForm.realPriceType == 0" style="width:250px;margin-right:10px"></el-input>
        <span v-if="resourceForm.realPriceType == 0">元</span>
      </el-form-item>
      

      <div class="title">参与方式</div>
      <el-form-item label="售价：" prop="realPriceType">
        <el-radio v-model="resourceForm.realPriceType" :label=0>付费</el-radio>
        <el-radio v-model="resourceForm.realPriceType" :label=1>免费</el-radio>
      </el-form-item>
      <el-form-item prop="realPrice">
        <el-input v-model="resourceForm.realPrice" clearable placeholder="请输入售价" v-if="resourceForm.realPriceType == 0" style="width:250px;margin-right:10px"></el-input>
        <span v-if="resourceForm.realPriceType == 0">元</span>
      </el-form-item>
      <el-form-item label="划线价：" prop="price">
        <el-input v-model="resourceForm.price" clearable placeholder="请输入定价" style="width:250px;margin-right:10px"></el-input>
        <span>元</span>
        <span class="tip">请设置资源划线价钱，不填写默认不显示</span>
      </el-form-item>
    </el-form>

    <div slot="footer" class="dialog-footer" v-if="pageStatus == 0">
      <el-button @click="dataSubmit">保存</el-button>
      <el-button @click="$router.go(-1)">取消</el-button>
    </div>

  </div>
</template>

<script>
import Tinymce from '@/components/Tinymce'
import { uploadUrl, uploadPath, requestPath } from '@/utils/global.js'
import queryString from 'query-string'
import { getOptionAPI, getNavAPI, getLabelAPI } from '@/api/database/video'
import { saveAPI, detailsAPI, editAPI, commonUploadAPI } from '@/api/database/theory' //接口
import { authorLibraryAPI } from '@/api/database/article' //接口
import { requestMsg, confirm } from '@/utils/publicFunctions'
import { getMultipleSelectionIDs } from '@/filters/getIds'
import { mapGetters } from 'vuex'

export default {
  components:{ Tinymce },
  data() {
    return {
      resourceForm: {
        video:null,
        audio:null,
        realPriceType:1,
        readTextPower:1,
        realPrice:'',
        nationsType: this.$store.getters.getNation,
        pdf_zh: ''
      },
      uploading:false,
      pageStatus:0,
      diyTypeCode: [],
      defaultProps: {
        children: 'list',
        label: 'showName'
      },
      options: [],
      navTree: [],
      rules:{
        title:[
          { required: true, message: '请输入文章标题', trigger: 'change' },
          {
            pattern:/^.{1,50}$/,message: '文章标题长度必须小于50位', trigger: 'change'
          }
        ],
        realPrice:[
          { required: false,pattern:/(^[1-9](\d+)?(\.\d{1,2})?$)|(^\d\.\d{1,2}$)/,message: '请输入0.1-99999之间的数字，最多保留两位小数', trigger: 'change' },
        ],
        price:[
          { required: false,pattern:/^\d+(\.\d{0,2})?$/,message: '请输入0-99999之间的数字，最多保留两位小数', trigger: 'change' },
        ],
        author:[
          { required: false, pattern: /^[\w\u4e00-\u9fa5]{1,10}$/, message: '作者长度必须小于10位', trigger: 'change'}
        ],
        source:[
          { required: false, pattern: /^.{1,50}$/, message: '来源长度必须小于50位', trigger: 'change'}
        ],
        keywords:[
          { required: false, pattern: /^.{1,30}$/, message: '关键词长度必须小于30位', trigger: 'change'}
        ],
        readTextPower:[
          { required: true}
        ],
        realPriceType: [{ required: true}]
      },
      checkedKeys:[],
      name:[
        '理论','药物','方剂','临床','医案','文化','机构','其他','未分类','蒙文'
      ],
      fileName: '',
      resourceList: [],
      authorList:[],

    }
  },
  computed: {
    ...mapGetters([
      'getNation'
    ])
  },
  watch: {
    getNation: function (val) {
      this.resourceForm.nationsType = val
    },
  },
  created(){
    this.uploadUrl = uploadUrl
    this.uploadPath = uploadPath
  },
  mounted() {
    this.isEdit();
  },
  methods: {
     // 查看pdf
    queryPDF() {
      window.open(uploadUrl+"?fileName="+this.resourceForm.pdf+"&isOnLine=true")
    },
    authorLibrary() {//查询
      this.authorList = [];
      this.resourceForm.authorId = '';
      if(this.resourceForm.author.split(" ").join("").length != 0){
        this.authorStatus = 0
        authorLibraryAPI(this.resourceForm.author.split(" ").join("")).then(res => {
          if(res.data.code==0){
            this.authorList = res.data.data;
            this.resourceForm.authorNameAndId=this.resourceForm.author.trim();
          }else{
            this.$message.error('获取数据失败');
          }
        })
      }
    },
    loseFocus(){
      this.authorStatus = 1;
    },
    choiceAuthor(name,id) {//选择模糊查询的作者
      this.resourceForm.author = name;
      this.resourceForm.authorId = id;
      this.resourceForm.authorNameAndId = `${name},${id}`;
      this.authorStatus = 1
    },
    // 提交公共资源
    commonUpload() {
      if(this.resourceList.length != 0) {
        commonUploadAPI(this.resourceList).then(res => {
        })
      }
    },
    isEdit(){
      this.resourceForm.resourceType = this.$route.query.articleType; //文章类型 0全部,1医著 2医家 31理论 32药物 33方剂 34临床 35医案 36文化 37机构 38医论 48其他 49未分类 50蒙文
      this.$route.meta.title = '添加蒙文文章'
      const id = this.$route.query.id; //修改id
      const detailsId = this.$route.query.detailsId; //查看id
      if(id){//修改
        this.$route.meta.title = '修改蒙文文章'
        this.pageDetails(id);
      }
      if(detailsId) {//查看
        this.pageStatus = 1
        this.$route.meta.title = '查看蒙文文章'
        this.pageDetails(detailsId);
      }
    },
    pageDetails(id){//查看和修改的页面信息
      detailsAPI(id).then(res => {
        if (res.data.code === 0) {
          if(res.data.data.realPrice == 0){//免费
            this.resourceForm = {...res.data.data,realPriceType:1}
          } else{ //付费
            this.resourceForm = {...res.data.data,realPriceType:0}
          }
        } else{
          this.$message.error('加载失败')
        }
      })
    },
    //pdf上传
    beforePdfUpload(file){
      const isPDF = file.type === 'application/pdf'
      if (!isPDF) {
        this.$message.error('附件文件只能是 PDF 格式!')
        return isPDF;
      }
      this.uploading = true
    },
    handlePdfSuccess(response, file, fileList){
      this.resourceForm.pdf_zh = response.originalFileName;
      this.resourceForm.pdf = response.fileName
      const pdf = {
        keywords: this.resourceForm.keywords,
        name: response.originalFileName,
        nationsType: this.resourceForm.nationsType,
        resourceCode: response.fileName,
        resourceType:  66,
        author: this.resourceForm.author,
        source: this.resourceForm.source,
      }
      this.resourceList.push(pdf)
      this.uploading = false
    },
    handlePdfExceed(files, fileList) {
      this.$message.warning(`单次上传最多1个文件`);
    },
    handlePdfRemove() {//删除pdf
      this.resourceForm.pdf = null;
    },
    //保存
    dataSubmit() {
      this.$refs['resourceForm'].validate((valid) => {
        if (valid) {//必填名称--规则
          if((this.resourceForm.realPriceType == 0 && this.resourceForm.realPrice != '') || this.resourceForm.realPriceType == 1){//售价---必填
            if(this.resourceForm.realPriceType != 0) this.resourceForm.realPrice = 0  // 售价
            const id = this.$route.query.id; //修改id
            delete this.resourceForm.author;
            delete this.resourceForm.authorId;
              if(id) {//修改
                this.resourceForm.id = id;
                editAPI(this.resourceForm).then(res => {
                  if (res.data.code === 0) {
                    this.$message.success('编辑成功')
                    this.$router.push({ path: '/database/mongolian' });
                    this.commonUpload()
                    this.$router.go(-1);
                  } else{
                    this.$message.error('编辑失败')
                  }
                })
              } else{
                saveAPI(this.resourceForm).then(res => {
                  if (res.data.code === 0) {
                    this.$message.success('添加成功')
                    this.$router.push({ path: '/database/mongolian' });
                    this.commonUpload()
                    this.$router.go(-1);
                  } else{
                    this.$message.error('添加失败')
                  }
                })
              }
          } else{
            this.$message.error('请填写售价哦');
          }
        } else{
          this.$message.error('请将信息补全!');
        }
      })
    },

    cancel() {//取消
      this.$router.go(-1);
    },
  }
}
</script>

<style scoped>
.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
  .ib{
    display:inline-block;
  }
  .tip{
    color: #ccc;
    margin-left: 30px;
  }
  .t {
    color: #ccc;
    display: inline-block;
    margin-top: 10px;
  }
  .aa{
    display: block !important;
    width: 100%;
  }
  .title {
    font-weight: bold;
    margin-bottom: 20px;
    margin-top: 10px;
  }
  .el-button {
    margin-bottom: 10px;
  }
</style>
