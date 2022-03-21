<template>
  <el-main>
    <!-- 搜索框
        el-form:就把他当做正常的div一样使用
        :model  表单绑定的数据域
        ref :相当于div的id,在当前页面是唯一的
        :inline为true时，排列在一行
         -->
    <el-form
      :model="parms"
      ref="searchForm"
      label-width="80px"
      :inline="true"
      size="small"
    >
      <el-form-item label="姓名">
        <el-input v-model="parms.userName"></el-input>
      </el-form-item>
      <el-form-item label="电话">
        <el-input v-model="parms.phone"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button icon="el-icon-search">查询</el-button>
        <el-button @click="addUser" type="primary" icon="el-icon-plus">新增</el-button>
      </el-form-item>
    </el-form>
    <!-- 用户列表
        :data表格的数据
        colum的prop需要跟返回的数据属性名对应
         -->
    <el-table
      :data="tableList"
      :height="tableHeight"
      size="small"
      empty-text="暂无数据"
      border
      stripe
    >
      <el-table-column prop="username" label="用户名"></el-table-column>
      <el-table-column prop="phone" label="电话"></el-table-column>
      <!-- <el-table-column prop="idCard" label="身份证"></el-table-column> -->
      <el-table-column align="center" prop="sex" label="性别">
         <template slot-scope="scope">
          <el-tag v-if="scope.row.sex == '1'" size="normal">男</el-tag>
          <el-tag v-if="scope.row.sex == '0'" type="success" size="normal">女</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="realName" label="真实姓名"></el-table-column>
      <el-table-column prop="email" label="邮箱"></el-table-column>
      <el-table-column prop="useStatus" label="账号状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.isUsed"
            :active-value="'0'"
            :inactive-value="'1'"
            active-text="启用"
            inactive-text="禁用"
            @change="changeUsed(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column align="center" width="200" label="操作">
        <template slot-scope="scope">
          <el-button
            icon="el-icon-edit"
            type="primary"
            size="small"
            @click="editUser(scope.row)">编辑</el-button>
          <el-button
            icon="el-icon-delete"
            type="danger"
            size="small"
            @click="deleteUser(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 
    size-change 页容量改变时触发事件
    current-change  页数改变时触发
    :current-page.sync 当前页数
    :page-size 每页几条数据
     :total 总条数
    -->
    <el-pagination
      @size-change="sizeChange"
      @current-change="currentChange"
      :current-page.sync="parms.curentPage"
      :page-sizes="[10, 20, 40, 80, 100]"
      :page-size="parms.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="parms.total"
      background
    >
    </el-pagination>
    <!-- 新增或编辑弹框 -->
    <sys-dialog
      :title="dialog.title"
      :visible="dialog.visible"
      :height="dialog.height"
      :width="dialog.width"
      @onClose="onClose"
      @onConfirm="onConfirm"
    >
      <div slot="content">
        <el-form
          :model="addModel"
          ref="addForm"
          :rules="rules"
          label-width="100px"
          :inline="true"
          size="small"
        >
          <el-form-item prop="username" label="用户名:">
            <el-input v-model="addModel.username"></el-input>
          </el-form-item>
          <el-form-item style="width:280px;" prop="sex" label="性别:">
            <el-radio-group  v-model="addModel.sex">
              <el-radio :label="'0'">男</el-radio>
              <el-radio :label="'1'">女</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item prop="phone" label="电话:">
            <el-input v-model="addModel.phone"></el-input>
          </el-form-item>
          <el-form-item prop="email" label="邮箱:">
            <el-input v-model="addModel.email"></el-input>
          </el-form-item>
          <!-- <el-form-item prop="idCard"  label="身份证:">
            <el-input v-model="addModel.idCard"></el-input>
          </el-form-item> -->
          <el-form-item prop="realName" label="真实姓名:">
            <el-input v-model="addModel.realName"></el-input>
          </el-form-item>
          <el-form-item prop="password" label="密码:">
            <el-input v-model="addModel.password"></el-input>
          </el-form-item>
          <!-- <el-form-item style="width:280px;" prop="status" label="离职:">
            <el-radio-group v-model="addModel.status">
              <el-radio :label="'1'">是</el-radio>
              <el-radio :label="'0'">否</el-radio>
            </el-radio-group>
          </el-form-item> -->
          <el-form-item style="width:280px;" prop="useStatus" label="账户启用:">
            <el-radio-group  v-model="addModel.useStatus">
              <el-radio :label="'1'">是</el-radio>
              <el-radio :label="'0'">否</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </div>
    </sys-dialog>
  </el-main>
