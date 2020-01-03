<template>
  <!-- <div class="dashboard-container">
    <div class="app-container">
      面试技巧
    <el-button @click="$router.push({name:'articles-add'})">新增技巧</el-button>
    </div>
  </div> -->
  <el-card v-loading='loading'>
    <!-- 头部新增按钮开始 -->
    <el-row slot="header">
      <el-button type="primary" @click="dialogVisible = true">新增面试技巧</el-button>
    </el-row>
    
    <!-- 新增面试文章dialog弹框 -->
    <el-dialog
    title="提示"
    :visible.sync="dialogVisible"
    width="50%">

    <el-form label-width="100px" :model="formData" :rules="addArticlesRules" ref="myArticles">
      <el-form-item label="学校名称" prop="title">

        <el-input v-model="formData.title"></el-input>

      </el-form-item>
<!-- --------------------------------------------------------------------------------- -->
      <el-form-item label="内容" prop="articleBody">

        <quill-editor v-model="formData.articleBody" style="height:300px;margin-bottom:150px"></quill-editor>

      </el-form-item>
<!-- ---------------------------------------------------------------------------------- -->
      <el-form-item label="视频地址" prop="videoURL">

        <el-input v-model="formData.videoURL"></el-input>

      </el-form-item>
<!-- -------------------------------------------------------------------------------->
    </el-form>
    <el-row slot="footer" class="dialog-footer" type='flex' justify="end" align="middle" >
      <el-button @click="cancelDialog">取 消</el-button>
      <el-button type="primary" @click="addArticles">确 定</el-button>
    </el-row>
</el-dialog>



<!-- ---------------------------------------------------------------------- -->
    <!-- 搜索框开始 -->
    <el-form class="demo-ruleForm">
      <el-form-item label="关键字">
        <el-input style="width:25%" placeholder="请输入关键字" v-model="KeyWords.title"></el-input>
        <el-button style="margin-left:10px" @click="clearKeyWords">清除</el-button>
        <el-button type = 'primary' @click="search">搜索</el-button>
      </el-form-item>
    </el-form>
<!-- ------------------------------------------------------------------------- -->
    <!-- 主体区域显示查询修改删除列表开始 -->
    <!-- :data="tableData"  这差点数据绑定 -->
     <el-table style="width: 100%" :data="list">  

      <el-table-column 
        prop="id"
        label="序号"
        width="180">
      </el-table-column>
      <!-- -------------------------------------------------------------------- -->
      <el-table-column
        prop="title"
        label="标题"
        width="180">
      </el-table-column>
      <!-- --------------------------------------------------------------------- -->
      <el-table-column
        prop="reads"
        label="阅读数">
      </el-table-column>
      <!-- ---------------------------------------------------------------------- -->
      <el-table-column
        prop="state"
        label="状态"
        :formatter="formatterBoolean">
      </el-table-column>
      <!-- ----------------------------------------------------------------------- -->
      <el-table-column
        prop="creator"
        label="录入人">
      </el-table-column>
      <!-- ------------------------------------------------------------------------ -->
      <el-table-column
        prop="address"
        label="操作">
        <!-- 操作按钮 -->
        <template slot-scope="obj">
          <el-button size="small" type="text">预览</el-button>
          <el-button size="small" type="text" @click="changeArticles(obj.row)">修改</el-button>

          <el-button size="small" type="text" @click="delArticles(obj.row.id)">删除</el-button>
          
          <el-button size="small" type="text" @click="changeArticlesState(obj.row)">{{obj.row.state === 1 ? '禁用' : '启用'}}</el-button>
        </template>

      </el-table-column>
      <!-- -------------------------------------------------------------------------- -->
    </el-table>
<!-- --------------------------------------------------------------------------------- -->
    <!-- 定义底部的分页组件 --> 
    <el-row style="margin-top:10px" type="flex" justify="end" align="middle">
        <span>共{{page.total}}条</span>
       <el-pagination  background
  layout="prev, pager, next"
  :total="page.total" :current-page="page.currentPage" :page-size="page.pageSize" @current-change = 'changePage'>
        </el-pagination>
    </el-row>
  </el-card>

</template>

<script>
import { list , remove , state , add , update , detail} from '../../api/hmmm/articles'   //引入接口文件  并解构
// import { log } from 'util'
export default {
  // name: 'ArticlesList',
  data() {
    return {
      dialogVisible:false,  //diaflog默认关闭

      loading:false,   //默认loading是关闭的

      list:[],  //用于接收文章的返回数据
      
      KeyWords:{  //搜索关键字
        title:''
      }, 

      page:{
        total:0,  //总记录数
        currentPage:1,  //当前页码
        pageSize: 5, //每页显示条目个数 
      },

      // 添加的表单验证
      formData:{
        title:'',  //学校名称
        articleBody:'',  //文章内容
        videoURL:'',  //视频地址
      },
      //自动校验规则
      addArticlesRules:{
        title:[{required:true,message:'标题不能为空',trigger : 'blur'}],
        articleBody:[{required:true,message:'文章内容不能为空',trigger : 'blur'}],
        videoURL:[{required:true,message:'标题地址不能为空',trigger : 'blur'}]
      }

    }
  },
  methods:{

    //搜索文章
    search(){

      this.currentPage = 1  //新页数每次为一

      this.getArticles(this.KeyWords)
      // this.clearKeyWords()
    },
    //修改文章方法
    async changeArticles(id){

        this.dialogVisible = true

        let res = await detail({id : id.id})

        this.formData = res.data

      },
      //点击确定完成手动校验  通过后添加文章
    async addArticles(){

      await this.$refs.myArticles.validate();

      this.formData.id ? await update(this.formData) : await add(this.formData)

      this.cancelDialog()   //清空信息

      this.getArticles()
      },

      //点击取消关闭弹层  清空内容
      cancelDialog(){
        this.formData = {
          title:'',  //学校名称
          articleBody:'',  //文章内容
          videoURL:'',  //视频地址
        }
        this.dialogVisible = false
      },

      // 点击分页获取数据
      changePage(newPage){
          this.page.currentPage = newPage
          this.getArticles()
      },
      clearKeyWords(){   //删除关键字
        this.KeyWords = {
          title : ''
        }
      },

    async getArticles(data){  //获取文章列表
      let res = await list({
        page:this.page.currentPage,
        pagesize:this.page.pageSize,
        ...data
      })
      this.list = res.data.items  //获取表格数据
      this.page.total = res.data.counts //获取总记录数
      },

      // 定义一个格式化的函数用于处理返回状态布尔值
      formatterBoolean(row, column, cellValue, index){
          return cellValue ? '启用' : '禁用'
      },
      //改变文章状态
    async changeArticlesState(row){
        
      await this.$confirm('确定进行此操作吗')

      await state({id:row.id, state: row.state === 1 ? 0 : 1 })

      this.getArticles()

      },
      // 删除文章的方法
      delArticles(id){
      // console.log(id);
       this.$confirm('确认要删除吗').then(()=>{
          remove({id}).then(res=>{
            this.$message({
            type:'success',
            message:'删除成功',
          })
           this.getArticles()   //重新请求数据
          })
       })
          
      }
  },
  //钩子函数调用
  created(){
      this.getArticles()
  }
}
</script>

<style scoped>

</style>
