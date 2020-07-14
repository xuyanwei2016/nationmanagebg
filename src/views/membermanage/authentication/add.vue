<template>
  <div class="app-container" @click="authorStatus=1,translatorStatus=1">
    <el-form :model="dataFrom" ref="dataFrom" :rules="rules" label-position="right" label-width="140px" class="ib">
      <el-row>
        <el-col :span="14">
          <el-form-item label="会员名称：" prop="meberId" class="float">
            <el-select v-model="dataFrom.meberId" style="width:300px" clearable placeholder="请选择会员">
              <el-option
                v-for="item in memberList"
                :key="item.value"
                :label="item.name"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="姓名：" prop="name" class="float">
            <el-input v-model="dataFrom.name" clearable placeholder=""
                      style="width:300px !important"></el-input>
          </el-form-item>
          <el-form-item label="性别：" prop="sex">
            <el-radio-group v-model="dataFrom.sex">
              <el-radio label="1">男</el-radio>
              <el-radio label="0">女</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="省份：" prop="province" class="float" style="clear:both">
            <el-select v-model="dataFrom.province" placeholder="请输入省份"  @change="provinceChanged" style="width:300px">
                <!-- <el-option :key="''" :label="'全部'" :value="''"></el-option> -->
                <el-option v-for="item in mapData.provinceList" :key="item.adcode" :label="item.name" :value="item.adcode"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="市区：" prop="city" class="float" style="clear:both">
            <el-select v-model="dataFrom.city" placeholder="请输入市" @change="cityChanged" style="width:300px">
                <!-- <el-option :key="''" :label="'全部'" :value="''"></el-option> -->
                <el-option v-for="item in mapData.cityList" :key="item.adcode" :label="item.name" :value="item.adcode"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="地区：" prop="district" class="float" style="clear:both">
            <el-select v-model="dataFrom.district" placeholder="请输入地区"  @change="areaChanged" style="width:300px">
                <!-- <el-option :key="''" :label="'全部'" :value="''"></el-option> -->
                <el-option v-for="item in mapData.areaList" :key="item.adcode" :label="item.name" :value="item.adcode"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="学校：" class="float" prop="school"  v-if="dataFrom.authenticationType === '1'">
            <el-input v-model="dataFrom.school" clearable style="width:300px !important"></el-input>
          </el-form-item>
          <el-form-item label="专业：" class="float" prop="major"  v-if="dataFrom.authenticationType === '1'">
            <el-input v-model="dataFrom.major" clearable style="width:300px !important"></el-input>
          </el-form-item>
          <el-form-item label="规培医院：" class="float" prop="isbn"  v-if="dataFrom.authenticationType === '1'">
            <el-input v-model="dataFrom.isbn" clearable style="width:300px !important"></el-input>
          </el-form-item>
          
          <el-form-item label="师承：" class="float" prop="master" v-if="dataFrom.authenticationType === '0'||dataFrom.authenticationType === '2'">
            <el-input v-model="dataFrom.master" clearable style="width:300px !important"></el-input>
          </el-form-item>
          <el-form-item label="医疗机构：" class="float" prop="medicalInstitution"  v-if="dataFrom.authenticationType === '0'">
            <el-input v-model="dataFrom.medicalInstitution" clearable style="width:300px !important"></el-input>
          </el-form-item>
          <el-form-item label="科室：" class="float" prop="departmentId"  v-if="dataFrom.authenticationType === '0'">
            <el-select v-model="dataFrom.departmentId" style="width:300px" clearable placeholder="请选择科室">
              <el-option
                v-for="item in departmentList"
                :key="item.value"
                :label="item.name"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="职称：" class="float" prop="professionalTitle"  v-if="dataFrom.authenticationType === '0'">
            <el-select v-model="dataFrom.professionalTitle" style="width:300px" clearable placeholder="请选择职称">
              <el-option
                v-for="item in titleList"
                :key="item.value"
                :label="item.name"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="认证类型：" prop="authenticationType">
            <el-radio-group v-model="dataFrom.authenticationType"  @change="changeType">
              <el-radio label="0">医师资格认证</el-radio>          
              <el-radio label="1">医学院师生认证</el-radio>
              <el-radio label="2">实名认证</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="证件类型" prop="IDCard" v-show="dataFrom.authenticationType === '0'">
            <el-radio-group v-model="dataFrom.IDCard"  @change="changeCard">
              <el-radio label="0">执业/助理医师资格证</el-radio>          
              <el-radio label="1">医师佩戴医院胸牌半身照</el-radio>
              <el-radio label="2">从业人员资格证书</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-col>
        <el-col :span="10" class="form-line-three">
          <el-form-item label="身份证正面：" prop="IDCardImgFront" style="height:100px">
            <div class="upload-item" v-if="!dataFrom.IDCardImgFront">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="IDCardImgFront" @change="uploadLogo($event,'1')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.IDCardImgFront" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.IDCardImgFront)" alt="" @mouseenter="hoverIndex = '1'">
                <div class="hover-img" v-show="hoverIndex == '1'" @mouseenter="hoverIndex = '1'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="IDCardImgFront" @change="uploadLogo($event,'1')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.IDCardImgFront)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
          <el-form-item label="身份证反面：" prop="IDCardImgBack" style="height:100px">
            <div class="upload-item" v-if="!dataFrom.IDCardImgBack">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="IDCardImgBack" @change="uploadLogo($event,'2')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.IDCardImgBack" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.IDCardImgBack)" alt="" @mouseenter="hoverIndex = '2'">
                <div class="hover-img" v-show="hoverIndex == '2'" @mouseenter="hoverIndex = '2'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="IDCardImgBack" @change="uploadLogo($event,'2')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.IDCardImgBack)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
          <el-form-item :label="labelOne" prop="PhysicianImgOne" style="height:100px" v-if="dataFrom.authenticationType !== '2'">
            <div class="upload-item" v-if="!dataFrom.PhysicianImgOne">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="PhysicianImgOne" @change="uploadLogo($event,'3')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.PhysicianImgOne" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.PhysicianImgOne)" alt="" @mouseenter="hoverIndex = '3'">
                <div class="hover-img" v-show="hoverIndex == '3'" @mouseenter="hoverIndex = '3'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="PhysicianImgOne" @change="uploadLogo($event,'3')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.PhysicianImgOne)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
          <el-form-item :label="labelTwo" prop="PhysicianImgTwo" style="height:100px" v-if="dataFrom.authenticationType !== '2'">
            <div class="upload-item" v-if="!dataFrom.PhysicianImgTwo">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="PhysicianImgTwo" @change="uploadLogo($event,'4')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.PhysicianImgTwo" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.PhysicianImgTwo)" alt="" @mouseenter="hoverIndex = '4'">
                <div class="hover-img" v-show="hoverIndex == '4'" @mouseenter="hoverIndex = '4'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="PhysicianImgTwo" @change="uploadLogo($event,'4')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.PhysicianImgTwo)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
          <el-form-item label="医师职称证正面：" prop="TittleImgOne" style="height:100px" v-if="dataFrom.authenticationType === '0'&&dataFrom.IDCard === '0'">
            <div class="upload-item" v-if="!dataFrom.TittleImgOne">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="TittleImgOne" @change="uploadLogo($event,'5')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.TittleImgOne" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.TittleImgOne)" alt="" @mouseenter="hoverIndex = '5'">
                <div class="hover-img" v-show="hoverIndex == '5'" @mouseenter="hoverIndex = '5'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="TittleImgOne" @change="uploadLogo($event,'5')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.TittleImgOne)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
          <el-form-item label="医师职称证反面：" prop="TittleImgTwo" style="height:100px" v-if="dataFrom.authenticationType === '0'&&dataFrom.IDCard === '0'">
            <div class="upload-item" v-if="!dataFrom.TittleImgTwo">
                <div class="item-con">
                    <img :src="addImg" alt="">
                    <span class="upload-text">上传照片</span>
                </div>
                <input name="TittleImgTwo" @change="uploadLogo($event,'6')" ref="file" type="file" class="upLoad-img">
            </div>
            <div v-if="dataFrom.TittleImgTwo" class="showPic">
                <img class="showImg"  :src="imgUrl(dataFrom.TittleImgTwo)" alt="" @mouseenter="hoverIndex = '6'">
                <div class="hover-img" v-show="hoverIndex == '6'" @mouseenter="hoverIndex = '6'" @mouseleave="hoverIndex = ''">
                    <div class="hover-img-text">
                       <!-- v-if="title != '查看学校'" -->
                        <div class="upload-again">
                            重新上传
                        </div>
                        <input name="TittleImgTwo" @change="uploadLogo($event,'6')" ref="file" type="file" class="again-img">
                    </div>
                    <span class="hover-img-text" @click="lookImg(dataFrom.TittleImgTwo)">查看照片</span>
                    <img class="hasimg" :src="hasSuccess" alt="">
                </div>
            </div>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="擅长：" class="float" prop="good">
        <el-col :span="20">
          <el-input v-model="dataFrom.good" type="textarea" :autosize="{ minRows: 2, maxRows: 4}" clearable></el-input>
        </el-col>
      </el-form-item>
      <el-form-item label="简介：" class="float" prop="synopsis">
        <el-col :span="20">
          <el-input v-model="dataFrom.synopsis"  type="textarea" :autosize="{ minRows: 2, maxRows: 4}" clearable></el-input>
        </el-col>
      </el-form-item>
      <el-form-item label="备注：" class="float" prop="remark">
        <el-col :span="20">
          <el-input v-model="dataFrom.remark"  type="textarea" :autosize="{ minRows: 2, maxRows: 4}" clearable></el-input>
        </el-col>
      </el-form-item>
      <div class="dialog-footer">
        <el-button @click="dataSubmit">保存</el-button>
        <el-button @click="$router.go(-1)">取消</el-button>
      </div>
    </el-form>
    <el-dialog title="查看大图" :visible.sync="showImg" width="650px" @close="showImg = false">
            <!-- <div>
              <select name="" id="" v-model="scaleNum"> 
                <option :value="1">1</option>
                <option :value="2">2</option>
                <option :value="3">3</option>
              </select>
            </div> -->
            <div class="img-center">
                <!-- <pic-zoom v-if="showImg" :url="fileUrl(imgPath)" :showEidt="true" :showScale="true" :scale="scaleNum" :big-url="fileUrl(imgPath)"></pic-zoom> -->
                <img class="showBigImg"  :src="imgUrl(imgPath)" alt="">
            </div>
        </el-dialog>
  </div>
