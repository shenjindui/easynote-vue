<template>
    <div style="padding-left: 1%;"
         v-loading.fullscreen.lock="loading"
         element-loading-text="拼命加载中"
         element-loading-spinner="el-icon-loading"
         element-loading-background="rgba(0, 0, 0, 0.8)"
    >


        <!--编辑器-->
        <el-col :span="21" style="padding-right: 1%;">
            <el-row>

                <div style="text-align: center;margin-bottom: 1%;margin-top: 1%;font-size: 30px">
                    <el-input v-model="article.title" placeholder="标题"></el-input>
                </div>

            </el-row>
            <el-row>
                <mavon-editor style="height:700px;" v-model="article.mkValue"
                              :ishljs="true"
                              ref="md"
                              @imgAdd="ImgAdd"
                              @imgDel="ImgDel"
                              @save="Save"/>
            </el-row>
            <el-row style="text-align: center">
                <el-link type="success">感谢使用</el-link>
            </el-row>
        </el-col>


        <!--        编辑器侧边信息-->
        <el-col :span="3" style="padding-top: 2%">

            <!--            日期-->
            <div style="text-align: center">
                创建日期: <i class="el-icon-date" style="color: deepskyblue">{{article.created_at}}</i><br>
                最近更新: <i class="el-icon-date" style="color: orange">{{article.updated_at}}</i>

            </div>
            <el-divider></el-divider>


            <!--          目录-->
            <div>
                目录:
                <el-cascader
                        filterable
                        :props="props"
                        v-model="article.dir_path"
                        :options="options"
                        clearable
                ></el-cascader>


            </div>
            <el-divider></el-divider>


            <!--           保存-->
            <div style="text-align: center">
                <el-button type="success" @click="FinishSave">保存文章</el-button>
                <el-link @click="DeleteCache">清空</el-link>
            </div>

        </el-col>

    </div>
</template>

<script>


    import request from "@/network/request";


    export default {
        name: "write",
        components: {},

        mounted() {
            if (this.$route.params.article) {
                this.article = this.$route.params.article;
            } else {
                this.loading = true;
                request({
                    url: "/article/temp_get"
                }).then(resp => {
                    this.article = resp.data.data;
                    this.loading = false;
                })
            }


        },
        beforeDestroy() {
            if (this.article.id !== 0 && this.article.id != null) {
                request({
                    url: "/article/temp_save",
                    method: 'post',
                    data: this.article
                }).then(resp => {
                    this.$message({
                        type: "warning",
                        message: resp.data.msg
                    });

                })
            }

        },


        data: function () {
            return {

                loading: false,
                //目录
                options: [],
                props: {
                    checkStrictly: true,
                    lazy: true,
                    lazyLoad(node, resolve) {
                        // eslint-disable-next-line no-console
                        request({
                            url: "/folder/sub_folder",
                            params: {
                                id: node.value,
                                // title: node.label
                            }
                        }).then(resp => {
                            resolve(resp.data.data);
                        });


                    }

                },


                article: {
                    id: null,
                    created_at: "0-0-0-0",
                    updated_at: "0-0-0-0",
                    title: "无标题",
                    dir_path: [],
                    mkValue: "",
                    folder_id: 0,
                },


            }


        },
        methods: {

            //点击保存事件
            FinishSave() {
                this.loading = true;
                this.article.mkValue = this.$refs.md.$data.d_value;
                request({
                    method: 'post',
                    url: '/article/update',
                    data: this.article
                }).then(resp => {
                    this.$message({
                        type: "success",
                        message: resp.data.msg
                    });
                    this.article = resp.data.data;
                    this.loading = false
                })
            },
            //编辑器
            //保存 Ctrl+S回调
            Save(mkValue) {

                this.article.mkValue = mkValue;
                this.loading = true;

                request({
                    url: "/article/update",
                    method: "post",
                    data: this.article
                }).then(resp => {
                    this.$message({
                        type: "success",
                        message: resp.data.msg
                    });
                    this.article = resp.data.data;
                    this.loading = false
                }).catch(err => {
                    console.log(err)
                })
            },

            //图片上传七牛云 图片名字唯一
            ImgAdd(pos, imgfile) {
                console.log(imgfile); //防止报错
                let data = new FormData();
                data.append('img', imgfile);
                this.loading = true;
                request({
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    },
                    method: 'post',
                    url: "/qiniu/img_upload",
                    data: data
                }).then(resp => {
                    this.$message({
                        type: "success",
                        message: resp.data.msg
                    });

                    this.$refs.md.$img2Url(pos, resp.data.data);
                    this.loading = false

                }).catch(err => {
                    this.$message({
                        type: "error",
                        message: err
                    })
                })
            },

            //图片从七牛云删除
            ImgDel(file) {
                this.loading = true;
                request({
                    url: "/qiniu/img_delete",
                    params: {
                        img_name: file[1].name
                    }
                }).then(resp => {
                    if (resp.data.code == 500) {
                        this.$message({
                            type: "error",
                            message: resp.data.msg
                        })
                    } else {
                        this.$message({
                            type: "success",
                            message: resp.data.msg
                        })
                    }

                    this.loading = false
                })
            },


//清空redis缓存
            DeleteCache() {
                request({
                    url: '/article/temp_delete',
                }).then(resp => {
                    this.$message({
                        type: "success",
                        message: resp.data.msg
                    });
                    this.article = {
                        id: null,
                        created_at: null,
                        updated_at: null,
                        title: "无标题",
                        dir_path: [],
                        mkValue: "",
                        folder_id: 0,

                    }

                })
            }


        }
    }
</script>

<style scoped>

</style>