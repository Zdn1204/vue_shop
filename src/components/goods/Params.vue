<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
        <el-alert
            title="注意：只允许为第三级分类设置相关参数！"
            type="warning"
            :closable="false"
            show-icon>
        </el-alert>

        <!-- 选择商品分类区域 -->
        <el-row class="cat_opt">
            <el-col>
                <span>选择商品分类：</span>
                <!-- 选择商品分类的级联选择框 -->
                <el-cascader
                    v-model="selectedKeys"
                    :options="cateList"
                    :props="cascaderProps"
                    @change="handleChange()"
                    style="width:250px">
                </el-cascader>
            </el-col>
        </el-row>

        <!-- 页签区 -->
        <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <!-- 添加动态参数面板 -->
            <el-tab-pane label="动态参数" name="many">
                <!-- 添加参数按钮 -->   
                <el-button type="primary" size="mini" :disabled="isBtndisabled" @click="addDialogVisible=true">添加参数</el-button>
                <!-- 动态参数表格 -->
                <el-table :data="manyTableData" border stripe>
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <el-tag 
                                v-for="(item,index) in scope.row.attr_vals" 
                                :key="index" 
                                closable
                                @close="handleClose(index,scope.row)"
                            >
                                {{item}}
                            </el-tag>
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index" label="#"></el-table-column>
                    <el-table-column prop="attr_name" label="参数名称"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
            <!-- 添加静态属性面板 -->
            <el-tab-pane label="静态属性" name="only">
                <!-- 添加属性按钮 -->
                <el-button type="primary" size="mini" :disabled="isBtndisabled" @click="addDialogVisible=true">添加属性</el-button>
                <!-- 静态属性表格 -->
                <el-table :data="onlyTableData" border stripe>
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <el-tag 
                                v-for="(item,index) in scope.row.attr_vals" 
                                :key="index" 
                                closable
                                @close="handleClose(index,scope.row)"
                            >
                                {{item}}
                            </el-tag>
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index" label="#"></el-table-column>
                    <el-table-column prop="attr_name" label="属性名称"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row)">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scope.row)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
        </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
        :title="'添加'+titleText"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed">
            <el-form :model="addForm" :rules="FormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
    </el-dialog>

    <!-- 修改参数的对话框 -->
    <el-dialog
        :title="'修改'+titleText"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed">
            <el-form :model="editForm" :rules="FormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
    data() {
        return {
            cateList:[],
            cascaderProps:{
                expandTrigger: "hover",
                value: "cat_id",
                label: "cat_name",
                children: "children",
            },
            // 级联选择框双向绑定到底数组
            selectedKeys:[],
            // 被激活的页签的名称
            activeName:"many",
            cateId:'',
            manyTableData:[],
            onlyTableData:[],
            // 控制添加对话框的显示与隐藏
            addDialogVisible:false,
            // 添加参数的表单数据对象
            addForm:{},
            // 添加表单的默认规则
            FormRules:{
                attr_name:[{required:true,message:'请输入参数名称',trigger:'blur'}],
            },
            editDialogVisible:false,
            editForm:{},
        }
    },
    created() {
        this.getCateList();
    },
    computed:{
        isBtndisabled(){
            if(this.selectedKeys.length !== 3){
                return true
            }
            return false
        },
        // 动态计算标题的文本
        titleText(){
            if(this.activeName === 'many'){
                return '动态参数'
            }
            return '静态属性'
        }
    },
    methods:{
        // 获取商品分类数据
        async getCateList() {
            const { data: res } = await this.$http.get("categories");
            if (res.meta.status !== 200) {
                return this.$message.error("获取商品分类失败");
            }
            // 把数据列表赋值给
            this.cateList = res.data;
        },
        // 级联选择框选中项的变化
        handleChange(){
            if(this.selectedKeys.length === 3){

                this.cateId = this.selectedKeys[2]

                this.getCatePramas(this.cateId)

                return
            }
            this.selectedKeys = [];
            this.manyTableData = []
            this.onlyTableData = []
            return null
        },
        // tabl页签点击事件等处理函数
        handleTabClick(){  
            this.getCatePramas(this.cateId)         
        },
        async getCatePramas(id){

            const {data: res} = await this.$http.get(`categories/${id}/attributes`,{params: {sel:this.activeName}})
            if(res.meta.status !== 200){
                this.$message.error('获取参数失败')
                return
            }
            res.data.forEach(item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                // 控制文本框的显示与隐藏
                item.inputVisible = false
                item.inputValue = ''
                // this.$set(item,'inputVisibel',false)
                // this.$set(item,'inputValue','')
            })
            
            // console.log(res.data)

            if(this.activeName === 'many'){
                this.manyTableData = res.data
            }else{
                this.onlyTableData = res.data
            }
        },
        // 监听添加对话框的关闭事件
        addDialogClosed(){
            this.$refs.addFormRef.resetFields();
        },
        // 点击确定，添加参数
        addParams(){
            this.$refs.addFormRef.validate(async valid=>{
                if(!valid) return
                const {data: res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
                    attr_name:this.addForm.attr_name,
                    attr_sel:this.activeName,
                })
                if(res.meta.status !== 201){
                    this.$message.error('添加参数失败！')
                    return 
                }
                this.$message.success('添加参数成功！')
                this.$refs.addFormRef.resetFields();
                this.getCatePramas(this.cateId)
            })
        },
        // 
        showEditDialog(params){
            // console.log(params)
            this.editForm = params
            this.editDialogVisible = true
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields();
        },
        // 点击确定，修改参数
        editParams(){
            this.$refs.editFormRef.validate(async valid=>{
                if(!valid) return
                const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
                    attr_name:this.editForm.attr_name,
                    attr_sel:this.activeName
                })
                if(res.meta.status !== 200){
                    this.$message.error('修改参数失败！')
                    return 
                }
                this.$message.success('修改参数成功！')
                this.getCatePramas(this.cateId)
            })
        },
        // 
        async removeParams(params){
            const confirmResult = await this.$confirm('此操作将永久删除该属性，是否继续？','提示',{
                confirmButtonText:'确定',
                cancelButtonText:'取消',
                type:'warning'
            }).catch(err=>err)
            if(confirmResult !== 'confirm'){
                this.$message.info('已取消本次操作')
                return
            }
            const {data: res} = await this.$http.delete(`categories/${this.cateId}/attributes/${params.attr_id}`);
            if(res.meta.status !== 200){
                this.$message.error('删除参数失败！')
                return
            }
            this.$message.success('删除参数成功！')
            this.getCatePramas(this.cateId)
        },
        // 点击按钮，展示文本输入框
        showInput(params){
            params.inputVisible = true
            // 让文本框自动获取焦点
            // $nextTick方法的作用，就是当页面上的元素被重新渲染之后，才会指定回调函数中的代码
            this.$nextTick(_=>{
                // console.log(this.$refs.saveTagInput) //一个vc实例
                // console.log(this.$refs.saveTagInput.$refs.input) //DOM节点
                this.$refs.saveTagInput.$refs.input.focus()
            })
        },
        // 文本框失去焦点或按下enter键，都会触发
        handleInputConfirm(params){
            if(params.inputValue.trim().length === 0){
                params.inputVisible = ''
                params.inputVisible = false
                return
            }
            params.attr_vals.push(params.inputValue.trim())
            params.inputValue = ''
            params.inputVisible = false

            this.saveAttrVals(params)
        },
        handleClose(i,params){
            params.attr_vals.splice(i,1)
            this.saveAttrVals(params)
        },

        async saveAttrVals(params){
            const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${params.attr_id}`,{
                attr_name:params.attr_name,
                attr_sel:params.attr_sel,
                attr_vals:params.attr_vals.join(' ')
            })

            if(res.meta.status !== 200){
                this.$message.error('修改参数项失败！')
                return
            }
            this.$message.success('修改参数项成功！')  
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt{
    margin-top: 20px;
}
.el-tabs{
    margin-top: 20px;
}
.el-tag{
    margin-left: 20px;
}
.input-new-tag{
    width: 120px;
    margin-left: 20px;
}
.button-new-tag{
    margin-left: 20px;
}
</style>