</template>

<script>
// import {uploadUrl, uploadPath, requestPath} from '@/utils/global.js'
// import queryString from 'query-string'
// import {getOptionAPI, getNavAPI} from '@/api/database/video'
// import {commonUploadAPI} from '@/api/database/theory' //接口
// import {authorLibraryAPI, translatorLibraryAPI} from '@/api/database/article' //接口
import {saveAPI, detailsAPI, uploadSchoolAPI} from '@/api/member/authentication'
import {requestMsg, confirm} from '@/utils/publicFunctions'
import {getMultipleSelectionIDs} from '@/filters/getIds'
import mapList from '../../../../static/gaode.json'
import hasSuccess from '@/assets/img/entry/success.png';
import addImg from '@/assets/img/entry/addImg.png';
import { baseUrl } from "@/utils/global.js";

export default {
  data() {
    return {
      hasSuccess:hasSuccess,
      addImg:addImg,
      baseUrl:baseUrl,
      dataFrom: {
        meberId:'', // 会员名称
        name:'',    // 姓名
        sex:'1',    // 性别 男1  女0
        province:'',  // 省
        city:'', // 市
        district:'',   //区
        master:"",   // 师承
        medicalInstitution:'', // 医疗机构
        departmentId:'',       // 科室
        professionalTitle:'',   // 职称
        authenticationType:'0',  // 认证类型
        good:"",   // 擅长
        synopsis:'', // 简介
        remark:"",  // 备注
        hospital:'', // 规培医院
        school:'',  // 学校
        major:'', // 专业

        IDCard:"0",   // 证件类型（医师资格认证时使用） 0：执业/助理医师资格证 1：医师佩戴医院胸牌半身照 2：从业人员资格证书
        IDCardImgFront:"",  // 身份证正面
        IDCardImgBack:"",  // 身份证反面
        PhysicianImgOne:"", // 医师执业证第一页 OR 手持医院胸牌照 OR 从业证书 OR 教师或学生证
        PhysicianImgTwo:"",  // 医师执业证第二页 OR 手持医院胸牌照 OR 从业证书
        TittleImgOne:"",  // 职称照片1
        TittleImgTwo:"",  // 职称照片2
      },
      memberList:[
        {
          name:'会员1',
          value:'123'
        }
      ], // 会员列表
      departmentList:[
        {
          name:'科室1',
          value:'456'
        }
      ], // 科室列表
      titleList:[
        {
          name:'职称1',
          value:'4522446'
        }
      ], // 职称列表
      rules: {
        meberId:[{required: true,  message: '请选择会员', trigger: 'change'}],
        name: [{required: true,  message: '请输入姓名', trigger: 'change'}, { max:10,message: '请输入10字以内的姓名'}],
        master:[{required: true,  message: '请输入师承', trigger: 'change'}, { max:20,message: '请输入20字以内的师承'}],
        medicalInstitution:[{required: true,  message: '请输入医疗机构', trigger: 'change'}, { max:20,message: '请输入20字以内的医疗机构'}],
        departmentId:[{required: true,  message: '请选择科室', trigger: 'change'}],
        professionalTitle:[{required: true,  message: '请选择职称', trigger: 'change'}],
        province:[{required: true,  message: '请选择省份', trigger: 'change'}],
        city:[{required: true,  message: '请选择市区', trigger: 'change'}],
        district:[{required: true,  message: '请选择地区', trigger: 'change'}],
        // good: [{required: true,  message: '请输入擅长内容', trigger: 'change'}, { max:10,message: '请输入10字以内的擅长内容'}],
        // synopsis: [{required: true,  message: '请输入简介', trigger: 'change'}, { max:10,message: '请输入10字以内的简介'}],
        remark: [{required: true,  message: '请输入备注', trigger: 'change'}, { max:50,message: '请输入50字以内的备注'}],
        hospital: [{required: true,  message: '请输入规培医院', trigger: 'change'}, { max:20,message: '请输入20字以内的规培医院'}],
        school: [{required: true,  message: '请输入学校', trigger: 'change'}, { max:20,message: '请输入20字以内的学校'}],
        major: [{required: true,  message: '请输入专业', trigger: 'change'}, { max:20,message: '请输入20字以内的专业'}],
        IDCardImgFront:[{ required: true, trigger: "change blur", message: "请选择照片" }],  // 身份证正面
        IDCardImgBack:[{ required: true, trigger: "change blur", message: "请选择照片" }],  // 身份证反面
        PhysicianImgOne:[{ required: true, trigger: "change blur", message: "请选择照片" }], // 医师执业证第一页 OR 手持医院胸牌照 OR 从业证书 OR 教师或学生证
        PhysicianImgTwo:[{ required: true, trigger: "change blur", message: "请选择照片" }],  // 医师执业证第二页 OR 手持医院胸牌照 OR 从业证书
        TittleImgOne:[{ required: false, trigger: "change blur", message: "请选择照片" }], // 职称照片1
        TittleImgTwo:[{ required: false, trigger: "change blur", message: "请选择照片" }],  // 职称照片2
      },
      labelOne:'医师执业证第一页:',
      labelTwo:'医师执业证第二页:',
      hoverIndex:'',
      showImg:false,
      imgPath:'',
      checkedKeys: [],

      /* 地图 */
      mapData:{
          provinceList:[],
          cityList:[],
          areaList:[]  
      },
      map:{}



    }
  },
  computed: {

  },
  watch: {
  },
  created() {
    this.map = this._initData(mapList.districtList)
    this.getProvinceList()
    // this.mapData.cityList = this.map.cityList
    // this.mapData.areaList = this.map.areaList
  },
  mounted() {
  },
  methods: {
    /* 地图数据 */
    _initData(data){
      let list ={
          provinceList:[],
          cityList:[],
          areaList:[]
      }
      function fun(data){
          data.forEach(el => {
              const obj = {
                  adcode:el.adcode,
                  districtList:el.districtList,
                  name:el.name
              }
              switch(el.level){
                  case "province":
                      list.provinceList.push(obj)
                  break;
                  case "city":
                      list.cityList.push(obj)
                  break;
                  case "district":
                      list.areaList.push(obj)
                  break; 
              }
              if(el.districtList&&el.districtList.length > 0){
                  fun(el.districtList)
              }
          });
      }
      fun(data)
      list.provinceList.sort(this._sortNumber)
      list.cityList.sort(this._sortNumber)
      list.areaList.sort(this._sortNumber)
      return list
    },
    _sortNumber(a,b){
        return a.adcode - b.adcode
    },
    getProvinceList(){
        this.map.provinceList.sort(this._sortNumber)
        this.mapData.provinceList=this.map.provinceList
    },
    _getData(data,index){
        return this.map[data].filter(item=>item.adcode == index)[0].districtList
    },
    provinceChanged(value){
        this.dataFrom.city='',
        this.dataFrom.district='',
        this.mapData.areaList = ''
        this.mapData.cityList = this._getData('provinceList',value)
    },
    cityChanged(value){
        this.dataFrom.district='',
        this.mapData.areaList =this._getData('cityList',value)
    },
    areaChanged(value){
      console.log(this.dataFrom);
    },



    /* 照片上传 */
    uploadLogo(e,index){
        let file = e.target.files[0];
        let that = this;
        let isJPG = (file.type == 'image/jpeg' || file.type == 'image/png' || file.type == 'image/bmp' || file.type == 'image/gif');
        if (!isJPG) {
        this.$message.error('上传图片只能是 JPG/JPEG/PNG/GIF/BMP 格式!');
        return isJPG
        }
        let formData = new FormData()
        formData.append('file', file)
        uploadSchoolAPI(formData).then(res => {
            if(res.status == '200'){
                this.$message.success('上传图片成功')
                console.log(res.data);
                this._picture(res.data,index)
                if(this.validStatus){
                  this.$refs["dataForm"].validate(valid => {});
                }
            }else{
                this.$message.error('图片上传失败')
            }
        })
    },
    _picture(path,index){
      console.log(index);
        switch (index) {
            case '1':
                this.dataFrom.IDCardImgFront = path
                break;
            case '2':
                this.dataFrom.IDCardImgBack = path
                break;
            case '3':
                this.dataFrom.PhysicianImgOne = path
                break;    
            case'4':
                this.dataFrom.PhysicianImgTwo = path
                break;
            case '5':
                this.dataFrom.TittleImgOne = path
                break;
            case '6':
                this.dataFrom.TittleImgTwo = path
                break;
            default:
                break;
        }
    },
    // 查看大图
    lookImg(path){
        this.showImg = true
        this.imgPath = path
    },


    // 切换认证类型
    changeType(value){
      if(value == '0'){
        this.labelOne = '医师执业证第一页:',
        this.labelTwo = '医师执业证第二页:'
      }else if(value == '1'){
        this.labelOne = '教师或学生证:',
        this.labelTwo = '教师或学生证:'
      }
    },

    // 切换证件类型
    changeCard(value){
      console.log(12312435345343);
      if(value == '0'){
        this.labelOne = '医师执业证第一页:',
        this.labelTwo = '医师执业证第二页:'
      }else if(value == '1'){
        this.labelOne = '手持医院胸牌照:',
        this.labelTwo = '手持医院胸牌照:'
      }else if(value == '2'){
        this.labelOne = '从业证书:',
        this.labelTwo = '从业证书:'
      }
    },






    //保存
    dataSubmit() {
      console.log(this.dataFrom);
      this.$refs['dataFrom'].validate((valid) => {
        if (valid) {//必填字段验证
          saveAPI(this.dataFrom).then(res =>{
            console.log(res.data);
          })
        }
      })
    },
    cancel() {//取消
      this.$router.go(-1);
    },

  },
}
</script>

