<template>
    <div class="user" v-has="getUserList">
        <div class="top-wrap">
          <el-button type="primary" class="mr20" size="mini" v-has="editUserMsg" @click="addUser">添加客服</el-button>
          <el-input v-model="userName" @keydown.native.enter="initList(1)" placeholder="请输入客服名" size="mini"></el-input>
          <el-input v-model="userPhone" @keydown.native.enter="initList(1)" placeholder="请输入手机号" size="mini"></el-input>
          <el-input v-model="userQq" @keydown.native.enter="initList(1)" placeholder="请输入QQ" size="mini"></el-input>
          <el-button @click="initList(1)" slot="append" type="primary" plain icon="el-icon-search" size="mini">搜索</el-button>
        </div>
        <div class="list-wrap mt20">
          <el-table :data="userList" stripe height="70vh">
            <el-table-column prop="num" label="序号" width="60"></el-table-column>
              <el-table-column prop="name" label="客服名" width="180"></el-table-column>
              <el-table-column prop="nickname" label="客服昵称" width="180"></el-table-column>
              <el-table-column label="客服类型" width="140px">
                <template slot-scope="type">
                  <span v-if="type.row.type === 1">客服人员</span>
                  <span v-if="type.row.type === 2">库内操作人员</span>
                  <span v-if="type.row.type === 3">客服及库内操作人员</span>
                </template>
              </el-table-column>
              <el-table-column label="小时成本" width="160px">
                <template slot-scope="hourfee">
                  <el-row :gutter="24">
                    <el-col :span="10">
                      <span v-if="isHasCheckFee === true">{{hourfee.row.hourfee}}</span>
                      <span v-else-if="hourfee.row.name === loginName">{{hourfee.row.hourfee}}</span>
                    </el-col>
                    <el-col :span="14">
                      <el-button @click="editUserFeeShow(hourfee.row)" v-has="editUserFee" type="text">修改</el-button>
                    </el-col>
                  </el-row>
                </template>
              </el-table-column>
              <el-table-column prop="phone" label="手机"></el-table-column>
              <el-table-column prop="qq" label="QQ"></el-table-column>
              <el-table-column prop="email" label="Email" width="180px"></el-table-column>
              <el-table-column prop="remark" label="备注"></el-table-column>
              <el-table-column prop="roleName" label="客服角色" width="140"></el-table-column>
              <el-table-column type="expand" label="客服权限" width="100">
                <template slot-scope="authList">
                  <el-tag v-for="(item,index) in authList.row.authList" type="info" :key="index" class="tags">{{item}}</el-tag>
                  <span v-if="authList.row.authList.length <= 0">暂无权限</span>
                </template>
              </el-table-column>
              <el-table-column prop="enable" label="状态" width="100px">
                <template slot-scope="scope">
                  <el-switch v-model="scope.row.enable" v-has="changeUserState" @change="changeRowUserState(scope.row)" active-value='1' inactive-value='0'></el-switch>
                </template>
              </el-table-column>
              <el-table-column prop="isAdmin" fixed="right" label="操作" width="200">
                <template slot-scope="scope">
                  <el-button size="mini" v-has="editUserMsg" @click="showEditUserDialog(scope.row)">编辑</el-button>
                  <el-button size="mini" v-has="editUserListPwd"  @click="resetUserPwd(scope.row.name)">重置登录密码</el-button>
                </template>
              </el-table-column>
          </el-table>
          <el-pagination background layout="prev, pager, next" :total="pageTotal" :current-page.sync="currentPage" :page-size="pagesize" @current-change="pageChange" class="mt20"></el-pagination>
        </div>

        <!-- 编辑/修改用户Form -->
        <el-dialog :title="editUserDialogTitle" :visible.sync="editUserDialog" @closed="closeEditUserDialog">
          <el-form :model="editUserForm" :rules="editUserRules" ref="editUserForm" label-width="80px">
            <el-form-item label="客服名" prop="username">
              <span v-if="isEdit">{{editUserForm.username}}</span>
              <el-input v-else v-model="editUserForm.username" maxlength=30></el-input>
            </el-form-item>
            <el-form-item label="客服昵称" prop="nickname">
              <el-input v-model="editUserForm.nickname" maxlength=30></el-input>
            </el-form-item>
            <el-form-item label="客服类型" prop="type">
              <el-select v-model="editUserForm.type">
                <el-option label="客服人员" :value="1"></el-option>
                <el-option label="库内操作人员" :value="2"></el-option>
                <el-option label="客服及库内操作人员" :value="3"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="手机" prop="phone">
              <el-input v-model="editUserForm.phone" maxlength="11"></el-input>
            </el-form-item>
            <el-form-item label="QQ" prop="qq">
              <el-input v-model="editUserForm.qq" ></el-input>
            </el-form-item>
            <el-form-item label="Email" prop="email">
              <el-input v-model="editUserForm.email" ></el-input>
            </el-form-item>
            <el-form-item label="角色" prop="rid">
              <el-select v-model="editUserForm.rid">
                <el-option label="请选择" :value="0"></el-option>
                <el-option v-for="item in userRoleList" :label="item.description" :value="item.rid" :key="item.rid"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="备注" prop="remark">
              <el-input v-model="editUserForm.remark"></el-input>
            </el-form-item>
            <el-input v-model="editUserForm.uid" type="hidden"></el-input>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="closeEditUserDialog">取 消</el-button>
            <el-button type="primary" v-has="editUserMsg" @click="submitEditUser(editUserForm)">确 定</el-button>
          </div>
        </el-dialog>

        <!-- 人工成本费用修改 -->
        <el-dialog title="修改客服小时成本" :visible.sync="editUserFeeDialog" @closed="clearEditUserFee">
          <el-form :model="editUserFeeForm" :rules="editUserFeeRules" ref="editUserFeeForm" label-width="80px">
            <el-form-item label="客服名" prop="name">
              <span>{{editUserFeeForm.name}}</span>
            </el-form-item>
            <el-form-item label="小时成本" prop="hourfee">
              <el-input v-model="editUserFeeForm.hourfee"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="editUserFeeDialog = false">取 消</el-button>
            <el-button type="primary" v-has="editUserFee" @click="submitEditUserFee">确 定</el-button>
          </div>
        </el-dialog>

    </div>
