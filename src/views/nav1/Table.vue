<template>
    <section>


        <el-col style="display: inline-block;text-align: center">
            <div style="margin-top: 50px;width: 50%;float: left">
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
                        <el-select v-model="kuType" size="small" style="width: 140px" placeholder="请选择裤子类别" clearable
                                   @change="currentType">
                            <el-option label="哈伦裤" value="hlk"></el-option>
                            <el-option label="阔腿裤" value="ktk"></el-option>
                            <el-option label="喇叭裤" value="lbk"></el-option>
                            <el-option label="小脚裤" value="xjk"></el-option>
                            <el-option label="直筒裤" value="ztk"></el-option>
                        </el-select>
                        <el-button icon="el-icon-search" size="small" @click="getKus">搜索</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <div style="float: right;width: 50%;margin-top:50px;">
                <!--显示-->
                <el-col>
                    <!--<div v-for="(imageInfo,index) in images">-->
                    <div align="center" style="color:#97a8be;margin-bottom: 5px">
                        相似度：{{images[this.imageIndex].hammingDistance}}
                    </div>
                    <el-upload
                            class="avatar-uploader"
                            action=""
                            disabled="true"
                            :show-file-list="false">
                        <img v-if="images[this.imageIndex].imgUrl" :src="images[this.imageIndex].imgUrl" class="avatar">
                        <i v-else class="el-icon-picture avatar-uploader-icon"></i>
                    </el-upload>
                    <!--<img v-if="images[this.imageIndex].imgUrl" :src="images[this.imageIndex].imgUrl" class="avatar">
                    <i v-else class="el-icon-picture avatar-uploader-icon"></i>-->
                    <div>
                        <el-button-group style="vertical-align: bottom;margin-top: 10px;">
                            <el-button @click='change(index,"pre")'><i
                                    class="el-icon-arrow-left el-icon-arrow-left"></i>上一张
                            </el-button>
                            <el-button @click='change(index,"next")'>下一张<i
                                    class="el-icon-arrow-right el-icon--right"></i></el-button>
                        </el-button-group>
                    </div>
                    <!--</div>-->
                </el-col>
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
                if (type == "") {
                    type = "hlk";
                }
                changeName(type).then((res) => {
                    if (res.status === 200) {
                        this.mbtx = require("@/assets/img/mbtx/" + this.kuType + ".jpg");
                    }
                }).catch(error => {
                    this.listLoading = false;
                })
            },
            getKus: function () {
                var kuType = this.kuType;
                if (kuType != "") {
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
                    if (this.imageIndex > 298) {
                        this.$message({
                            message: '当前已是图库最后一张',
                            type: 'warning'
                        });
                        return;
                    }
                    this.imageIndex++;
                }
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
        width: 300px;
        height: 300px;
        line-height: 300px;
        text-align: center;
    }

    .avatar {
        width: auto;
        height: 300px;
        display: block;
    }
</style>