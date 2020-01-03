<template>
  <!-- <div class="question-container"> -->
    <el-card>
        <el-row slot="header">
          <el-button type="primary"  @click="$router.push('/questions/new')">新增试题</el-button>
          <el-button type="primary">批量导入</el-button>
        </el-row>
        <div class="el-card_body" style="margin-top:20px">
          <el-form :model="formData" ref="myForm" inline label-width="80px">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">学科</label>
              <el-select v-model="formData.subjectID">
                <el-option v-for="item in subjects" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            
            <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">难度</label>
              <el-select v-model="formData.difficulty">
                <el-option v-for="item in difficulty" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            </div>
            <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">试题类型</label>
              <el-select v-model="formData.questionType">
                <el-option v-for="item in questionType" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            </div>
            <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">标签</label>
              <el-select v-model="formData.tags">
                <el-option v-for="item in tags" :key="item.value" :label="item.label" :value="item.value"> </el-option>
              </el-select>
            </div>
            <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">城市</label>
              <el-select v-model="formData.province">
                <el-option v-for="(item,index) in provinces" :key="index" :label="item" :value="item"> </el-option>
              </el-select>
            </div>
            <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 80px;">区域</label>
              <el-select v-model="formData.city">
                <el-option v-for="(item,index) in myCitys" :key="index" :label="item" :value="item"></el-option>
              </el-select>
            </div>
            <div class="el-form-item">
              <label for="keywod" class="el-form-item__label" style="width: 100px;">关键字</label>
                <input type="text" autocomplete="off" class="el-input_inner">
            </div>
             <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 100px;">学科备注</label>
               <input type="text" autocomplete="off" class="el-input_inner" style="border: 1px solid #EBEEF5;">
            </div>
             <div class="el-form-item">
              <label for="subjectID" class="el-form-item__label" style="width: 100px;">企业简称</label>
               <input type="text" autocomplete="off" class="el-input_inner">
            </div>
              <div class="el-form-item">
                <label for="subjectID" class="el-form-item__label" style="width: 80px;">方向</label>
                <el-select v-model="formData.direction">
                  <el-option v-for="(item,index) in direction" :key="index" :value="item" :label="item"> </el-option>
                </el-select>
              </div>
              <div class="el-form-item">
                <label for="subjectID" class="el-form-item__label" style="width: 80px;">录入人</label>
                <el-select v-model="formData.creatorID">
                  <el-option v-for="(item) in users" :key="item.id" :value="item.id" :label="item.username"></el-option>
                </el-select>
              </div>
              <div class="el-form-item">
                <label for="subjectID" class="el-form-item__label" style="width: 80px;">二级目录</label>
                <el-select v-model="formData.catalogID">
                  <el-option v-for="(item) in directioy" :key="item.id" :value="item.value" :label="item.label"></el-option>
                </el-select>
              </div>
            </el-form>
              <el-row type="flex" justify="center" style="margin: 20px 0">
                 <el-button @click="clear">清除</el-button>
                 <el-button @click="search" type="primary">搜索</el-button>
              </el-row>
            
            <el-table :data="list" width="100%"> 
              <el-table-column prop="id" label="序号" width="70px"></el-table-column>
              <el-table-column prop="number" label="试题编号"></el-table-column>
              <el-table-column prop="subjectID" label="学科" :formatter="subjectFormatter" ></el-table-column>
              <el-table-column prop="questionType" label="题型" :formatter="questionFormatter"></el-table-column>
              <el-table-column prop="question" label="题干"></el-table-column>
              <el-table-column prop="addDate" label="录入时间" width="180px">
                <template slot-scope="obj">{{ obj.row.addDate | parseTimeByString}}</template>
              </el-table-column>
              <el-table-column prop="difficulty" label="难度" :formatter="diffocultyFormatter"></el-table-column>
              <el-table-column prop="creator" label="录取人"></el-table-column>
              <el-table-column prop="operation" label="操作" width="180px">
                <template slot-scope="obj">
                   <!-- 作用域插槽 -->
                  <el-button type="text" @click="previewId(obj.row.id)" size="small">预览</el-button>
                  <el-button type="text" @click="modify(obj.row.id)" size="small" >修改</el-button>
                  <el-button type="text" @click="delItem(obj.row.id)" size="small">删除</el-button>
                  <el-button type="text" @click="joinItem(obj.row.id)" size="small">加入精选</el-button>
                </template>
              </el-table-column>
            </el-table>
          <!-- 分页 -->
            <el-row type='flex' justify="end" align="middle" style="height:60px">
                <el-pagination  background layout="prev, pager, next" 
                :total="page.total" 
                :current-page="page.currentPage" 
                :page-size="page.pageSize" 
                @current-change="changePage">
                </el-pagination>
            </el-row> 
          <!-- 分页结束 -->
          <!-- 弹窗部分 el-dialog -->
          <el-dialog :visible="visible" @close="closePreview" title="题目预览">
            <el-row style="height:40px" type="flex" align="middler">
              <el-col :span="6">[题库]:面试宝典题库</el-col>
              <el-col :span="6">[学科]:{{ computedSubject }}</el-col>
            </el-row>
            <el-row style="height:40px" type="flex" align="middler">
              <el-col :span="6">[题型]: {{ computedQuestionType }}</el-col>
              <el-col :span="6">[编号]: {{ formData.number }}</el-col>
              <el-col :span="6">[难度]: {{ computedDifficulty }}</el-col>
              <el-col :span="6">[标签]: {{ formData.tags }}</el-col>
            </el-row>
            <el-row style="height:40px" type="flex" align="middler">
              <el-col :span="6">[目录]: {{ computedDirectory }}</el-col>
              <el-col :span="6">[企业]: {{ formData.enterpriseID }}</el-col>
              <el-col :span="6">[方向]: {{ formData.direction }}</el-col>
              <el-col :span="6">[城市]: {{ formData.province + formData.city }}</el-col>
            </el-row>
            <el-row></el-row>
            <el-row slot="footer" type="flex" justify="end">
              <el-button type="primary" @click="closePreview">关闭</el-button>
            </el-row>
            <el-row v-if="formData.questionType == '1'">
              <el-radio disabled v-model="item.isRight" :label="1" v-for="(item,index) in formData.options" :key="index">{{ item.title }}</el-radio>
            </el-row>
            <el-row v-else-if="formData.questionType == '2'">
              <el-checkbox disabled v-model="item.isRight" :true-label="1" :false-label="0" v-for="(item,index) in formData.options" :key="index">{{ item.title }}</el-checkbox>
            </el-row>
        </el-dialog>

        </div>
    </el-card>       

