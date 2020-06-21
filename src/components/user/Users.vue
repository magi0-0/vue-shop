<template>
    <div>
        <!--面包屑导航区-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-row :gutter="50">
                <el-col :span="7">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query"
                     :clearable="true" @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
                </el-col>
            </el-row>
            <el-table :data="userList" style="width: 100%" border stripe>
                <el-table-column type="index"/>
                <el-table-column prop="username" label="姓名"/>
                <el-table-column prop="email" label="邮箱"/>
                <el-table-column prop="mobile" label="电话"/>
                <el-table-column prop="role_name" label="角色"/>
                <el-table-column label="状态">
                    <!-- <template slot-scope="scope">
                        {{scope.row}}
                    </template> -->
                    <template v-slot="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"/>
                    </template>
                </el-table-column>
                <el-table-column label="操作">
                    <template v-slot="scope">
                        <el-tooltip class="item" effect="dark" :enterable="false" content="修改" placement="top">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" :enterable="false" content="删除" placement="top">
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="delUserById(scope.row.id)"></el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" :enterable="false" content="分配角色" placement="top">
                            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <!--分页区域-->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum" :page-sizes="[1,2,5,10]"
                :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" 
                :total="total">
            </el-pagination>
        </el-card>
        <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="70px">
                <!--prop是校验规则-->
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password" type="password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="70px">
                <!--prop是校验规则-->
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUser">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
export default {
    data(){
        var checkEmail=(rule, value, callback) => {
            const regEmail=/^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/
            if(regEmail.test(value)){
                return callback()
            }
            callback(new Error('请输入合法的邮箱'))
        }
        var checkMobile=(rule, value, callback) => {
            const regMobile=/^1[3456789]\d{9}$/
            if(regMobile.test(value)){
                return callback()
            }
            callback(new Error('请输入合法的电话'))
        }

        return{
            //获取用户列表的参数对象
            queryInfo:{
                query:'',
                //当前的页数
                pagenum:1,
                //当前每页显示多少条数据
                pagesize:5,
            },
            userList:[],
            total:0,
            addDialogVisible:false,
            editDialogVisible:false,
            addForm:{
                username:'',
                password:'',
            },
            addFormRules:{
                username:[
                    {required:true,message:'请输入用户名',trigger:'blur'},
                    {min:3,max:10,message:'用户名的长度在3-10之间',trigger:'blur'}
                ],
                password:[
                    {required:true,message:'请输入密码',trigger:'blur'},
                    {min:6,max:15,message:'密码的长度在6-15之间',trigger:'blur'}
                ],
                email:[
                    {required:true,message:'请输入邮箱',trigger:'blur'},
                    {validator:checkEmail,trigger:'blur'}
                ],
                mobile:[
                    {required:true,message:'请输入电话',trigger:'blur'},
                    {validator:checkMobile,trigger:'blur'}
                ]
            },
            addFormRef:{},
            editForm:{
            },
            editFormRef:{

            },
            editFormRules:{
                email:[
                    {required:true,message:'请输入邮箱',trigger:'blur'},
                    {validator:checkEmail,trigger:'blur'}
                ],
                mobile:[
                    {required:true,message:'请输入电话',trigger:'blur'},
                    {validator:checkMobile,trigger:'blur'}
                ]
            },
        }
    },
    created(){
        this.getUserList()
    },
    methods:{
        async getUserList(){
            const {data:res} = await this.$http.get('users',{params:this.queryInfo})
            console.log(res)
            if(res.meta.status!=200) return this.$message.error('获取用户列表失败！')
            this.userList=res.data.users
            this.total=res.data.total
        },
        handleSizeChange(newSize){
            console.log(newSize)
            this.queryInfo.pagesize=newSize
            this.getUserList()
        },
        //监听页码值改变
        handleCurrentChange(pageNum){
            console.log(pageNum)
            this.queryInfo.pagenum=pageNum
            this.getUserList()
        },
        //监听开关事件的改变
        async userStateChanged(userinfo){
            const {data:res} = await this.$http.put(
                `users/${userinfo.id}/state/${userinfo.mg_state}`)
            if(res.meta.status!=200){
                userinfo.mg_state=!userinfo.mg_state
                return this.$message.error('更新用户状态失败！')
            } 
            this.$message.success('更新用户状态成功！')
        },
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        addUser(){
            console.log(this.$refs.addFormRef)
            this.$refs.addFormRef.validate(async valid=>{
                if(!valid){
                    return
                }
                const {data:res} =await this.$http.post('users',this.addForm)
                if(res.meta.status!=201){
                    return this.$message.error('添加用户失败！')
                }
                this.$message.success('添加用户成功！')
                this.addDialogVisible=false
                this.getUserList()
            })
        },
        async showEditDialog(id){
            const {data:res} = await this.$http.get(`users/${id}`)
            this.editForm=res.data
            this.editDialogVisible=true
            console.log(res)
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        editUser(){
            this.$refs.editFormRef.validate(async valid=>{
                if(!valid){
                    return
                }
                const {data:res} = await this.$http.put(`users/${this.editForm.id}`,this.editForm)
                if(res.meta.status!=200){
                    return this.$message.error('更新失败！')
                }
                this.$message.success('更新成功！')
                this.editDialogVisible=false
                this.getUserList()
            })
        },
        delUserById(id){
            this.$confirm('是否删除用户?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
                }).then(async () => {
                    const {data:res} = await this.$http.delete(`users/${id}`)
                    if(res.meta.status!=200){
                        this.$message({
                            type: 'error',
                            message: '删除失败!'
                        });
                    }else{
                        this.getUserList()
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                    }
                    
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });          
            });
        },
    },
}
</script>

<style lang="less" scoped>
// .el-card{
//     padding-bottom: 20px;
// }
</style>