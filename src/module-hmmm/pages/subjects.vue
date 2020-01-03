<template>
<el-card class="dashboard-container">
  <!--头部按钮-->
  <div slot="header">    
    <el-button type="primary" @click="dialogVisible= true">新增学科</el-button>      
  </div>
  <!--el-form表单-->
  <el-form label-width="100px">
    <el-form-item label="学科名称" >
      <el-input style="width:20%" v-model="searchForm.subjectName"></el-input>
      <el-button style="margin-left:10px" @click="clear">清除</el-button>
       <el-button type="primary" @click="search">搜索</el-button>
    </el-form-item>
      
  </el-form>

  <!--table表格-->
     <!-- {{list}} -->
  <el-table
      :data="list"
      style="width: 100%">
      <el-table-column
        prop="id"
        label="序号"
        width="120">
      </el-table-column>
      <el-table-column
        prop="subjectName"
        label="学科名称"                                                                                                                                                                                                            
        width="100">
      </el-table-column>
      <el-table-column
        prop="creator"
        label="创建者"
        width="90">
      </el-table-column>
      <el-table-column
        prop="addDate"
        :formatter="dateFormat"
        label="创建日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="isFrontDisplay"
        :formatter="formatterBoolean"
        label="前台是否显示"
        width="80">
      </el-table-column>
      <el-table-column
        prop="twoLevelDirectory"
        label="二级目录"
        width="100">
      </el-table-column>
      <el-table-column
        prop="tags"
        label="标签"
        width="100">
      </el-table-column>
      <el-table-column
        prop="totals"
        label="题目数量"
        width="100">
      </el-table-column>
      <el-table-column
        label="操作">
        <template slot-scope="scope">
         <el-button size="small" type="text">学科分类</el-button>
         <el-button size="small" type="text">学科标签</el-button>
         <el-button size="small" type="text" @click="changeSubject(scope.row.id)">修改</el-button>
         <el-button size="small" type="text" @click="delSubject(scope.row)">删除</el-button>
        </template>
      </el-table-column>
      {{list}}
    </el-table>

    <!--分页组件-->
    <el-row type="flex" justify="end" align="middle">
      <span style="font-size:14px; color: #606266;">共{{page.counts}}条</span>
      <el-pagination
        background
        layout="prev, pager, next"
        :current-page="page.currentPage"
        @current-change="changePage"
        :page-size="page.pagesize"
        :total="page.counts">
      </el-pagination>
    </el-row>

    <!--新增Dialog-->
    <el-dialog
  :visible.sync="dialogVisible"
  :show-close=false
  width="50%">
  <el-form :model="formData" :rules="rules" ref="ruleForm" label-width="100px" style="margin-left:25px">
    <el-form-item label="学科姓名" prop="subjectName">
      <el-input v-model="formData.subjectName"></el-input>
    </el-form-item>

    <el-form-item label="是否前台显示" prop="isFrontDisplay">
    <el-switch v-model="formData.isFrontDisplay"></el-switch>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="cancelVisible">取 消</el-button>
    <el-button type="primary" @click="addVisible">确 定</el-button>
  </span>
</el-dialog>

</el-card>
  
</template>

<script>
import {list,remove,add,update,detail} from '../../api/hmmm/subjects'
import { createDiffieHellman } from 'crypto';
export default {
  data() {
    return {
      //搜索对象
      searchForm:{
        subjectName:'',   //学科名称
      },
      dialogVisible: false,      
      list:[], //定义一个数据接收返回结果
      //定义page分页对象
      page:{
        counts:0, //定义总页数
        pagesize:10, //每页数量
        currentPage: 1 // 当前页码
      },
      // 定义formData
      formData:{
        subjectName:'',   //学科名称
        isFrontDisplay:true  //前台是否显示
      },
      //验证规则
      rules:{
        subjectName:[
          {required: true, message: '学科名称不能为空', trigger: 'blur'}
          ]
      }
    }
  },
  methods:{
    //搜索功能
    search(){
      this.page.currentPage=1
      this.getSubjectList(this.searchForm)

    },
    //取消新增
    cancelVisible(){
      this.dialogVisible=false
      this.formData={
        subjectName:'',   //学科名称
        isFrontDisplay:true,  //前台是否显示
      }
    },
    //清除内容
    clear(){
      this.searchForm={
        subjectName:'',   //学科名称
        isFrontDisplay:true,  //前台是否显示
      }
    },
    //点击新增学科
    async addVisible() {
      await this.$refs.ruleForm.validate()
      alert(this.formData.id)
      this.formData.id ? await update(this.formData) : await add(this.formData)
        this.dialogVisible=false
          this.formData={
            subjectName:'',   //学科名称
            isFrontDisplay:true,  //前台是否显示
          }
          this.$message({
            type: 'success',
            message: '操作成功'
          })
          this.getSubjectList()
      },
      //点击修改学科
      async changeSubject(id){
        this.dialogVisible=true
        let res=await detail({id})
        // this.formData.subjectName=res.data.subjectName
        // this.formData.isFrontDisplay=res.data.isFrontDisplay?true:false
        this.formData=res.data

      },


      //点击删除学科
      delSubject(id) {
        this.$confirm('确定删除此目录吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          remove({id:id.id}).then(res=>{
            this.$message({
            type: 'success',
            message: '删除成功!'
          });
          this.getSubjectList()
          })
         
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        });
        },

    //改变页码
    changePage(newPage){
      this.page.currentPage=newPage
      //再次获取数据
      this.getSubjectList()
    },
    //获取学科详情
    getSubjectList(data){
      list({
        page: this.page.currentPage,
        pagesize: this.page.pageSize,
        ...data
      }).then(res=>{
        this.list=res.data.items
        this.page.counts = res.data.counts
      })
    },
    //布尔值的判断
    formatterBoolean(row, column, cellValue, index){
      return cellValue?'显示':'不显示'
    },

    //时间的处理
    dateFormat:function(row,column){
         var date = row[column.property]; 
         if (date == undefined) { 
           return ""; 
        } 
        var moment = require("moment");
         return moment(date).format("YYYY-MM-DD HH:mm:ss"); 
   },
  },
    created(){
      this.getSubjectList()
  }
}
</script>

<style lang="less" scoped>



</style>
