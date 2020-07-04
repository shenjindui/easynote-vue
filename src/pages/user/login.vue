<template>
    <!-- 登录 -->
    <div class="login">
        <div class="container">
            <div class="row">
                <div class="col-md-8 m-auto">
                    <h1 class="display-4 text-center">登录</h1>
                    <p class="lead text-center">用正确的账号登录</p>
                    <form @submit.prevent="submit" autocomplete="off" method="post">
                        <div class="form-group">
                            <input v-model="user.email" :class="{'is-invalid':errors.email}" autocomplete="off" type="email" class="form-control form-control-lg"
                                   placeholder="邮箱地址" name="email" />
                            <div v-if="errors.email" class="invalid-feedback">{{errors.email}}</div>
                        </div>
                        <div class="form-group">
                            <input v-model="user.password" :class="{'is-invalid':errors.password}" autocomplete="off" type="password" class="form-control form-control-lg"
                                   placeholder="密码" name="password" />
                            <div v-if="errors.password" class="invalid-feedback">{{errors.password}}</div>

                        </div>
                        <input value="登录" type="submit" class="btn btn-info btn-block mt-4" />
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    // token存入的localstorage 需要通过jwt-decode解析
    //import jwt_decode from 'jwt-decode'
    export default {
        name: "Login",
        data() {
            return {
                user: {
                    email: "",
                    password: ""
                },
                errors: {}
            }
        },
        methods: {
            submit() {
                // console.log(this.user.email)
                this.$axios.post("http://localhost:5000/api/users/login", this.user)
                    .then(res => {
                        // console.log(res.data)
                        // 存储token 到localstore
                        // const token = res.data.token与下面一样
                        const {
                            token
                        } = res.data
                        window.localStorage.setItem("jToken", token);

                        // 解析token
                        //const decoded = jwt_decode(token);
                        // console.log(decoded)

                        // 分发action 更改store 的state
                        ///this.$store.dispatch('setIsAuthenticated',!this.isEmpty(decoded))
                        //this.$store.dispatch('setUser',decoded);
                        // 页面跳转
                        this.$router.push('/dashboard')

                    }).catch(err => {
                    // console.log(err.response.data);
                    this.errors = err.response.data
                })
            },
            isEmpty(value) {
                return value === undefined || value === null ||
                    (typeof value === "object" && Object.keys(value).length === 0) ||
                    (typeof value === "string" && value.trim().length === 0)
            }
        }
    }
</script>

<style scoped="scoped">
</style>