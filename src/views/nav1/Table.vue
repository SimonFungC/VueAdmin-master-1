<template>
    <section>
        <el-col style="display: inline-block;text-align: center">
            <div style="margin-top: 50px;width: 25%;float: left">
                <div align="center" style="color:#97a8be;margin-bottom: 5px">目标图像(仅支持.jpg文件)</div>
                <el-form :inline="true">
                    <!--图片上传-->
                    <el-form-item>
                        <el-upload
                                class="avatar-uploader"
                                action="http://127.0.0.1:8080/pic"
                                :on-success="handleAvatarSuccess"
                                :show-file-list="false">
                            <img v-if="this.mbtx" :src="this.mbtx" class="avatar">
                            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                        </el-upload>
                        <el-select v-model="kuType" size="small" style="width: 140px" placeholder="请选择裤子类别" @change="currentType">
                            <el-option label="吊裆裤" value="hlk"></el-option>
                            <el-option label="阔腿裤" value="ktk"></el-option>
                            <el-option label="喇叭裤" value="lbk"></el-option>
                            <el-option label="小脚裤" value="xjk"></el-option>
                            <el-option label="直筒裤" value="ztk"></el-option>
                        </el-select>
                        <el-button icon="el-icon-search" size="small" @click="getKus">搜索</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <div style="float: right;width: 75%;margin-top:50px;">
                <!--显示-->
                <el-row :gutter="20" style="color:#97a8be;margin-bottom: 5px;">
                    <el-col :span="8">相似度：{{images[this.preImageIndex].hammingDistance}}</el-col>
                    <el-col :span="8">相似度：{{images[this.onImageIndex].hammingDistance}}</el-col>
                    <el-col :span="8">相似度：{{images[this.postImageIndex].hammingDistance}}</el-col>
                </el-row>

                <el-row :gutter="20" style="color:#97a8be;margin-bottom: 5px;" align="center">
                    <el-col :span="8" align="center"><img v-if="images[this.preImageIndex].imgUrl"
                                                          :src="images[this.preImageIndex].imgUrl"
                                                          class="avatar">
                        <i v-else class="el-icon-picture avatar-uploader-icon"></i></el-col>
                    <el-col :span="8" align="center"><img v-if="images[this.onImageIndex].imgUrl"
                                                          :src="images[this.onImageIndex].imgUrl"
                                                          class="avatar">
                        <i v-else class="el-icon-picture avatar-uploader-icon"></i></el-col>
                    <el-col :span="8" align="center"><img v-if="images[this.postImageIndex].imgUrl"
                                                          :src="images[this.postImageIndex].imgUrl"
                                                          class="avatar">
                        <i v-else class="el-icon-picture avatar-uploader-icon"></i></el-col>
                </el-row>

                <div>
                    <el-button-group style="vertical-align: bottom;margin-top: 10px;">
                        <el-button @click='change(index,"pre")'><i
                                class="el-icon-arrow-left el-icon-arrow-left"></i>上一页
                        </el-button>
                        <el-button @click='change(index,"next")'>下一页<i
                                class="el-icon-arrow-right el-icon--right"></i></el-button>
                    </el-button-group>
                </div>
            </div>
        </el-col>

    </section>
</template>

<script>
    import {Loading} from 'element-ui';
    import {
        getKus,
        changeName
    } from '../../api/api';

    export default {
        data() {
            return {
                kuType: '',//查询裤子类别
                images: [
                    {
                        imgUrl: '',
                        hammingDistance: '',
                        index: ""
                    }
                ],
                mbtx: '',
                imageIndex: 0,
                preImageIndex: 0,
                onImageIndex: 0,
                postImageIndex: 0,
                //编辑界面数据
                editForm: {
                    id: 0,
                    companyName: '',
                    companyAddress: '杭州',
                    submitTime: new Date(),
                    submitType: '网申',
                    progress: '初筛',
                    comment: ''
                }
            }
        },
        methods: {
            handleAvatarSuccess: function (res, file) {
                this.mbtx = URL.createObjectURL(file.raw);
            },
            currentType: function () {
                var type = this.kuType;
                changeName(type).then((res) => {
                    if (res.status === 200) {
                        this.mbtx = require("E:\\IDEA-1.6\\img\\mbtx\\" + this.kuType + ".jpg");
                    }
                }).catch(error => {
                    this.listLoading = false;
                })
            },
            getKus: function () {
                this.imageIndex = 0;
                this.preImageIndex = 3 * this.imageIndex;
                this.onImageIndex = 3 * this.imageIndex + 1;
                this.postImageIndex = 3 * this.imageIndex + 2;
                var kuType = this.kuType;
                if (kuType !== "") {
                    let loadingInstance = Loading.service({fullscreen: true, text: "搜索中"});
                    getKus(kuType).then((res) => {
                        console.log(res.data);
                        if (res.status === 200) {
                            if (res.data.length > 0) {
                                this.images = [];
                            }
                            for (var i = 0; i < res.data.length; i++) {
                                var temp = {
                                    imgUrl: require("@/assets/img/" + this.kuType + "/" + this.kuType + " (" + res.data[i].number + ").jpg"),
                                    hammingDistance: res.data[i].hammingDistance,
                                    index: i
                                }
                                this.images.push(temp);
                                loadingInstance.close();
                            }
                        }
                    }).catch(error => {
                        loadingInstance.close();
                        this.$message({
                            message: '查询失败',
                            type: 'error'
                        });
                    })
                } else {
                    this.$message({
                        message: '请选择查询类别',
                        type: 'warning'
                    });
                    this.images = [
                        {
                            imgUrl: require("@/assets/logo4.png"),
                            hammingDistance: '',
                            index: ""
                        }
                    ];
                }
            },
            change(i, type) {
                if (type === "pre") {
                    if (this.imageIndex < 1) {
                        this.$message({
                            message: '当前已是图库第一张',
                            type: 'warning'
                        });
                        return;
                    }
                    this.imageIndex--;
                }
                if (type === "next") {
                    if (this.imageIndex > 98) {
                        this.$message({
                            message: '当前已是图库最后一张',
                            type: 'warning'
                        });
                        return;
                    }
                    this.imageIndex++;
                }
                this.preImageIndex = 3 * this.imageIndex;
                this.onImageIndex = 3 * this.imageIndex + 1;
                this.postImageIndex = 3 * this.imageIndex + 2;
            }

        }
    }

</script>

<style>

    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }

    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }

    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 200px;
        height: 200px;
        line-height: 200px;
        text-align: center;
    }

    .avatar {
        width: 200px;
        /*height: 300px;*/
        display: block;
    }

    .el-row {
        margin-bottom: 20px;

    &
    :last-child {
        margin-bottom: 0;
    }

    }
    .el-col {
        border-radius: 4px;
    }

    .bg-purple-dark {
        background: #99a9bf;
    }

    .bg-purple {
        background: #d3dce6;
    }

    .bg-purple-light {
        background: #e5e9f2;
    }

    .grid-content {
        border-radius: 4px;
        min-height: 36px;
    }

    .row-bg {
        padding: 10px 0;
        background-color: #f9fafc;
    }
</style>