</template>

<script>
import { getUserListApi, addUserApi } from "@/api/user";
import SysDialog from "@/components/System/SysDialog";
export default {
  //组件在使用之前需要注册
  components: {
    SysDialog,
  },
  //所有需要在页面展示的数据，都要在data里面显示的定义
  data() {
    return {
      rules: {
        userName: [
          {
            required: true,
            trigger: "change",
            message: "请填写姓名",
          },
        ],
        phone: [
          {
            required: true,
            trigger: "change",
            message: "请填写电话号码",
          },
        ],
        // idCard: [
        //   {
        //     required: true,
        //     trigger: "change",
        //     message: "请填写身份证号码",
        //   },
        // ],
        // status: [
        //   {
        //     required: true,
        //     trigger: "change",
        //     message: "请选择是否离职",
        //   },
        // ],
        useStatus: [
          {
            required: true,
            trigger: "change",
            message: "请选择是否启用",
          },
        ],
        sex: [
          {
            required: true,
            trigger: "change",
            message: "请选择性别",
          },
        ],
      },
      //新增或编辑表单数据域
      addModel: {
        id:'',
        type:'', //0： 新增 1： 编辑
        username:'',
        sex:'',
        phone:'',
        idcard:'',
        realName:'',
        password:'',
        // status:'',
        useStatus:'',
        email:''
      },
      //弹框属性
      dialog: {
        title: "",
        visible: false,
        height: 240,
        width: 610,
      },
      //表格的高度
      tableHeight: 0,
      //搜索框的数据绑定
      parms: {
        userName: "",
        phone: "",
        curentPage: 1,
        pageSize: 10,
        total: 0,
      },
      //表格的数据
      tableList: [],
    };
  },
  created() {
    this.getUserList();
  },
  mounted() {
    this.$nextTick(() => {
      this.tableHeight = window.innerHeight - 200;
    });
  },
  methods: {
    deleteUser(row) {
      console.log(row);
    },
    //编辑按钮
    editUser(row) {
      console.log(row);
    },
    //表格启用、禁用事件
    changeUsed(row) {
      console.log(row);
    },
    // //表格在职、离职点击事件
    // changeStatus(row) {
    //   console.log(row);
    // },
    //对话框确认事件
    onConfirm() {
      console.log(this.addModel)
            this.$refs.addForm.validate(async (valid) => {
        if (valid) {
          let res = null;
          if (this.addModel.type == "0") {
            res = await addUserApi(this.addModel);
          }
          //成功，刷新数据列表
          if (res && res.code == 200) {
            this.getUserList();
            this.dialog.visible = false;
          }
          console.log(res);
        }
      });
          console.log(res);
      // this.dialog.visible = false;
    },
    //对话框关闭
    onClose() {
      this.dialog.visible = false;
    },
    //新增按钮事件
    addUser() {
      //清空表单数据
      this.$resetForm("addForm", this.addModel);
      this.addModel.type = "0";
      this.dialog.title = "新增用户";
      this.dialog.visible = true;
    },
    //获取用户列表
    async getUserList() {
      let res = await getUserListApi(this.parms);
      if (res.code == 200) {
        console.log(res);
        this.tableList = res.data.content;
        this.parms.total = res.data.totalElements;
      }
    },
    //页容量改变触发
    sizeChange(val) {},
    //页数改变时触发
    currentChange(val) {},
  },
};
</script>

<style lang="scss" scoped>
</style>