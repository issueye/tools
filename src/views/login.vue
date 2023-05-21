<template>
	<div class="login-wrap">
		<div class="ms-login">
			<div class="ms-title">日志采集工具</div>
			<el-form :model="param" :rules="rules" ref="login" label-width="0px" class="ms-content">
				<el-form-item prop="username">
					<el-input v-model="param.username" placeholder="username">
						<template #prepend>
							<el-button :icon="User"></el-button>
						</template>
					</el-input>
				</el-form-item>
				<el-form-item prop="password">
					<el-input type="password" placeholder="password" v-model="param.password"
						@keyup.enter="submitForm(login)">
						<template #prepend>
							<el-button :icon="Lock"></el-button>
						</template>
					</el-input>
				</el-form-item>
				<div class="login-btn">
					<el-button type="primary" @click="submitForm(login)">登录</el-button>
				</div>
			</el-form>
		</div>
	</div>
</template>

<script setup name="login">
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { apiLogin } from '../api/user'



const router = useRouter();
const param = reactive({
	username: 'admin',
	password: '123456'
});

const rules = {
    username: [{required: true,message: '请输入用户名',trigger: 'blur'}],
	password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
};
const login = ref();
const submitForm = (e) => {
	if (!e) return;
	e.validate(async (valid) => {
		if (valid) {
			const info = {
				account: param.username,
				password: param.password,
			};
			// 调用后台接口
			let { data } = await apiLogin(info)
			// 如果登录成功
			console.log('data', data);
			if (data.code == 200) {
				ElMessage.success('登录成功');
				console.log('token', data.data.token);
				localStorage.setItem('token', data.data.token);
				router.push('/');
			}
		} else {
			ElMessage.error('登录失败');
			return false;
		}
	});
};

</script>

<style scoped>
.login-wrap {
	position: absolute;
	width: 100%;
	height: 100%;
	background-size: 100%;
}

.ms-title {
	width: 100%;
	line-height: 50px;
	text-align: center;
	font-size: 20px;
	color: #4c1f24;
	border-bottom: 1px solid #ddd;
}

.ms-login {
	position: absolute;
	left: 50%;
	top: 50%;
	width: 350px;
	margin: -190px 0 0 -175px;
	border-radius: 5px;
	background: rgb(21 235 245 / 18%);
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

</style>
