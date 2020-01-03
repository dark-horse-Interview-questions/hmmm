<template>
  <el-card>
    <!-- 新建目录区域 -->
      <el-row slot="header">
        <el-button type="primary">新建目录</el-button>
        <el-button type="primary">返回学科</el-button>
      </el-row>
      <!-- 搜索区域form区 -->
      <el-form inline>
        <el-form-item label="目录名称" >
          <el-input v-model="searchForm.directoryName" ></el-input>
        </el-form-item>
        <el-form-item label="状态">
         <el-select v-model="searchForm.state"></el-select>
        </el-form-item>
        <el-form-item >
           <el-button>清除</el-button>
          <el-button type="primary">搜索</el-button>
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
            <el-button type="text">{{obj.row.state===1?'禁用':'启用'}}</el-button>
            <el-button type="text">删除</el-button>
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
  </el-card>
 
  
</template>

<script>
import {list as getList} from '../../api/hmmm/directorys'
// 引入状态，引过来是一个变量名我们要在data里面定义一个属性接收
import {status} from '../../api/hmmm/constants'
export default {
  name: 'DirectorysList',
  data() {
    return {
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
      status
    }
  },
  methods:{
    //加载数据
    async getDirectory(){
      //通过接口得知getList需要传递参数
      //page/pagesize
    let result = await getList({
      page:this.currentPage,
      pagesize:this.pageSize
    })
    this.getlist = result.data.items//列表数据
    this.page.total = result.data.count//总条数
    }
    
    },
   created(){
     this.getDirectory()
   } 

  
}
</script>

<style scoped>
</style>
