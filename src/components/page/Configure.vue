<template>
    <div>
        <div>
            <el-row :gutter="24">
                <el-col :span="2">
                    <el-button
                        icon="el-icon-edit"
                        @click="openSubTable(0,null,true)"
                        type="primary"
                    >新增</el-button>
                </el-col>
                <el-col :span="20">
                    <div class="grid-content bg-purple"></div>
                </el-col>

                <el-col :span="2">
                    <el-button type="info" icon="el-icon-refresh" circle @click="getTableData"></el-button>
                    <el-button type="info" icon="el-icon-document" circle></el-button>
                </el-col>
            </el-row>
        </div>
        <div>
            <br />
        </div>
        <div>
            <el-table :data="tableData" stripe style="width: 100%">
                <el-table-column prop="uuid" label="名称" width="180"></el-table-column>
                <el-table-column prop="address" label="地址"></el-table-column>
                <el-table-column prop="forecast_cap" label="预估容量(MB)" width="180"></el-table-column>
                <el-table-column prop="error_rate" label="错误率"></el-table-column>
                <el-table-column prop="expirarion_time" label="到期时间"></el-table-column>
                <el-table-column prop="status" label="状态"></el-table-column>
                <el-table-column prop="UpdatedAt" label="更新"></el-table-column>
                <el-table-column label="操作">
                    <template slot-scope="scope">
                        <el-button
                            type="primary"
                            @click="openSubTable(scope.$index, scope.row,false)"
                        >编辑</el-button>
                        <el-button type="danger" @click="deleteRow(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div slot="footer">
                <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="page.currentPage"
                    :page-sizes="[5, 10, 20, 40]"
                    :page-size="page.currentSize"
                    layout="total, sizes, prev, pager, next"
                    :total="page.totalSize"
                ></el-pagination>
            </div>
        </div>
        <div>
            <el-dialog title="信息" :visible.sync="dialogFormVisible">
                <el-form :model="form">
                    <el-form-item label="名称" :label-width="formLabelWidth">
                        <el-input v-model="form.uuid" autocomplete="off" placeholder="新增不用填写名称"></el-input>
                    </el-form-item>
                    <el-form-item label="到期时间" :label-width="formLabelWidth">
                        <el-date-picker
                            v-model="expiration"
                            type="datetime"
                            placeholder="选择日期时间"
                            value-format="yyyy-MM-dd HH:mm:ss"
                        ></el-date-picker>
                    </el-form-item>
                    <el-form-item label="预估容量(MB)" :label-width="formLabelWidth">
                        <el-input-number
                            v-model="size"
                            controls-position="right"
                            :step="64"
                            :min="64"
                            :max="2048"
                        ></el-input-number>
                    </el-form-item>
                </el-form>
                <div slot="footer">
                    <el-button>取 消</el-button>
                    <el-button type="primary" @click="postForm()">确 定</el-button>
                </div>
            </el-dialog>
        </div>
    </div>
</template>

<script>
import Vue from 'vue';
export default {
    data() {
        return {
            dialogFormVisible: false,
            form: {
                uuid: '',
                region: '',
                capacity: 0
            },
            formLabelWidth: '120px',
            page: {
                totalPage: 0,
                totalSize: 0,
                currentPage: 1,
                currentSize: 5
            },
            tableData: [],
            expiration: null,
            size: 0
        };
    },
    mounted: function() {
        this.getTableData();
    },
    methods: {
        getTableData() {
            Vue.axios({
                method: 'get',
                url: 'http://192.168.1.106:65221/api/v1/host/pagination',
                params: {
                    page_size: this.page.currentSize,
                    current_page: this.page.currentPage
                }
            }).then(res => {
                this.tableData = res.data.data;
                this.page.totalPage = res.data.page.total_page;
                this.page.totalSize = res.data.page.total_size;
            });
        },
        openSubTable(index, row, add) {
            this.dialogFormVisible = true;
            this.form.uuid = '';
            this.size = 0;
            if (add) {
                this.form.uuid = '';
            } else {
                this.form.uuid = row.uuid;
                this.size = row.forecast_cap;
            }
        },
        deleteRow(index, row) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            })
                .then(() => {
                    Vue.axios({
                        method: 'delete',
                        url: 'http://192.168.1.106:65221/api/v1/host/applyinfo',
                        params: {
                            id: row.uuid
                        }
                    })
                        .then(res => {
                            this.$message({
                                type: 'success',
                                message: '删除成功!'
                            });
                        })
                        .catch(() => {
                            this.$message({
                                type: 'info',
                                message: '删除失败'
                            });
                        });
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
        },
        postForm() {
            if (this.form.uuid != '') {
                Vue.axios({
                    method: 'post',
                    url: 'http://192.168.1.106:65221/api/v1/host/applyinfo/update',
                    data: {
                        uuid: this.form.uuid,
                        size: this.size,
                        expiration_at: this.expiration
                    }
                })
                    .then(res => {
                        this.$message({
                            type: 'success',
                            message: '修改成功!'
                        });
                    })
                    .catch(res => {
                        this.$message({
                            type: 'info',
                            message: '删除失败' + res.data.message
                        });
                    });
            } else {
                Vue.axios({
                    method: 'post',
                    url: 'http://192.168.1.106:65221/api/v1/bloomfilter/apply',
                    data: {
                        size: this.size,
                        expiration: this.expiration
                    }
                })
                    .then(res => {
                        this.$message({
                            type: 'success',
                            message: '创建成功!'
                        });
                    })
                    .catch(res => {
                        this.$message({
                            type: 'info',
                            message: '创建失败' + res.data.message
                        });
                    });
            }
            this.dialogFormVisible = false;
        },
        handleCurrentChange(size) {
            this.page.currentPage = size;
            this.getTableData();
        },
        handleSizeChange(size) {
            this.page.currentSize = size;
            this.getTableData();
        }
    }
};
</script>

<style>
.grid-content {
    border-radius: 4px;
    min-height: 36px;
}
</style>