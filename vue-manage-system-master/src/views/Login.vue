<template>
    <div class="login-wrap">
        <div class="ms-login">
            <div class="ms-title">博物馆后台管理系统</div>
            <el-form :model="param" :rules="rules" ref="login" label-width="0px" class="ms-content">
                <el-form-item prop="username">
                    <el-input v-model="param.username" placeholder="username">
                        <template #prepend>
                            <el-button icon="el-icon-user"></el-button>
                        </template>
                    </el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input type="password" placeholder="password" v-model="param.password"
                        @keyup.enter="submitForm()">
                        <template #prepend>
                            <el-button icon="el-icon-lock"></el-button>
                        </template>
                    </el-input>
                </el-form-item>
                <div v-if="user.isVisible" class="user-error-remind">您输入的账户或密码有误</div>
                <div class="login-btn">
                    <el-button type="primary" @click="submitForm()">登录</el-button>
                </div>
                <!-- <p  >Tips : 用户名和密码随便填。</p> -->
                <!-- <router-link class="login-tips"  to="register">点我注册</router-link> -->
            </el-form>
        </div>
    </div>
</template>

<script>
import { ref, reactive } from "vue";
import { useStore } from "vuex";
import { useRouter } from "vue-router";
import { ElMessage } from "element-plus";
import { sign } from "../utils/api" 
import { searchuser } from "../api/user";
import user from "../store/modules/user";
// import {setToken} from "../composables/auth"
export default {
    setup() {
        const router = useRouter();
        const param = reactive({
            username: "admin",
            password: "aaa111",
            
        });

        const rules = {
            username: [
                {
                    required: true,
                    message: "请输入用户名",
                    trigger: "blur",
                },
            ],
            password: [
                { required: true, message: "请输入密码", trigger: "blur" },
            ],
        };
        const roleList = [
            {id: 1, role: '超级管理员'},
            {id: 2, role: '藏品管理员'},
            {id: 3, role: '安保管理员'},
        ]
        const user = reactive({
            name: '',
            roleName: '',
            isVisible: false
        })
        const getUserInfo = async (id) => {
            try {
                const res = await searchuser(id);
                user.name = res.data.name;
                localStorage.setItem('ms_userName',user.name)
                const findedObj = roleList.find(item => item.id === res.data.role);
                user.roleName = findedObj.role;
            } catch (error) {
                console.error(error);
            }
            };

            const login = ref(null)
            const submitForm = () => {
            login.value.validate(async () => {
                try {
                const res = await sign(param.username, param.password, true);
                // console.log(res);
                const token = res.data.data.token;
                localStorage.setItem("token", token);
                const userId = res.data.data.id;
                await getUserInfo(userId);
                localStorage.setItem("ms_roleName", user.roleName);

                ElMessage.success("登录成功");
                user.isVisible = false;
                router.push("/");
                } catch (error) {
                console.error(error);
                user.isVisible = true;
                ElMessage.error("登录失败");
                }
            });
            };


        const store = useStore();
        store.commit("clearTags");

        return {
            user,
            param,
            rules,
            login,
            submitForm,
        };
    },
};
</script>

<style scoped>
.login-wrap {
    position: relative;
    width: 100%;
    height: 100%;
    background-image: url(../assets/img/login-bg.jpg);
    background-size: 100%;
}
.ms-title {
    width: 100%;
    line-height: 50px;
    text-align: center;
    font-size: 20px;
    color: #fff;
    border-bottom: 1px solid #ddd;
}
.ms-login {
    position: absolute;
    left: 50%;
    top: 50%;
    width: 350px;
    margin: -190px 0 0 -175px;
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.3);
    overflow: hidden;
}
.ms-content {
    padding: 30px 30px;
}
.login-btn {
    text-align: center;
}
.login-btn button {
    width: 100%;
    height: 36px;
    margin-bottom: 10px;
}
.login-tips {
    font-size: 12px;
    line-height: 30px;
    color: #fff;
}
.user-error-remind {
    color: red;
    font-size: 14px;
    margin-bottom: 10px;
    font-weight: 700;
}
</style>