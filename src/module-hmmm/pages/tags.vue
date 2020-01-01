<template>
  <el-card slot="header">
    <!-- {{list}} -->
 
    <el-form>
        <!-- 新增标签 -->
        <el-form-item style="border-bottom:1px solid #ccc">
         <el-button type="primary"> 新增标签</el-button>
          <el-button type="primary" > 返回学科</el-button>
        </el-form-item>
        <!-- 标签名称 -->
        <el-form-item label="标签名称" type="flex">
         <el-input style="width:20% ;margin-right:10px"></el-input>
       
          <el-button> 清除</el-button>
           <el-button type="primary" > 搜索</el-button>
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
          <el-button type="text">修改</el-button>
        <!-- 需要根据状态判断是禁用还是启用 -->
        <el-button type="text"> {{obj.row.state ? '禁用' : '启用'}}</el-button>
        <el-button type="text">删除</el-button>
        </template>
        
      </el-table-column>

    </el-table>  

  </el-card>
</template>

<script>
import {list as getlist} from "../../api/hmmm/tags"
// import {simple} from "../../api/hmmm/tags"
export default {
  // name: 'TagsList',
  data() {
    return {
      list:{}
     
    }
  },
  methods:{
    async  gettagslist(){
      let result= await getlist()
      this.list=result.data.items
    
    },
    //定义一个格式化的函数
    formatterBoolean(row,column,cellValue,index){
      //row=>当前行数据
      //column当前列信息
      //cellValue当前单元格的值
      //index索引
      return cellValue ? '启用' : '禁用'

    }
  },
  created(){
   this.gettagslist()
   

  }
  
}
</script>

<style scoped>

</style>
