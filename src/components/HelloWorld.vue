<template>
    <div class="hello">
        <el-card>
            <div class="buttonList">
                <el-button @click="classChoose('all')">全部画作</el-button>
                <el-button @click="classChoose('circle')">圆画</el-button>
                <el-button @click="classChoose('rectangle')">长画</el-button>
                <el-button @click="classChoose('sector')">扇形画</el-button>
                <el-button @click="classChoose('square')">方画</el-button>
                <el-button @click="add" type="primary" icon="el-icon-plus">添加新画</el-button>
            </div>
            <el-table :data="curPageData" border stripe>
                <el-table-column prop="id" label="ID"></el-table-column>
                <el-table-column prop="name" label="名称"></el-table-column>
                <el-table-column label="图片">
                    <template slot-scope="scope">
                        <el-image :src="'/fileServer'+scope.row.src" :fit="'fit'"></el-image>
                    </template>
                </el-table-column>
                <el-table-column label="分类">
                    <template slot-scope="scope">
                        <span v-if="scope.row.shape == 'rectangle'">长</span>
                        <span v-if="scope.row.shape == 'square'">方</span>
                        <span v-if="scope.row.shape == 'circle'">圆</span>
                        <span v-if="scope.row.shape == 'sector'">扇</span>
                    </template>
                </el-table-column>
                <el-table-column prop="src" label="地址"></el-table-column>
                <el-table-column prop="price" label="价格">
                  <template slot-scope="scope">
                      <el-link :underline="false" type="info">￥{{scope.row.price}}</el-link>
                  </template>
                </el-table-column>
                <el-table-column prop="introduce" label="简介"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button @click="edit(scope)" type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <el-button @click="del(scope)" type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <div>
            <el-pagination layout="prev, pager, next" :total="tableData.length" @current-change="curChange" :current-page.sync="curPage">
            </el-pagination>
        </div>
        <el-dialog title="编辑" :visible.sync="editDialogVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px">
                <el-form-item label="ID" required>
                    <el-input v-model="editForm.id" disabled></el-input>
                </el-form-item>
                <el-form-item label="名称" required>
                    <el-input v-model="editForm.title"></el-input>
                </el-form-item>
                <el-form-item label="简介" required>
                    <el-input v-model="editForm.introduce" type="textarea" :rows="2"></el-input>
                </el-form-item>
                <el-form-item label="价格" required>
                    <el-input v-model="editForm.price"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editConfirm">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog title="添加" :visible.sync="addDialogVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" ref="addForm" :rules="addFormRules">
                <el-form-item label="名称" required prop="title">
                    <el-input v-model="addForm.title"></el-input>
                </el-form-item>
                <el-form-item label="简介" required prop="introduce">
                    <el-input v-model="addForm.introduce" type="textarea" :rows="2"></el-input>
                </el-form-item>
                <el-form-item label="价格" required prop="price">
                    <el-input v-model="addForm.price"></el-input>
                </el-form-item>
                <el-form-item label="分类" required prop="class" style="text-align:left">
                    <el-select v-model="addForm.class">
                        <el-option v-for="(item,index) in allClass" :key="index" :label="item.name" :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item style="text-align:left">
                    <el-upload action="/api/saveimg" :file-list="fileList" :multiple="false" :on-success="uploadSuccess"
                        type="primary" :accept="'.jpg,.png'">
                        <el-button slot="trigger" size="small" type="primary">上传文件</el-button>
                    </el-upload>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addConfirm">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
/* eslint-disable */
    export default {
        name: 'HelloWorld',
        data() {
            return {
                curPage:1,
                tableData: [],
                curClassname: "all",
                curPageData: [],
                curClassData: [],
                fileList: [],
                allClass: [],
                editForm: {
                    id: "",
                    title: "",
                    introduce: "",
                    price: ""
                },
                addForm: {
                    title: "",
                    introduce: "",
                    price: "",
                    class: "",
                    filename: "",
                },
                curData: [],

                editDialogVisible: false,
                addDialogVisible: false,

                addFormRules: {
                    title: [{
                        required: true,
                        message: '名称不能为空',
                        trigger: 'blur'
                    }, ],
                    introduce: [{
                        required: true,
                        message: '介绍不能为空',
                        trigger: 'blur'
                    }, ],
                    price: [{
                        required: true,
                        message: '价格不能为空',
                        trigger: 'blur'
                    }, ],
                    class: [{
                        required: true,
                        message: '分类不能为空',
                        trigger: 'blur'
                    }, ],
                }
            }
        },
        methods: {
            classChoose(classname) {
                this.curClassname = classname;
                this.curPage = 1;
                this.reload(classname);
            },
            edit(scope) {   //显示修改弹框
                //this.editForm = scope.row;         浅拷贝
                this.editForm = JSON.parse(JSON.stringify(scope.row));
                this.editDialogVisible = true;
                
            },
            editConfirm() {   //修改确认
                this.editDialogVisible = false;
                this.$axios.post('/api/paintingChange', {
                    id: this.editForm.id,
                    title: this.editForm.title,
                    introduce: this.editForm.introduce,
                    price: this.editForm.price,
                }).then(res => {
                    this.reload(this.curClassname);
                })
            },
            del(scope) {
                this.$confirm("确认删除吗？", "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: 'warning'
                }).then(() => {
                    this.$axios.post('/api/paintingDel', {
                            id: scope.row.id
                        })
                        .then(res => {
                            if (res.data.code == 'v') {
                                this.$message({
                                    type: 'success',
                                    message: '删除成功!'
                                });
                                this.reload(this.curClassname);
                            }
                        })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            curChange(e) {
                this.curPageData = this.tableData.slice((e - 1) * 10, e * 10);
            },
            add() {
                this.addDialogVisible = true;
            },
            uploadSuccess(response, file, fileList) {
                this.$message({
                    type: 'success',
                    message: "上传成功"
                });
                this.fileList = fileList;
                
                this.addForm.filename = response.filename;
            },
            addConfirm() {
                this.$refs.addForm.validate((valid) => {
                    if (this.fileList.length == 0) {
                        this.$message({
                            type: "warning",
                            message: "还未上传文件"
                        })
                    } else {
                        this.$axios.post('/api/creatPaiting', this.addForm).then(res => {
                            
                            if (res.data.code == 'v') {
                                this.addDialogVisible = false;
                                this.$message({
                                    type: 'success',
                                    message: "添加成功"
                                });
                                this.reload(this.curClassname);
                            }
                        })
                    }
                })
            },
            reload(classname) {
                this.$axios.get('/api/paintingClass',{
                  params:{
                    class:classname != undefined ? classname : this.curClassname
                  }
                }).then(res => {
                    this.tableData = res.data.data;
                    this.curPageData = res.data.data.slice((this.curPage - 1) * 10, this.curPage * 10);
                })
            }
        },
        mounted: function () {
            this.$axios.get('/paint/getList').then(res => {
                
                this.tableData = res.data.data.records;
                this.curPageData = this.tableData.slice(0, 10);
            })
            this.$axios.get('/paint/getClassList').then(res => {
                this.allClass = res.data.data.records;
            })
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h1,
    h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    .buttonList {
        text-align: right;
        margin: 10px auto;
    }
</style>