</template>

<script>
    // import {list as questionList,detail,remove} from '../../api/hmmm/questions';
    import {list,detail,remove} from '../../api/hmmm/questions';
    import {simple as subjectList} from '../../api/hmmm/subjects';
    import {questionType,difficulty,direction} from'../../api/hmmm/constants';
    import { simple as UserList } from "../../api/base/users";
    import { simple as directioy } from "../../api/hmmm/directorys";
    import { simple as TagList } from "../../api/hmmm/tags";
    import { citys, provinces } from "../../api/hmmm/citys";
    export default {
    data() {
    return{
      visible:false,
      list: [],
      subjects: [],
      tableData: [],
      citys:[],
      provinces: provinces(),
      tags:[],
      directioy:[],
      users:[],
      questionType,
      difficulty,
      direction,
     
      formData: {
        difficulty: null,
        questionType: null,
        tags: null,
        subjectID: null,
        province: null,
        city: null,
        keyword: null,
        remarks: null,
        shortName: null,
        direction: null,
        creatorID: null,
        catalogID: null,
        tableData:null
      },
      page: {
        currentPage: 1,
        pageSize: 10, // 黑马头条后端限制 最低10条 => 文章列表
        total: 0
      }
    };
    },
    computed: {
    myCitys() {
      return citys(this.formData.province);
    },
    computedQuestionType() {
      let result =
        this.questionType &&
        this.questionType.filter(
          item => item.value == this.formData.questionType
        );
      return result.length ? result[0].label : "未知";
    },
    computedDifficulty() {
      let result =
        this.difficulty &&
        this.difficulty.filter(item => item.value == this.formData.difficulty);
      return result && result.length ? result[0].label : "未知";
    },
    computedSubject(value) {
      let result =
        this.subjects &&
        this.subjects.filter(item => item.value == this.formData.subjectID);
      return result && result.length ? result[0].label : "未知";
    },
    computedDirectory(value) {
      let result =
        this.directory &&
        this.directory.filter(item => item.value == this.formData.catalogID);
      return result && result.length ? result[0].label : "未知";
    }
  },
    methods:{
    closePreview() {
      this.formData = {
        difficulty: null,
        questionType: null,
        tags: null,
        subjectID: null,
        province: null,
        city: null,
        keyword: null,
        remarks: null,
        shortName: null,
        direction: null,
        creatorID: null,
        catalogID: null
      };
      this.visible = false;
    },
    // 预览内容
    async previewId(id) {
      let result = await detail({ id });
      this.formData = result.data;
      this.visible = true;
    },
    modify(id) {
      this.$router.push({ path: "/questions/new", query: { id } });
    },
    // 删除文章
    async delItem(id) {
      await this.$confirm("是否删除此文章");
      await remove({ id });
      this.$message({ type: "success", message: "删除成功" });
      this.getCondition();
    },
    async joinItem(id) {
      await this.$confirm("是否加入精选");
    },
    changePage(newPage) {
      this.page.currentPage = newPage;
      this.getCondition();
    },
    getCondition() {
      let params = {
        page: this.page.currentPage,
        pageSize: this.page.pageSize,
        ...this.formData
      };
      this.getQuestion(params);
    },
    questionFormatter(row, column, cellValue) {
      let result = this.questionType.filter(item => item.value == cellValue);
      return result.length ? result[0].label : "未知";
    },
    subjectFormatter(row, column, cellValue) {
      let result = this.subjects.filter(item => item.value == cellValue);
      return result.length ? result[0].label : "未知";
    },
    diffocultyFormatter(row, column, cellValue) {
      let result = this.difficulty.filter(item => item.value == cellValue);
      return result.length ? result[0].label : "未知";
    },
    search() {
    
    },
    clear() {
      this.closePreview();
    },



    async getSubject() {
      let result=await  subjectList();
      this.subjects = result.data;
    },
     async getQuestion(data) {
      let result = await list(data);
      this.list = result.data.items;
      this.page.total = result.data.counts;
    },
    async getTags() {
      let result = await TagList();
      this.tags = result.data;
    },
    async getDetail(){
      await detail({id: 1});
    },
    async getArticles(){
     let result =  await list();
    this.tableData = result.data.items; //这里有问题
    },
    async getUserList() {
      let result = await UserList();
      this.users = result.data;
    },
    async getDirectioy() {
      let result = await directioy();
      this.directioy = result.data;
    } 
    },
    created(){
    this.getArticles();
    this.getSubject();
    this.getTags();
    this.getDirectioy();
    this.getUserList();
    this.getQuestion();
    }
  };
</script>

<style lang="scss" scoped>
  .el-card_body{
    input{
      border:1px solid #ebeef5;
      height: 34px;
      // width: 80px;
    }
    
  }
</style>
