<!--余票管理-->
<template>
    <div>
        <div style="margin: 10px 0">

            <!-- <el-input style="width: 200px" placeholder="请输入日期" suffix-icon="el-icon-date" class="ml-5" v-model="sdate" clearable></el-input>                -->
                <!-- 改为日期选择器           -->
            <el-date-picker
                v-model="sdate"
                type="date"
                placeholder="选择日期"
                value-format="yyyy-MM-dd"
                :picker-options="time_option">
            </el-date-picker>
         
            <el-input style="width: 200px" placeholder="请输入座位类型" suffix-icon="el-icon-search" v-model="seattype" clearable></el-input>
            <el-input style="width: 200px" placeholder="请输入列车编号" suffix-icon="el-icon-position" class="ml-5" v-model="tid" clearable></el-input>
            <el-input style="width: 200px" placeholder="请输入车站编号" suffix-icon="el-icon-message" class="ml-5" v-model="stationId" clearable></el-input>
            <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
            <el-button type="warning" @click="reset">重置</el-button>
        </div>

        <div style="margin: 10px 0">
            <el-button type="primary" @click="handleAdd">新增 <i class="el-icon-circle-plus-outline"></i></el-button>
            <el-popconfirm
                    class="ml-5"
                    confirm-button-text='确定'
                    cancel-button-text='我再想想'
                    icon="el-icon-info"
                    icon-color="red"
                    title="您确定批量删除这些数据吗？"
                    @confirm="delBatch"
            >
                <el-button type="danger" slot="reference">批量删除 <i class="el-icon-remove-outline"></i></el-button>
            </el-popconfirm>
            <el-upload action="http://localhost:9090/user/import" :show-file-list="false" accept="xlsx" :on-success="handleExcelImportSuccess" style="display: inline-block">
                <el-button type="primary" class="ml-5">导入 <i class="el-icon-bottom"></i></el-button>
            </el-upload>
            <el-button type="primary" @click="exp" class="ml-5">导出 <i class="el-icon-top"></i></el-button>
        </div>

        <el-table :data="tableData" border stripe :header-cell-class-name="'headerBg'"  @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column prop="lid" label="余票表编号" width="100"></el-table-column>
            <el-table-column prop="sdate" label="日期" width="100"></el-table-column>
            <el-table-column prop="seattype" label="座位类型" width="150"></el-table-column>
            <el-table-column prop="stationId" label="车站编号" width="150"></el-table-column>
            <el-table-column prop="tid" label="列车编号" width="150"></el-table-column>
            <el-table-column prop="leftnum" label="剩余票数" ></el-table-column>
            <el-table-column label="操作"  width="200" align="center">
                <template slot-scope="scope">
                    <el-button type="success" @click="handleEdit(scope.row)">编辑 <i class="el-icon-edit"></i></el-button>
                    <el-popconfirm
                            class="ml-5"
                            confirm-button-text='确定'
                            cancel-button-text='我再想想'
                            icon="el-icon-info"
                            icon-color="red"
                            title="您确定删除吗？"
                            @confirm="del(scope.row.lid)"
                    >
                        <el-button type="danger" slot="reference">删除 <i class="el-icon-remove-outline"></i></el-button>
                        <el-button>放票 <i class="el-icon-remove-outline"></i></el-button>
                    </el-popconfirm>
                </template>
            </el-table-column>
        </el-table>
        <div style="padding: 10px 0">
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="pageNum"
                    :page-sizes="[2, 5, 10, 20]"
                    :page-size="pageSize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </div>

        <el-dialog title="列车信息" :visible.sync="dialogFormVisible" width="30%" v-dialogDrag>
            <el-form label-width="80px" size="small" >
                <el-form-item label="日期" style="width: 200px">
                <el-date-picker
                v-model="form.sdate"
                type="date"
                placeholder="选择日期"
                value-format="yyyy-MM-dd"
                :picker-options="time_option"
                    >
                </el-date-picker>
                    <!-- <el-input v-model="form.sdate" autocomplete="off"></el-input> -->
                </el-form-item>
                <el-form-item label="座位类型">
                    <el-input v-model="form.seattype" autocomplete="off" clearable></el-input>
                </el-form-item>
                <el-form-item label="车站编号">
                    <el-input v-model="form.stationId" autocomplete="off" clearable></el-input>
                </el-form-item>
                <el-form-item label="列车编号" clearable>
                    <el-input v-model="form.tid" autocomplete="off" clearable></el-input>
                </el-form-item>
                <el-form-item label="剩余票数">
                    <el-input v-model="form.leftnum" autocomplete="off" clearable></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="save">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
 
        name: "Lefts",
        data() {
            return {
                tableData: [],
                total: 0,
                pageNum: 1,
                pageSize: 10,
                sdate:"",
                seattype: "",
                tid: "",
                stationId: "",
                form: {},
                dialogFormVisible: false,
                multipleSelection: [],
                roles: [],
                // 日期选择
         
        time_option: {
            disabledDate(time) {
                return time.getTime() < Date.now() -8.64e7;
            },
        },       

        }
        },
        created() {
            this.load()
        },
        methods: {
            load() {
                this.request.get("/lefts/page", {
                    params: {
                        pageNum: this.pageNum,
                        pageSize: this.pageSize,
                        sdate:this.sdate,
                        seattype: this.seattype,
                        tid: this.tid,
                        stationId: this.stationId,
                    }
                }).then(res => {

                    this.tableData = res.data.records
                    this.total = res.data.total

                })

                // this.request.get("/role").then(res => {
                //     this.roles = res.data
                // })
            },
            save() {
                this.request.post("/lefts", this.form).then(res => {
                    if (res.code === '200') {
                        this.$message.success("保存成功")
                        this.dialogFormVisible = false
                        this.load()
                    } else {
                        this.$message.error("保存失败")
                    }
                })
            },
            handleAdd() {
                this.dialogFormVisible = true
                this.form = {}
            },
            handleEdit(row) {
                this.form = JSON.parse(JSON.stringify(row))
                this.dialogFormVisible = true
            },
            del(lid) {
                this.request.delete("/lefts/" + lid).then(res => {
                    if (res.code === '200') {
                        this.$message.success("删除成功")
                        this.load()
                    } else {
                        this.$message.error("删除失败")
                    }
                })
            },
            handleSelectionChange(val) {
                console.log(val)
                this.multipleSelection = val
            },
            delBatch() {
                let ids = this.multipleSelection.map(v => v.lid)  // [{}, {}, {}] => [1,2,3]
                this.request.post("/lefts/del/batch", ids).then(res => {
                    if (res.code === '200') {
                        this.$message.success("批量删除成功")
                        this.load()
                    } else {
                        this.$message.error("批量删除失败")
                    }
                })
            },
            reset() {
                this.sdate=""
                this.seattype = ""
                this.tid = ""
                this.stationId = ""
                this.load()
            },
            handleSizeChange(pageSize) {
                console.log(pageSize)
                this.pageSize = pageSize
                this.load()
            },
            handleCurrentChange(pageNum) {
                console.log(pageNum)
                this.pageNum = pageNum
                this.load()
            },
            exp() {
                window.open("http://localhost:9090/lefts/export")
            },
            handleExcelImportSuccess() {
                this.$message.success("导入成功")
                this.load()
            }
        }
    }
</script>

<style scoped>

</style>