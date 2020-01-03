<template>
  <el-card slot="header">
    <!-- {{list}} -->
 
    <el-form>
        <!-- 新增标签 -->
        <el-form-item style="border-bottom:1px solid #ccc">
         <el-button type="primary" @click="dialogVisible = true"> 新增标签</el-button>
          <el-button type="primary" > 返回学科</el-button>
        </el-form-item>
        <!-- 标签名称 -->
        <el-form-item label="标签名称" type="flex">
         <el-input style="width:20% ;margin-right:10px" v-model="seachForm.tagName"></el-input>
       
          <el-button @click="clear"> 清除</el-button>
           <el-button type="primary" @click="seach"> 搜索</el-button>
        </el-form-item>
      </el-form>
    <!-- 表格部分 -->
    <el-table :data="list">
      <el-table-column  prop="id"  label="序号"></el-table-column>
      <el-table-column  prop="tagName"  label="标签名称"></el-table-column>
      <el-table-column  prop="creator"  label="创建者"></el-table-column>
      <el-table-column  prop="totals" label="面试数量"></el-table-column>
      <el-table-column :formatter="formatterBoolean" prop="state" label="状态"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="obj">
         
          <!-- 作用域插槽 -->
          <el-button type="text" @click="modify(obj.row.id)">修改</el-button>
            <el-dialog :visible="dialogVisible" :show-close="false">
              <el-form ref="myForm" :model="formData" :rules="rules">
                <el-form-item label="学科名称" prop="tagName">
                  <el-input v-model="formData.tagName" style="width:50%"></el-input>
                </el-form-item>
                <el-form-item label="学科" prop="subjectID">
                  <el-select v-model="formData.subjectID" value="">
                    <el-option v-for="item in subjects" :key="item.value" :label="item.label" :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
          
              </el-form>
                  <el-row slot="footer" type="flex" justify="end" style="magin:30px 0">
                    <el-button type="primary" size="small" @click="submitForm">确定</el-button>
                    <el-button size="small" @click="btnCancel">取消</el-button>
                  </el-row> 

            </el-dialog>

        <!-- 需要根据状态判断是禁用还是启用 -->
        <el-button @click="EnableOrDisable(obj.row)" type="text">{{obj.row.state ? '禁用' : '启用'}}</el-button>
        <el-button @click="removeTags(obj.row.id)" type="text">删除</el-button>
        </template>
        
      </el-table-column>

    </el-table>  
    <el-row type="flex" justify="center" align="middle">
      <el-pagination
        background
        layout="prev, pager, next"
        :total="page.total"
        :page-size="page.pageSize"
        :current-page="page.currentPage"
        @current-change="changepage"
       >  
    </el-pagination>
    </el-row>
    
  </el-card>
</template>

<script>
import {list as getlist} from "../../api/hmmm/tags"
 import {removeState,remove,add,update,detail} from "../../api/hmmm/tags"
 import {simple as getSubjects} from "../../api/hmmm/subjects"
export default {
  // name: 'TagsList',
  data() {
    return {
      seachForm:{
        tagName:''
      },
      formData:{
        subjectID:'',
        tagName:'',
      },
      rules:{
       subjectID:[{required:true,message:'标签名不能为空',trigger:'blur'}] ,
        tagName:[{required:true,message:'标签名不能为空',trigger:'blur'}]
      },
      dialogVisible:false,
      list:[],
      simpleList:[],
      page:{
        total:0,
        pageSize:10,
        currentPage:1
      },
      subjects:[]
     
    }
  },
  methods:{
    //搜索
  seach(){
    this.currentPage=1
     this.gettagslist(this.seachForm)

  },

    //清除输入框数据
    clear(){
     this.seachForm = ''
    },
     
    async submitForm(){
        // 表单校验提交
        //获取表单数据
      await this.$refs.myForm.validate()
      //调用新增接口
      this.formData.id
      ? await update(this.formData)//如果formData.id存在调用修改接口
       :await add(this.formData)//如果formData.id不存在调用新增接口
        this.$message({
          type:'success',
          message:'操作成功',
        }) 
        this.gettagslist()
        this.btnCancel()
      },
       btnCancel(){
         this.formData = {
          subjectID:'',
          tagName:'',
      }
       //关闭弹层  
       this.dialogVisible = false

      },
    
 async getSubjects() {
    let result = await getSubjects()
    this.subjects = result.data
  },
    //点击修改功能
    //打开弹层
   async modify(id){
    let result = await detail({id})
    this.formData = result.data
     this.dialogVisible = true
    },
  //  openDiaolog(id){
  //     this.dialogVisible = true
  //   },
    // 点击删除功能
    removeTags(id){
      //友好提示
      this.$confirm('您确定要删除此标签吗？').then(()=>{
        //调用接口方法
      remove({id}).then(result=>{
        this.$message({
          type:'success',
          message:'删除成功'
        }) 
      })
      this.gettagslist()
    })
      
    },
    // 分页方法
    changepage(newPage){
      this.page.currentPage = newPage
       this.gettagslist()
    },

    async gettagslist(data) {
      let result= await getlist( {page: this.page.currentPage,
        per_page: this.page.pageSize,...data})
      this.list=result.data.items
      
      this.page.total = result.data.counts// 总条数
    },
    //定义一个格式化的函数
    formatterBoolean(row,column,cellValue,index){
      //row=>当前行数据
      //column当前列信息
      //cellValue当前单元格的值
      //index索引
      return cellValue ? '启用' : '禁用'

    },
    //启用或者禁用标签的方法
    EnableOrDisable(row){
      let mess = row.state ? '禁用' :'启用'
        this.$confirm(`您是否确定要${mess}状态吗`, '提示').then(() => {
           removeState({id:row.id,state:row.state === 1 ? 0 : 1}).then(result => {
            this.gettagslist()  
           })
        })
      }
  },
  created(){
   
   this.gettagslist()
    this.getSubjects()
   
   

  }
  
}
</script>

<style scoped>

</style>
