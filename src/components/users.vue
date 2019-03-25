<template>
    <el-card class="box-card">
        <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>用户管理</el-breadcrumb-item>
          <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 搜索框 -->
        <el-row class="searchBox">
            <el-col>
                <el-input @clear="getAllUsers()" clearable class="searchInput" placeholder="请输入内容" v-model="query" >
                    <el-button @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
                </el-input>
                <el-button @click="showDiaAddUsers()" type="primary">添加用户</el-button>
            </el-col>
        </el-row>
        <!-- 表格区域 -->
            <!-- 
            // id: 500
            // username: "admin" 
            // email: "adsfad@qq.com"
            // mobile: "12345678"
            // create_time: 1486720211
            
            // mg_state: true
            // role_name: "主管"
            -->
        <el-table :data="list" style="width: 100%">
          <el-table-column prop="id" label="#" width="80"></el-table-column>
          <el-table-column prop="username" label="姓名" width="100"></el-table-column>
          <el-table-column prop="email" label="邮箱" width="140"></el-table-column>
          <el-table-column prop="mobile" label="电话" width="140"></el-table-column>

          <el-table-column label="创建日期" width="200">
              <template slot-scope="scope">
                  {{scope.row.create_time|fmtdate}}
              </template>
          </el-table-column>

          <el-table-column prop="date" label="用户状态" width="120">
              <template slot-scope="scope">
                  <el-switch
                    @change="changeState(scope.row)"
                    v-model="scope.row.mg_state"
                    active-color="#13ce66"
                    inactive-color="#ff4949">
                  </el-switch>
              </template>
          </el-table-column>

          <el-table-column prop="date" label="操作" width="200">
              <template slot-scope="scope">
                    <el-button 
                     @click="showDiEditUsers(scope.row)"
                    type="primary" icon="el-icon-edit" circle size="mini" plain></el-button>
                    <el-button 
                    @click="showMsgBox(scope.row)"
                    type="danger" icon="el-icon-delete" circle size="mini" plain></el-button>
                    <el-button 
                    @click="showDiaSetRole(scope.row)"
                    type="success" icon="el-icon-check" circle size="mini" plain></el-button>
              </template>
          </el-table-column>
        </el-table>
        <!-- 分页 -->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pagenum"
          :page-sizes="[2, 4, 6, 8]"
          :page-size="2"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
        <!-- 分配角色对话框 -->
        <el-dialog title="角色分配" :visible.sync="dialogFormVisibleRole">
          <el-form :model="formdata" label-position="left" label-width="80px">
            <el-form-item label="用户名">
                {{formdata.username}}
            </el-form-item>
            <el-form-item label="角色">
              <el-select v-model="selectVal" placeholder="请选择角色名称">
                <el-option label="请选择" :value="-1"></el-option>
                <el-option 
                :label="item.roleName"
                :value="item.id"
                v-for="(item) in roles" :key="item.id"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleRole = false">取 消</el-button>
            <el-button type="primary" @click="dialogFormVisibleRole = false">确 定</el-button>
          </div>
        </el-dialog>
        <!-- 编辑对话框 -->
        <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
          <el-form label-position="left" label-width="80px" :model="formdata">
            <el-form-item label="用户名">
              <el-input v-model="formdata.username"></el-input>
            </el-form-item>
            <el-form-item label="邮箱">
              <el-input v-model="formdata.email"></el-input>
            </el-form-item>
            <el-form-item label="电话">
              <el-input v-model="formdata.mobile"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
            <el-button type="primary" @click="editUser()">确 定</el-button>
          </div>
        </el-dialog>
        <!-- 添加用户对话框 -->
        <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
          <el-form label-position="left" label-width="80px" :model="formdata">
            <el-form-item label="用户名">
              <el-input v-model="formdata.username"></el-input>
            </el-form-item>
            <el-form-item label="密码">
              <el-input v-model="formdata.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱">
              <el-input v-model="formdata.email"></el-input>
            </el-form-item>
            <el-form-item label="电话">
              <el-input v-model="formdata.mobile"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
            <el-button type="primary" @click="addUser()">确 定</el-button>
          </div>
        </el-dialog>

    </el-card>
</template>

<script>
export default {
    data() {
        return {
            query:'',
            pagenum:1,
            pagesize:2,
            list:[],
            total:-1,
            dialogFormVisibleAdd:false,
            dialogFormVisibleEdit:false,
            dialogFormVisibleRole:false,

            // 添加表单数据
            formdata:{
                username:'',
                password:'',
                email:'',
                mobile:''
            },
            selectVal:-1,
            roles:[]
        }
    },
    created() {
        this.getTableData()
    },
    methods: {
        // 分配角色
        async showDiaSetRole(user){
            this.formdata = user
            this.dialogFormVisibleRole= true
            const res = await this.$http.get(`roles`)
            const {data,meta:{msg,status}} = res.data
            if (status===200){
                this.roles = data
                // console.log(this.roles)
            }
        },
        // 状态管理
        async changeState(user){
            const res =await this.$http.put(`users/${user.id}/state/${user.mg_state}`)
            // console.log(res)
            const {meta:{msg,status}} = res.data
            if(status===200){
                this.$message.success(msg)
            }
        },
        // 编辑用户
        async editUser(){
            const res =await this.$http.put(`users/${this.formdata.id}`,this.formdata)
            console.log(res)
            const {meta:{msg,status}} = res.data
            if(status===200){
                this.dialogFormVisibleEdit = false
                this.getTableData()
            }
        },
        showDiEditUsers(user){
            this.formdata = user
            this.dialogFormVisibleEdit = true
        },
        // 删除-弹出
        showMsgBox(user){
            this.$confirm('是否删除用户?', '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            })
            .then(async () => {
                // 发送请求
                const res = await this.$http.delete(`users/${user.id}`,)
                const {meta:{msg,status}} = res.data
                if (status===200){
                    // 提示框
                    this.$message.success('删除成功')
                    this.getTableData()
                }
              })
            .catch(() => {
                this.$message.info('已取消删除')    
            });
        },
        // 添加用户
        async addUser(){
            const res =await this.$http.post(`users`,this.formdata)
            // console.log(res)
            this.dialogFormVisibleAdd=false
            this.getTableData()
        },
        // 添加用户对话框
        showDiaAddUsers(){
            this.formdata = {}
            this.dialogFormVisibleAdd=true
        },
        // 搜索
        getAllUsers(){
            this.getTableData()
        },
        searchUser(){
            this.getTableData()
        },
        // 分页方法
        handleSizeChange(val) {
          console.log(`每页 ${val} 条`);
        //   每页几条请求
        this.pagenum = 1
        this.pagesize = val
        this.getTableData()
        },
        handleCurrentChange(val) {
          console.log(`当前页: ${val}`);
        //   根据新页码发送请求
        this.pagenum = val
        this.getTableData()
        },
        // 设置请求头
        async getTableData(){
            const AUTH_TOKEN = localStorage.getItem('token')
            this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN;
            const res = await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
            // console.log(res)
            const {data,meta:{status,msg}} = res.data
            if(status===200){
                this.total = data.total
                this.list = data.users
            }

        }
    },
}
</script>

<style>
    .searchBox{
        margin-top: 20px;
    }
    .searchInput{
        width: 400px;
    }
</style>
