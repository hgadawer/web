<template>
    <a-form ref="registerFormRef" :model="formData" style="width: 65%;" layout="vertical" @finish="onRegister"
        :rules="rules">
        <a-form-item>
            <div class="title">账号注册</div>
        </a-form-item>
        <a-form-item name="email">
            <a-input v-model:value="formData.email" size="large" placeholder="邮箱" />
        </a-form-item>
        <a-form-item name="code">
            <a-input v-model:value="formData.code" size="large" style="width: 55%;" placeholder="验证码" />
            <a-button @click="onGetCode" size="large" style="width: 40%;float: right;" :loading="loading"
                :disabled="disabled">
                {{ buttonText }}</a-button>
        </a-form-item>
        <a-form-item name="password1">
            <a-input-password v-model:value="formData.password1" size="large" placeholder="密码" />
        </a-form-item>
        <a-form-item name="password2">
            <a-input-password v-model:value="formData.password2" size="large" placeholder="确认密码" />
        </a-form-item>
        <a-form-item>
            <a-button type="primary" html-type="submit" size="large" style="width: 100%;">注册</a-button>
        </a-form-item>
        <a-form-item>
            已有账号？<a @click="onLogin"> 直接登录</a>
        </a-form-item>
    </a-form>
</template>

<script setup>
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { userRegister, getVerifyCode } from '../api/user';
import { message } from 'ant-design-vue';

const router = useRouter()

// 用户注册
const formData = reactive({
    email: '',
    code: '',
    password1: '',
    password2: '',
});

// 表单校验
const rules = {
    email: [{
        required: true,
        message: '请输入邮箱!',
        trigger: 'blur',
    }, {
        pattern: /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/,
        message: '邮箱格式不正确',
        trigger: 'blur',
    }],
    code: [{ required: true, message: '请输入验证码!' }],
    password1: [{ required: true, message: '请输入密码!' }],
    password2: [{ required: true, message: '请输入密码!' }],
};

const loading = ref(false)
const disabled = ref(false)
const registerFormRef = ref()
const buttonText = ref('获取验证码')

const onRegister = () => {
    registerFormRef.value.validateFields().then(() => {
        if (formData.password1 != formData.password2) {
            message.warn('密码不一致');
            return
        }
        let param = {
            email: formData.email,
            code: formData.code,
            password: formData.password2,
        }
        userRegister(param).then((res) => {
            if (res.data.code == 200) {
                message.success('注册成功');
                onLogin()
            }
            if(res.data.code == 500){
                message.warn(res.data.info)
            }
            if (res.data.code == 10001) {
                message.warn(res.data.msg);
            }
            if (res.data.code == 10005) {
                message.error(res.data.msg);
            }
        })
    })
};

// 获取验证码
const onGetCode = () => {
    if (formData.email == '') {
        message.warn('邮箱不能为空')
        return
    }
    loading.value = true
    let param = {
        email: formData.email
    }
    getVerifyCode(param).then((res) => {
        if (res.data.code == 200) {
            loading.value = false
            disabled.value = true
            buttonText.value = '验证码已发送'
        }
        if (res.data.code == 500) {
            loading.value = false
            message.error('验证码发送失败')
        }
    })
}

// 跳转到登录页面
const onLogin = () => {
    router.push("/login")
}
</script>

<style scoped>
.title {
    color: #303133;
    font-size: 30px;
    line-height: 65px;
    font-weight: 500;
}
</style>