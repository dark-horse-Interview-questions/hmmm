<template>
  <el-card>
    <el-form style="margin-left:70px" label-width="120px" :model="formData" ref="formRef" :rules="rules">
      <el-form-item label="学科" prop="subjectID">
        <el-select v-model="formData.subjectID" style="width:450px">
          <el-option v-for="item in subjectIDList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="目录" prop="catalogID">
        <el-select v-model="formData.catalogID" style="width:450px">
          <el-option v-for="item in catalogIDList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="企业" prop="enterpriseID">
        <el-select v-model="formData.enterpriseID">
          <el-option v-for="item in enterpriseIDList" :key="item.id" :label="item.shortName" :value="item.id"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="城市" prop="province">
        <el-select v-model="formData.city" style="width:180px">
          <el-option v-for="(item,index) in citys" :key="index" :label="item" :value="item"></el-option>
        </el-select>

        <el-select v-model="formData.province" style="width:180px;margin-left:20px">
          <el-option v-for="(item,index) in provinces" :key="index" :label="item" :value="item"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="方向" prop="direction">
        <el-select v-model="formData.direction" style="width:450px">
          <el-option v-for="(item,index) in directionList" :key="index" :label="item" :value="item"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="题型">
        <el-radio-group v-model="formData.questionType">
          <el-radio v-for="item in questionTypeList" :key="item.value" :label="item.value + ''">{{item.label}}</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="难度">
        <el-radio-group v-model="formData.difficulty">
          <el-radio v-for="item in difficultyList" :key="item.value" :label="item.value + ''">{{item.label}}</el-radio>
        </el-radio-group>
      </el-form-item>

      <el-form-item label="题干" prop="question">
        <quill-editor class="quillEditor" v-model="formData.question">
          
        </quill-editor>
      </el-form-item>

      <el-form-item v-if="!shortAnswer" label="选项(以下选中的选项为正确答案)"></el-form-item>

<!-- 单选们 -->
      <el-form-item v-if="choiceAnswer">
        <el-row v-for="item in radioOptions" :key="item?item.id:null">
          <el-radio v-model="rightRadio" :label="item?item.code:null">{{item?item.code:null}}</el-radio>
          <el-input v-model="item.title" style="width:45%;margin-right:15px"></el-input>
          <el-button @click="delOptions(item.id)" style="font-size:12px;"><i class="el-icon-delete"></i><span>删除该选项</span></el-button>
        </el-row>
      </el-form-item>
      
      
      <!-- 多选们 -->
      <el-form-item v-if="choiceAnswers">
        <el-row v-for="item in checkboxOptions" :key="item?item.id:null">
          <el-checkbox v-model="item.isRight" :true-label="1" :false-label="0">{{item?item.code:null}}</el-checkbox>
          <el-input v-model="item.title" style="width:45%;margin-right:15px"></el-input>
          <el-button @click="delOptions(item.id)" style="font-size:12px;"><i class="el-icon-delete"></i><span>删除该选项</span></el-button>
        </el-row>
      </el-form-item>

      <el-form-item v-if="!shortAnswer">
        <el-button @click="addOptions" type="primary">增加选项与答案</el-button>
      </el-form-item>

      <el-form-item label="视频解析" prop="videoURL">
        <el-input v-model="formData.videoURL" style="width:450px">
          
        </el-input>
      </el-form-item>

      <el-form-item label="答案解析" prop="answer">
        <quill-editor class="quillEditor" v-model="formData.answer">
          
        </quill-editor>
      </el-form-item>

      <el-form-item label="题目备注" prop="remarks">
        <el-input v-model="formData.remarks" style="width:450px">
          
        </el-input>
      </el-form-item>

      <el-form-item label="试题标签" prop="tags">
        <quill-editor class="quillEditor" v-model="formData.tags">
          
        </quill-editor>
      </el-form-item>

      <el-form-item>
        <el-row type="flex" justify="center" style="margin-right:190px">
          <el-button @click="submitBtn" type="primary">提交</el-button>
          <el-button @click="resetForm">重置</el-button>
        </el-row>
      </el-form-item>
      
    </el-form>