</template>

<script>
import {getUserList, changeUserState, editUserMsg,editUserListPwd,getUserSetRole,editUserFee,queryUserFee,PAGESIZE } from '@/api'
import { validatePhone,priceNumber,toDecimal2 } from '@/public/rules'
import util from '@/public/util'

export default {
     data() {
      return {
        getUserList, changeUserState, editUserMsg,editUserListPwd,editUserFee,
        userList: [],
        userName:'',
        userPhone:'',
        userQq:'',
        currentPage:1,
        pageTotal:0,
        pagesize:PAGESIZE,
        userRoleList:{},
        editUserDialogTitle:'',
        isEdit:false,
        editUserDialog:false,
        editUserForm:{
          username:'',
          nickname:'',
          phone:'',
          qq:'',
          email:'',
          uid:'',
          rid:'',
          remark:'',
          type:1
        },
        editUserRules:{
          username:[
            {required: true, message: '请输入客服姓名', trigger: 'blur'},
            {max:30,message:'最多30个字符',trigger:['blur', 'change']}
          ],
          nickname:[{required: true, message: '请输入客服昵称', trigger: 'blur'}],
          phone:[
            {required: false, message: '请输入正确的手机号码', trigger: 'blur'},
            {validator:validatePhone,trigger:['blur', 'change']}
          ],
          type:[
            {required: true, message: '请选择客服类型', trigger:['blur', 'change']}
          ],
          qq:[
            {required: false}
          ],
          email:[
            {required: false}
          ],
          rid:[
            {required: false}
          ],
          remark:[
            {required: false}
          ]
        },
        loginName:sessionStorage.getItem('username'),
        isHasCheckFee:util.hasPrivileges(queryUserFee),
        editUserFeeDialog:false,
        editUserFeeForm:{
          name:'',
          hourfee:''
        },
        editUserFeeRules:{
          name:[{required: false}],
          hourfee:[
            {required: true, message: '请输入客服小时成本', trigger: 'blur'},
            {validator:priceNumber,trigger:['blur', 'change']}
          ]  
        }
      }
    },
    created(){
      this.initList(1);
      this.initUserRole();
    },
     methods: {
      initList(page){
        this.currentPage = page;
        getUserList.r({uName:this.userName,phone:this.userPhone,qq:this.userQq,currentPage:this.currentPage}).then( res =>{
          this.userList = res.userList;
          this.pageTotal = res.pageCount * this.pagesize;
          for(var i=0;i<res.userList.length;i++){
            res.userList[i].enable = res.userList[i].enable + '';
          }
          for(var i=0;i<this.userList.length;i++){
            this.userList[i].hourfee = toDecimal2(this.userList[i].hourfee) ;
          }
        })
      },
      pageChange(val){
        this.initList(val);
      },
      initUserRole(){
        getUserSetRole.r({userId:''}).then(res=>{
          if (res.error === 1000) {
            this.userRoleList = res.unRole;
          } else {
            this.$message.error('角色列表获取失败，请稍后再试')
          }
        })
      },
      changeRowUserState(row){
        let enable = parseInt(row.enable);//enable值改为Boolean类型，传给后台
        changeUserState.r({username:encodeURIComponent(row.name),enable:enable}).then ( res => {
          if (res.error == 0) {
            if ( row.enable == 1 ) {
              this.$message({
                message: '客服激活成功',
                type: 'success'
              });
            } else {
              this.$message({
                message: '客服锁定成功',
                type: 'success'
              });
            }
            
          } else {
            if ( row.enable == 1 ) {
              this.$message({
                message: '客服激活失败',
                type: 'error'
              });
            }else{
              this.$message({
                message: '客服锁定失败',
                type: 'error'
              });
            }
          }
        });
      },
      closeEditUserDialog(){
        this.editUserDialog = false;
        this.editUserForm={
          username:'',
          nickname:'',
          phone:'',
          qq:'',
          email:'',
          uid:'',
          rid:'',
          remark:'',
          type:1
        };
        this.$refs['editUserForm'].clearValidate();
      },
      addUser(){
        this.isEdit = false;
        this.editUserDialogTitle = '新增客服';
        this.editUserDialog = true;
      },
      showEditUserDialog(row){
        this.isEdit = true;
        this.editUserDialogTitle = '修改客服资料';
        this.editUserDialog = true;
        this.editUserForm = JSON.parse(JSON.stringify(row));
        this.editUserForm.username = row.name;
      },
      submitEditUser(form){
        if (form.remark === undefined) {
          form.remark = ''
        }
        form.username = encodeURI(form.username);
        form.remark = encodeURI(form.remark)
        this.$refs.editUserForm.validate((valid) =>{
          if(valid){
            editUserMsg.r(form).then(res => {
              if ( res.error === 0 ) {
                let msg = '';
                if (this.isEdit === false) {
                  msg = '新增客服成功'
                } else {
                  msg = '客服信息修改成功'
                }
                this.$message({
                  type:'success',
                  message:msg
                });
                this.editUserDialog = false;
              } else {
                this.$message({
                  type:'error',
                  message:res.message
                })
              };
              this.initList(this.currentPage)
            })
          }else{
            this.$message({
              type:'error',
              message:'请正确填写内容'
            })
          }
        })
      },
      resetUserPwd(name){
        editUserListPwd.r({uname:name}).then(res =>{
          if (res.error === 0) {
            this.$alert(res.message,'客服'+name, {confirmButtonText: '确定',showClose:false})
          } else {
            this.$message({
              type:'error',
              message:res.message
            })
          }
        })
      },
      editUserFeeShow(row){
        let rowArr = JSON.parse(JSON.stringify(row));
        this.editUserFeeForm = {
          name:rowArr.name,
          hourfee:rowArr.hourfee
        };
        this.editUserFeeDialog = true;
      },
      submitEditUserFee(){

        this.$refs.editUserFeeForm.validate((valid)=>{
          if (valid) {
            editUserFee.r(this.editUserFeeForm).then(res=>{
              if (res.error === 0) {
                this.$message.success('客服小时成本修改成功!');
                this.initList(this.currentPage);
                this.editUserFeeDialog = false;
              }
            })
          }else{
            this.$message.error('请正确填写内容');
          }
        })
        
      },
      clearEditUserFee(){
        this.editUserFeeForm = {name:'',hourfee:''};
        this.$refs.editUserFeeForm.resetFields();
      }
    }
}
</script>

<style>
  .el-breadcrumb {
    padding:20px 10px;
    background-color: #B3C0D1;
    
  }
  .el-breadcrumb__separator {
    color:#333;
  }
  .tags {
    margin: 5px;
  }
</style>
