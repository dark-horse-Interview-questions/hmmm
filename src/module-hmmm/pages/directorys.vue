<template>
  <el-card>
    <!-- 新建目录区域 -->
      <el-row slot="header">
        <el-button type="primary" @click="dialogVisible = true">新建目录</el-button>
        <el-button type="primary">返回学科</el-button>
      </el-row>
      <!-- 搜索区域form区 -->
      <el-form inline>
        <el-form-item label="目录名称" >
          <el-input v-model="searchForm.directoryName" ></el-input>
        </el-form-item>
        <el-form-item label="状态">
         <el-select v-model="searchForm.state">
           <el-option v-for="item in status" :key="item.value" :value="item.value" :label="item.label">

           </el-option>
         </el-select>
        </el-form-item>
        <el-form-item >
           <el-button @click="clear">清除</el-button>
          <el-button @click="search" type="primary">搜索</el-button>
        </el-form-item>
      </el-form>
      <!-- el-table表格区域 -->
      <el-table :data="getlist">
        <el-table-column prop="id" label="序号"></el-table-column>
        <el-table-column prop="directoryName" label="目录名称"></el-table-column>
        <el-table-column prop="creator" label="创建者"></el-table-column>
        <el-table-column prop="addDate" label="创建时间">
          <!-- 改变事件格式 -->
          <!-- 用作用域插槽或者formatter这里面用作用域插槽 -->
          <!--slot-scope="obj" 可以获取当前行的数据 -->
          <template slot-scope="obj">
            <!-- 这里用提前封装好的过滤器改变事件格式 -->
            {{obj.row.addDate | parseTimeByString}}
          </template>
        </el-table-column>
        <el-table-column prop="totals" label="面试题数量" align="center"></el-table-column>
        <el-table-column prop="state" label="状态" align="center">
          <template slot-scope="obj">
            {{obj.row.state===1?'启用':'禁用'}}
          </template>

        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="obj">
            <el-button type="text">修改</el-button>
            <el-button type="text" @click="changeState(obj.row)">{{obj.row.state===1?'禁用':'启用'}}</el-button>
            <el-button type="text" @click="delItem(obj.row.id)">删除</el-button>
          </template>
          
        </el-table-column>
      </el-table>
       <!-- 分页区域 -->
      <el-row type="flex" justify="end" style="margin:30px 0">
         <el-pagination
          background
          layout="prev,pager,next"
          :total="page.total"
          :page-size="page.pageSzie"
          :current-page="page.currentPage"
      >
      </el-pagination>
      </el-row>
      <!-- 弹层 -->
      <el-dialog :show-close="false" :visible="dialogVisible">
        <!-- 弹层主题 -->
        <el-form label-width="80px" :model="formData" :rules="rules" ref="myForm">
          <el-form-item prop="directoryName" label="目录名称">
            <el-input style="width:50%" v-model="formData.directoryName"></el-input>
          </el-form-item>
          <el-form-item prop="subjectID"  label="学科">
            <el-select style="width:50%" v-model="formData.subjectID">
              <el-option v-for="item in Subjects" :key="item.value" :value="item.value" :label="item.label"></el-option>
            </el-select>
          </el-form-item>
          
        </el-form>
          <el-row slot="footer" type="flex" justify="end">
            <el-button type="primary" @click="submitForm">确定</el-button>
            <el-button> 取消</el-button>
          </el-row>
      </el-dialog>
  </el-card>
 
  
</template>

<script>
import {list as getList,
        remove as removeDirectory,
        removeState as changeState,
        simple as getSubjects,
        add as addDirectory} from '../../api/hmmm/directorys'
import {status} from '../../api/hmmm/constants'
export default {
  name: 'DirectorysList',
  data() {
    return {
      //弹层
      dialogVisible:false,
      
      // 定义目录名称和状态的数据
      //并且V-model绑定
      searchForm:{
        directoryName:"",
        state:""

      },
      //定义列表数据是个空数组
      getlist:[],
    //定义分页对象
      page:{
        pageSize:10,
        currentPage:1,
        total:0
      },
      status,
      //弹层数据定义
      formData:{
        subjectID:'',
        directoryName:'',
      },
      rules:{
        subjectID:[{required:true,message:'内容不能为空'}],
        directoryName:[{required:true,message:'内容不能为空'}]
      },
      Subjects:[]
    }
  },
  methods:{
    //手动校验表单
    async submitForm(){
    await this.$refs.myForm.validate();
    //调用新增接口add
    await addDirectory(this.formData);
    //成功提示
    this.$message({
      type:'success',
      message:'提交成功'
    });
    //清空弹层
    this.formData ={
      subjectID:'',
      directoryName:'',
    };
    //关闭弹层
    this.dialogVisible = false;

    },

    //切换状态
   async changeState(row){
     await this.$confirm('您确定改变此状态')
     await changeState({id:row.id,state: row.state === 1 ? 0 : 1})
     this.$message({type:"success",message:'改变状态成功'})
       this.getDirectory(this.searchForm )
    },
    //获取弹层下拉菜单
   async getSubjects(){
     let result =await getSubjects()
     this.Subjects = result.data

    },
    //删除功能
   async delItem (id) {
        //友好提示
    await this.$confirm('您确定要删除吗')
    await  removeDirectory({id})
    this.$message({
      type:'success',
      message:'删除成功'
    })
    //带状态查询所以要携带参数    
   this.getDirectory(this.searchForm )
      
    },
    //搜索功能
    search(){
      this.page.currentPage=1,
      this.getDirectory(this.searchForm)

    },
    //清除数据
      clear(){
        this.searchForm = {
        directoryName:"",
        state:""

      }
      },
    //加载数据
    async getDirectory(data){
      //通过接口得知getList需要传递参数
      //page/pagesize
    let result = await getList({
      page:this.currentPage,
      pagesize:this.pageSize,
      ...data
    })
    this.getlist = result.data.items//列表数据
    this.page.total = result.data.count//总条数
    }
    
    },
   created(){
     this.getSubjects()//调用学科
     this.getDirectory()
   } 

  
}
</script>

<style scoped>
</style>
