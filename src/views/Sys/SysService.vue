<template>
  <div class="page-container">
    <!--工具栏-->
    <div class="toolbar" style="float:left;padding-top:10px;padding-left:15px;">
      <el-form :inline="true" :model="filters" :size="size">
        <el-form-item>
          <el-input v-model="filters.name" placeholder="名称"></el-input>
        </el-form-item>
        <el-form-item>
          <kt-button :label="$t('action.search')" perms="sys:sysService:view" type="primary" @click="findPage(null)"/>
        </el-form-item>
        <el-form-item>
          <kt-button :label="$t('action.add')" perms="sys:sysService:add" type="primary" @click="handleAdd" />
        </el-form-item>
      </el-form>
    </div>
    <!--表格内容栏-->
    <kt-service-table permsEdit="sys:sysService:edit" permsDelete="sys:sysService:delete"
              permsCreateJwt="sys:sysService:createJwt"
              :data="pageResult" :columns="columns"
              @findPage="findPage" @handleEdit="handleEdit" @handleCreateJwt="handleCreateJwt" @handleDelete="handleDelete">
    </kt-service-table>
    <!--新增编辑界面-->
    <el-dialog :title="operation?'新增':'编辑'" width="40%" :visible.sync="editDialogVisible" :close-on-click-modal="false">
      <el-form :model="dataForm" label-width="100px" :rules="dataFormRules" ref="dataForm" :size="size">
        <el-form-item label="id" prop="id"  v-if="dataForm.isPrimaryKey">
          <el-input v-model="dataForm.id" auto-complete="off"></el-input>
        </el-form-item>
        <!--<el-form-item label="系统表主键" prop="sysKey">
          <el-input v-model="dataForm.sysKey" auto-complete="off"></el-input>
        </el-form-item>-->
        <el-form-item label="所属系统" prop="sysKey">
          <BaseSelector ref="sysBaseSelector" v-model="dataForm.sysKey"
                        :isForbidden="!operation" :options="sysList"
                        @selectChangeEvent = "sysSelectChanged"></BaseSelector>
        </el-form-item>

        <el-form-item label="微服务编号" prop="code">
          <el-input v-model="dataForm.code" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="微服务名称" prop="name">
          <el-input v-model="dataForm.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="网关路由" prop="gateCode">
          <el-input v-model="dataForm.gateCode" auto-complete="off" placeholder="网关路由，比如/gate/01"></el-input>
        </el-form-item>
        <el-form-item label="开发负责人" prop="developer">
          <el-input v-model="dataForm.developer" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="微服务描述" prop="remark">
          <el-input v-model="dataForm.remark" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="加密key" prop="secret">
          <el-input v-model="dataForm.secret" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="创建人" prop="createBy"  v-if="false">
          <el-input v-model="dataForm.createBy" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="创建时间" prop="createTime"  v-if="false">
          <el-input v-model="dataForm.createTime" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="更新人" prop="lastUpdateBy"  v-if="false">
          <el-input v-model="dataForm.lastUpdateBy" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="更新时间" prop="lastUpdateTime"  v-if="false">
          <el-input v-model="dataForm.lastUpdateTime" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="是否删除  -1：已删除  0：正常" prop="delFlag"  v-if="false">
          <el-input v-model="dataForm.delFlag" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button :size="size" @click.native="editDialogVisible = false">{{$t('action.cancel')}}</el-button>
        <el-button :size="size" type="primary" @click.native="submitForm" :loading="editLoading">{{$t('action.submit')}}</el-button>
      </div>
    </el-dialog>

    <!--生成jwt界面-->
    <el-dialog :title="生成jwt" width="40%" :visible.sync="jwtDialogVisible" :close-on-click-modal="false">
      <el-form :model="dataForm" label-width="100px" ref="dataForm" :size="size">
        <el-form-item label="id" prop="id"  v-if="dataForm.isPrimaryKey">
          <el-input v-model="dataForm.id" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色列表" prop="remark">
          <el-input v-model="dataForm.userRoles" placeholder="输入角色编号用,隔开；如：admin,test" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="有效期" prop="remark">
          <el-input v-model="dataForm.validity" placeholder="日期格式yyyy-MM-dd，有效期到配置日期的0点0分0秒" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button :size="size" @click.native="jwtDialogVisible = false">{{$t('action.cancel')}}</el-button>
        <el-button :size="size" type="primary" @click.native="submitFormJwt" :loading="editLoading">{{$t('action.submit')}}</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
    import KtServiceTable from "@/views/Core/KtServiceTable"
    import KtButton from "@/views/Core/KtButton"
    import BaseSelector from "@/components/BaseSelector"
    import { format } from "@/utils/datetime"
    export default {
        components:{
            KtServiceTable,
            KtButton,
            BaseSelector
        },
        data() {
            return {
                size: 'small',
                filters: {
                    name: ''
                },
                columns: [
                    {prop:"id", label:"id", minWidth:100},
                    /*{prop:"sysKey", label:"系统表主键", minWidth:100},*/
                    {prop:"code", label:"微服务编号", minWidth:100},
                    {prop:"name", label:"微服务名称", minWidth:100},
                    {prop:"gateCode", label:"网关路由", minWidth:100},
                    {prop:"developer", label:"开发负责人", minWidth:100},
                    {prop:"remark", label:"微服务描述", minWidth:100},
                    {prop:"secret", label:"加密key", minWidth:100},
                ],
                pageRequest: { pageNum: 1, pageSize: 8 },
                subSysPageRequest:{ pageNum: 1, pageSize: 999 },
                pageResult: {},

                operation: false, // true:新增, false:编辑
                editDialogVisible: false, // 新增编辑界面是否显示
                jwtDialogVisible: false,
                editLoading: false,
                dataFormRules: {
                    label: [
                        { required: true, message: '请输入名称', trigger: 'blur' }
                    ]
                },
                sysList:[],
                // 新增编辑界面数据
                dataForm: {
                    id: null,
                    sysKey: null,
                    code: null,
                    name: null,
                    gateCode: null,
                    developer: null,
                    remark: null,
                    secret: null,
                    createBy: null,
                    createTime: null,
                    lastUpdateBy: null,
                    lastUpdateTime: null,
                    delFlag: null,
                }
            }
        },
        methods: {
            // 获取分页数据
            findPage: function (data) {
                if(data !== null) {
                    this.pageRequest = data.pageRequest
                }
                this.pageRequest.columnFilters = {name: {name:'name', value:this.filters.name}}
                this.$api.sysService.findPage(this.pageRequest).then((res) => {
                    this.pageResult = res.data
                }).then(data!=null?data.callback:'')
            },
            // 加载子系统信息
            findSubSyss: function () {
                this.$api.sysInfo.findPage(this.subSysPageRequest).then((res) => {
                    this.sysList = res.data.content
                })
            },
            // 批量删除
            handleDelete: function (data) {
                this.$api.sysService.batchDelete(data.params).then(data!=null?data.callback:'')
            },
            // 显示新增界面
            handleAdd: function () {
                this.editDialogVisible = true
                this.operation = true
                this.dataForm = {
                    id: '',
                    sysKey:'',
                    code: '',
                    name: '',
                    gateCode: '',
                    developer: '',
                    remark: '',
                    secret:'',
                    createBy: '',
                    createTime: '',
                    lastUpdateBy: '',
                    lastUpdateTime: '',
                    delFlag: '',
                }
                if(this.$refs.sysBaseSelector != null && this.$refs.sysBaseSelector!='undefined'){
                    this.$refs.sysBaseSelector.setSelectOptionNum(this.dataForm.sysKey)
                }
            },
            // 显示编辑界面
            handleEdit: function (params) {
                this.editDialogVisible = true
                this.operation = false
                this.dataForm = Object.assign({}, params.row)
                if(this.$refs.sysBaseSelector != null && this.$refs.sysBaseSelector!='undefined'){
                    this.$refs.sysBaseSelector.setSelectOptionNum(this.dataForm.sysKey)
                }
            },
            // 编辑
            submitForm: function () {
                this.$refs.dataForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true
                            let params = Object.assign({}, this.dataForm)
                            this.$api.sysService.save(params).then((res) => {
                                if(res.code == 200) {
                                    this.$message({ message: '操作成功', type: 'success' })
                                } else {
                                    this.$message({message: '操作失败, ' + res.msg, type: 'error'})
                                }
                                this.editLoading = false
                                this.$refs['dataForm'].resetFields()
                                this.editDialogVisible = false
                                this.findPage(null)
                            })
                        })
                    }
                })
            },

            // 显示jwt生成界面
            handleCreateJwt: function (params) {
                this.jwtDialogVisible = true
                this.operation = false
                this.dataForm = Object.assign({}, params.row)
            },
             // 编辑jwt
            submitFormJwt: function () {
                this.$refs.dataForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true
                            let params = Object.assign({}, this.dataForm)
                            this.$api.sysService.createJwt(params).then((res) => {
                                if(res.code == 200) {
                                    this.$alert(res.msg);

                                 //   this.$message({ dangerouslyUseHTMLString: true,
                                   //               message: '操作成功:' + res.msg, type: 'error' })
                                } else {
                                    this.$message({message: '操作失败, ' + res.msg, type: 'error'})
                                }
                                this.editLoading = false
                                this.$refs['dataForm'].resetFields()
                                this.jwtDialogVisible = false

                            })
                        })
                    }
                })
            },

            // 时间格式化
            dateFormat: function (row, column, cellValue, index){
                return format(row[column.property])
            },
            sysSelectChanged:function(sysId){
                this.dataForm.sysKey = sysId
            }
        },
        mounted() {
        },
        created() {
            this.findSubSyss()
        }
    }
</script>

<style scoped>

</style>
