<template>
	<view class="content">
		<image class="logo" src="/static/center/portrate.png"></image>
 		<view class="text-area">
			<text class="title">{{title}}</text>
		</view> 
		<section>
			<input class="uni-input" placeholder="请输入手机号" v-model="userPhone"></input>
		</section>
		<section>
			<input class="uni-input" type="password" placeholder="请输入密码" v-model="password"></input>
		</section>
		<section>
			<button type="button" @click="onLoginClicked">登录</button>
		</section>
		<section>
			<button type="button" @click="onRegister">注册</button>
		</section>
	</view>
</template>

<script>
	import {getStatic, postLoginIn, postJoinIn, getUserInfo} from "../../fetch/api.js";
	import {getEncryptedPassword} from "../../common/encrypt.js";
	import {
		mapMutations
	} from 'vuex';
	export default {
		data() {
			return {
				title: 'WELCOME',
				userInfo: '',
				userPhone: '',
				password: '',
				loginStatus: '未登录'
			}
		},
		onLoad() {

		},
		methods: {
			onLoginClicked(){
				let t_name = this.userPhone
				let t_pwd = this.password
				this.title="登录中..."
				postLoginIn(t_name, t_pwd).then((res)=>{
					if(res.st==0){
						this.title="登录成功\n"
						getUserInfo(t_name).then((res)=>{
							this.userInfo=res.data.userName
							console.log("res")
							console.log(res.data);
							this.login(res.data);
							uni.navigateBack();
						})
					}
					else if(res.st==1)
						this.title="登录失败"
				});
			},
			onRegister(){
				uni.navigateTo({
					url:"../center/register"
				})
			},
			...mapMutations(['login'])
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
