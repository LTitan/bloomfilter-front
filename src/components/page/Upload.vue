<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-calendar"></i> 文件上传
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="content-title">拖拽上传</div>
            <el-radio v-model="radio" label="1">写入</el-radio>
            <el-radio v-model="radio" label="2">查询</el-radio>
            <el-input placeholder="请输入key" v-model="input" clearable style="width: 160px;"></el-input>&#8195;
            <el-button type="primary" @click="submitFile">提交</el-button>
            <br />
            <br />
            <el-upload
                class="upload-demo"
                drag
                action="http://192.168.1.106:65221/api/v1/file/upload"
                :limit="1"
                :on-success="uploadSuccess"
                :on-remove="remove"
                multiple
            >
                <i class="el-icon-upload"></i>
                <div class="el-upload__text">
                    将文件拖到此处，或
                    <em>点击上传</em>
                </div>
            </el-upload>
            <div class="el-upload__tip" slot="tip">
                <span>只能上传txt文件，且不超过500MB</span>
            </div>
        </div>
    </div>
</template>

<script>
import VueCropper from 'vue-cropperjs';
import Vue from 'vue';
export default {
    name: 'upload',
    data: function() {
        return {
            defaultSrc: require('../../assets/img/img.jpg'),
            fileList: [],
            imgSrc: '',
            cropImg: '',
            dialogVisible: false,
            fileName: '',
            radio: '1',
            input: ''
        };
    },
    components: {
        VueCropper
    },
    methods: {
        uploadSuccess(res, file, fileList) {
            this.fileName = file.name;
        },
        remove() {
            this.fileName = '';
        },
        submitFile() {
            if (this.fileName === '') {
                this.$message('请上传文件');
                return;
            }
            if (this.input === '') {
                this.$message('请输入key');
                return;
            }
            Vue.axios({
                method: 'get',
                url: 'http://192.168.1.106:65221/api/v1/file/upload',
                params: {
                    file_name: this.fileName,
                    status: this.radio,
                    key: this.input
                }
            })
                .then(res => {
                    if (res.data.code === 2000) {
                        this.$message({
                            message: '写入成功',
                            type: 'success'
                        });
                    } else {
                        this.$message({
                            message: '写入失败',
                            type: 'warning'
                        });
                    }
                })
                .catch(res => {
                    this.$message('后台服务出错啦！!');
                });
        }
    },
    created() {}
};
</script>

<style scoped>
.content-title {
    font-weight: 400;
    line-height: 50px;
    margin: 10px 0;
    font-size: 22px;
    color: #1f2f3d;
}
.pre-img {
    width: 100px;
    height: 100px;
    background: #f8f8f8;
    border: 1px solid #eee;
    border-radius: 5px;
}
.crop-demo {
    display: flex;
    align-items: flex-end;
}
.crop-demo-btn {
    position: relative;
    width: 100px;
    height: 40px;
    line-height: 40px;
    padding: 0 20px;
    margin-left: 30px;
    background-color: #409eff;
    color: #fff;
    font-size: 14px;
    border-radius: 4px;
    box-sizing: border-box;
}
.crop-input {
    position: absolute;
    width: 100px;
    height: 40px;
    left: 0;
    top: 0;
    opacity: 0;
    cursor: pointer;
}
</style>