<!-- <div style="display:none"> -->
  {{clickNum}}
  {{clickNums}}
  {{formData.province}}
<!-- </div> -->

  </el-card>
</template>

<script>
import { quillEditor } from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
//引入富文本
import { simple as subjectsSimple } from '@/api/hmmm/subjects'
import { simple as directorysSimple } from '@/api/hmmm/directorys'
import { simple as tagsSimple } from '@/api/hmmm/tags'
import { list as enterpriseIDList } from '@/api/hmmm/companys'
import { provinces as getAllCity,citys as getAllRegion } from '@/api/hmmm/citys'
import { direction, questionType, difficulty } from '@/api/hmmm/constants'
import { add, detail, update } from '@/api/hmmm/questions'
export default {
  name: 'QuestionsNew',
  data() {
    return {
      test:'',
      formData:{
        id:null,
        subjectID:null,                     //学科
        catalogID:null,                     //目录
        enterpriseID:null,                  //企业
        city:'',                          //城市
        province:'',                      //区域
        direction:'',                     //方向
        questionType:'1',                  //题型
        difficulty:'1',                    //难度
        question:'',                      //题干
        options:[],
        videoURL:'',                      //解析视频
        answer:'',                        //答案解析
        remarks:'',                       //题目备注
        tags:'',                          //试题标签
        isPerfect:false                   //是否精选题
      },
      rules:{
        subjectID:[{required:true,message:'学科不可为空'}],
        catalogID:[{required:true,message:'目录不可为空'}],
        enterpriseID:[{required:true,message:'企业不可为空'}],
        province:[{required:true,message:'城市和区域不可为空'}],
        direction:[{required:true,message:'方向不可为空'}],
        question:[{required:true,message:'题干不可为空'}],
        videoURL:[{required:true,message:'视频解析不可为空'}],
        answer:[{required:true,message:'答案解析不可为空'}],
        remarks:[{required:true,message:'题目备注不可为空'}],
        tags:[{required:true,message:'试题标签不可为空'}],
      },
      subjectIDList:[],                   //学科列表
      catalogIDList:[],                    //目录列表
      enterpriseIDList:[],                 ///企业列表
      citys:[],                             //城市列表
      provinces:[],                          //区域列表
      directionList:direction,              //方向列表
      questionTypeList:questionType,        //题型列表
      difficultyList:difficulty,             //难度列表
      options:['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z',],
      clickNum:0,                            //单选点击次数
      clickNums:0,                            //多选点击次数
      rightRadio:'',                         //正确单选
      rightCheckbox:'',                      //正确多选
      choiceAnswer:true,                    //单选类型
      choiceAnswers:true,                    //多选类型
      shortAnswer:false,                     //简答类型
      radioOptions:[],                      //临时单选组
      checkboxOptions:[],                   //临时多选组
    }
  },
  methods:{
    // checkboxChange(id){
    //   this.checkboxOptions.forEach(item => {
    //     if(item.id === id){
    //       item.isRight = !item.isRight
    //     }
    //   })
    // },
    async submitBtn(){
      if(this.formData.questionType === '1'){
        this.formData.options = this.radioOptions
      }else if(this.formData.questionType === '2'){
        this.formData.options = this.checkboxOptions
      }else if(this.formData.questionType === '3'){
        this.formData.options = []
      }                                             //处理单选多选
      if(this.$route.query.id){     //是修改
        await update(this.formData)
        this.$message({
          type:'success',
          message:'试题修改成功'
        })
        this.$router.go(-1)
      }else{                        //是新增
        await this.$refs.formRef.validate()
        await add(this.formData)
        this.$message({
          type:'success',
          message:'试题录入成功'
        })
        this.$router.push('/questions/list')
      }

    },
    addOptions(){
      if(this.clickNum === this.options.length){
        return
      }
      if(this.formData.questionType === '1'){
        this.clickNum++
      }else if(this.formData.questionType === '2'){
        this.clickNums++
      }
    },
    delOptions(id){
      if(this.formData.questionType === '1'){
        if(this.radioOptions.length === 1){
          this.radioOptions = []
          return
        }
        this.radioOptions.forEach((item,index) => {
          if(item.id === id){
            this.radioOptions.splice(index,1)
          }
        })
      }else if(this.formData.questionType === '2'){
        if(this.checkboxOptions.length === 1){
          this.checkboxOptions = []
          return
        }
        this.checkboxOptions.forEach((item,index) => {
          if(item.id === id){
            this.checkboxOptions.splice(index,1)
          }
        })
      }
    },
    async getSubjectIDList(){
      let result = await subjectsSimple()
      this.subjectIDList = result.data
    },
    async getCatalogIDList(){
      let result = await directorysSimple()
      this.catalogIDList = result.data
    },
    async getEnterpriseIDList(){
      let result = await enterpriseIDList()
      this.enterpriseIDList = result.data.items
    },
    async getDetailByID(){
      let result = await detail({ id:this.$route.query.id})
      this.formData = result.data
      if(this.formData.questionType === '1'){
        this.radioOptions = this.formData.options
      }else if(this.formData.questionType === '2'){
        this.checkboxOptions = this.formData.options
      }
    },
    gitAlcitys(){
      this.citys = getAllCity()
    },
      resetForm() {
        this.$refs.formRef.resetFields();
      }
  },
  watch:{
    $route (to,from) {
      this.formData = {
        // number:100,                        //试题编号
        subjectID:null,                     //学科
        catalogID:null,                     //目录
        enterpriseID:null,                  //企业
        city:'',                          //城市
        province:'',                      //区域
        direction:'',                     //方向
        questionType:'1',                  //题型
        difficulty:'1',                    //难度
        question:'',                      //题干
        options:[],
        videoURL:'',                      //解析视频
        answer:'',                        //答案解析
        remarks:'',                       //题目备注
        tags:'',                          //试题标签
        isPerfect:false                   //是否精选题
      }
    },
    'formData.city'(newVal){
    // this.formData.province = '请选择'
    
    this.provinces = getAllRegion(newVal)
    },
    clickNum(newVal){
      // this.formData.options.length = newVal
      if(newVal){
        // this.radioOptions[this.radioOptions.length] = {code:this.options[newVal - 1],title:'',img:'',isRight:false,id:newVal}
        this.radioOptions.push({code:this.options[this.radioOptions.length],title:'',img:'',isRight:false,id:newVal})
      }
    },
    clickNums(newVal){
      // this.formData.options.length = newVal
      if(newVal){
        // this.checkboxOptions[this.checkboxOptions.length] = {code:this.options[newVal - 1],title:'',img:'',isRight:false,id:newVal}
        this.checkboxOptions.push({code:this.options[this.checkboxOptions.length],title:'',img:'',isRight:false,id:newVal})
      }
    },
    'formData.questionType'(newVal){
      if(newVal === '1'){
        console.log('是单选')
        this.choiceAnswer = true
        this.choiceAnswers = false
        this.shortAnswer = false
      }else if(newVal === '2'){
        console.log('是多选')
        this.choiceAnswers = true
        this.choiceAnswer = false
        this.shortAnswer = false
      }else if(newVal === '3'){
        this.shortAnswer = true
        this.choiceAnswer = false
        this.choiceAnswers = false
      }
    },
    rightRadio(newVal){
      this.radioOptions.forEach(item => {
        if(item.code === newVal){
          item.isRight = true
        }else{
          item.isRight = false
        }
      })
    }
  },
  mounted(){
    this.getSubjectIDList()
    this.getCatalogIDList()
    this.getEnterpriseIDList()
    this.gitAlcitys()
    if(this.$route.query.id){
      this.getDetailByID()
    }
  },
  components:{
    'quill-editor':quillEditor
  }
}
</script>

<style scoped>
.quillEditor{
  width:700px;
  height:300px;
  margin-bottom:100px;
}
</style>
