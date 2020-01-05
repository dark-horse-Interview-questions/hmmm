<template>
  <!-- 放置一个容器 -->
  <el-card>
    <el-row slot="header">
      <el-button type="primary" @click="dialogVisible = true">新建目录</el-button>
      <el-button type="primary">返回学科</el-button>
    </el-row>
    <!-- 表单容器 -->
    <el-form inline>
      <el-form-item label="目录名称">
        <el-input v-model="searchForm.directoryName"></el-input>
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="searchForm.state">
          <el-option
            v-for="item in status"
            :key="item.value"
            :value="item.value"
            :label="item.label"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="clear">清除</el-button>
        <el-button @click="search" type="primary">搜索</el-button>
      </el-form-item>
    </el-form>
    <!-- 表格 -->
    <el-table :data="list">
      <el-table-column prop="id" label="序号"></el-table-column>
      <el-table-column prop="directoryName" label="目录名称"></el-table-column>
      <el-table-column prop="creator" label="创建者"></el-table-column>
      <!-- 属性 formatter -->
      <el-table-column prop="addDate" label="创建日期">
        <!-- 作用域插槽 -->
        <!-- 过滤器  表达式 | 过滤器名称 -->
        <template slot-scope="obj">{{ obj.row.addDate | parseTimeByString}}</template>
      </el-table-column>
      <el-table-column prop="totals" align="center" label="面试题数量"></el-table-column>
      <el-table-column prop="state" align="center" label="状态">
        <template slot-scope="obj">{{ obj.row.state === 1 ? '开启' : '关闭 ' }}</template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="obj">
          <el-button @click="modify(obj.row.id)" type="text">修改</el-button>
          <el-button
            @click="changeState(obj.row)"
            type="text"
          >{{ obj.row.state === 1 ? '禁用' : '开启' }}</el-button>
          <el-button @click="delItem(obj.row.id)" type="text">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-row type="flex" justify="end" style="margin:30px 0">
      <el-pagination
        background
        layout="prev,pager,next"
        :total="page.total"
        :current-page="page.currentPage"
        :page-size="page.pageSize"
        @current-change="changePage"
      ></el-pagination>
    </el-row>
    <el-dialog :show-close="false" :visible="dialogVisible">
      <!-- 主体内容 -->
      <el-form ref="myForm" :model="formData" :rules="rules" label-width="80px">
        <el-form-item label="目录名称" prop="directoryName">
          <el-input v-model="formData.directoryName" style="width:50%"></el-input>
        </el-form-item>
        <el-form-item label="学科" prop="subjectID">
          <el-select v-model="formData.subjectID" style="width:50%">
            <el-option
              v-for="item in subjects"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <el-row slot="footer" type="flex" justify="end">
        <el-button type="primary" @click="submitForm">确定</el-button>
        <el-button @click="btnCancel">取消</el-button>
      </el-row>
    </el-dialog>
  </el-card>
</template>

<script>
import {
  list as DirectorysList,
  remove as removeDirectory,
  removeState as changeState,
  add as addDirectory,
  detail as detailDirectory,
  update as updateDirectory
} from "../../api/hmmm/directorys";
import { status } from "../../api/hmmm/constants";
import { simple as getSubjects } from "../../api/hmmm/subjects";
export default {
  name: "DirectorysList",
  data() {
    return {
      dialogVisible: false, // 控制弹框的显示和隐藏
      // 搜索对象
      searchForm: {
        directoryName: "", // 目录名称
        state: null // 默认没状态
      },
      list: [], // 列表的数据
      page: {
        currentPage: 1, // 当前页码
        pageSize: 10, // 每页条数
        total: 0 // 总数量
      },
      status, // 状态数据
      formData: {
        subjectID: null,
        directoryName: ""
      },
      rules: {
        subjectID: [
          { required: true, message: "学科id不能为空", trigger: "blur" }
        ],
        directoryName: [
          { required: true, message: "目录名称不能为空", trigger: "blur" }
        ]
      },
      subjects: [] // 用来接收数据
    };
  },
  methods: {
    changePage(newPage) {
      this.page.currentPage = newPage;
      this.getDirectory(this.searchForm);
    },
    // 两种方案 1 直接修改前端数据(行不通，因为前端数据没有subjectID) 2  从后端查数据 给前端 再修改
    async modify(id) {
      // // 方案1
      // this.formData = row; // 直接获取我的数据
      // this.dialogVisible = true;
      let result = await detailDirectory({ id });
      this.formData = result.data;
      this.dialogVisible = true;
    },
    // 校验表单 并且提交
    async submitForm() {
      // 支持两种模式  回调函数  promise
      await this.$refs.myForm.validate();
      // 调用新增接口
      this.formData.id
        ? await updateDirectory(this.formData) // 是由id的
        : await addDirectory(this.formData); // id存在调用修改接口 否则调用新增接口
      //  操作成功
      this.$message({ type: "success", message: "操作成功" });
      // 重置数据
      this.getDirectory(this.searchForm); // 应该带状态查询
      this.btnCancel();
    },
    btnCancel() {
      this.formData = {
        subjectID: null,
        directoryName: ""
      };
      this.dialogVisible = false; // 关闭弹层
    },
    // 切换状态
    async changeState(row) {
      await this.$confirm("您是否确定改变此状态?");
      await changeState({ id: row.id, state: row.state === 1 ? 0 : 1 }); // 状态是反着的
      this.$message({ type: "success", message: "改变状态成功" });
      this.getDirectory(this.searchForm); // 应该带状态查询
    },
    // 删除方法
    async delItem(id) {
      await this.$confirm("您是否确定删除此数据?");
      // 确定删除
      await removeDirectory({ id });
      this.$message({ type: "success", message: "删除成功" });
      this.getDirectory(this.searchForm); // 应该带状态查询
    },
    search() {
      // 搜索方法
      this.page.currentPage = 1; // 搜索回到第一页
      this.getDirectory(this.searchForm);
    },
    // 将数据还原为初始状态
    clear() {
      this.searchForm = {
        directoryName: "", // 目录名称
        state: null // 默认没状态
      };
    },
    // 获取目录数据
    async getDirectory(data) {
      // 调用接口
      let result = await DirectorysList({
        page: this.page.currentPage,
        pagesize: this.page.pageSize,
        ...data
      });
      this.list = result.data.items; // 列表数据
      this.page.total = result.data.counts; // 总条数
    },
    async getSubjects() {
      let result = await getSubjects();
      this.subjects = result.data;
    }
  },
  created() {
    this.getSubjects(); // 调用学科
    this.getDirectory();
  }
};
</script>

<style scoped>
</style>