<style lang="less" scoped>
.upload-item{
  position: relative;
  .upLoad-img{
      width: 168px;
      height: 98px;
      position: absolute;
      left: 0;
      top: 0;
      opacity: 0;
  }
  .upLoad-img:hover{
      cursor: pointer;
  }
  .item-con{
      display: flex;
      flex-direction: column;
      text-align: center;
      width: 168px;
      height: 98px;
      background: #fff;
      border: 1px solid #DCDFE6;
      border-radius: 4px;
      position: absolute;
      left: 0;
      top: 0;
      img{
          margin: 20px auto 16px;;
          height: 30px;
          width: 30px;
      }
      .upload-text{
          line-height: 14px;
          color: #A9AFBC;
          font-size: 14px;
      }
  }
  
}
.showPic{
  position: relative;
  display: flex;
  width: 165px;
  height: 98px;
  .showImg{
      margin: auto;
      text-align: center;
      max-width: 165px;
      max-height: 98px;
  }
  .hover-img{
      position: absolute;
      top: 0;
      left: 0;
      width: 165px;
      height: 98px; 
      background: rgba(152, 153, 152, 0.5);
      display: flex;
      flex-direction: column;
      text-align: center;
      padding: 20px 0 ;
      .hover-img-text{
          line-height: 17px;
          padding: 5px 0;
          font-size: 14px;
          color:#fff;
          position: relative;
          .upload-again{
              position: relative;
          }
          .again-img{
              width: 168px;
              height: 17px;
              position: absolute;
              left: 0;
              top: 0;
              opacity: 0;
          }
          .again-img:hover{
              cursor: pointer;
          }
      }
      .hover-img-text:hover{
          cursor: pointer;
      }
      .hasimg{
          position: absolute;
          right: 0;
          top: 0;
      }
  }
  .showImg:hover{
      cursor: pointer;
  }
}
.img-center{
  height: 500px;
  width: 600px;
  text-align: center;
  display: flex;
  justify-content: center;
  .showBigImg{
    max-height: 500px;
    max-width: 600px;

  }
}
</style>
<style lang="less" >
  .form-line-three{
    .el-form-item{
        .el-form-item__content{
            .el-form-item__error{
                padding-top:65px;
            }
        }
    }
  }
</style>

