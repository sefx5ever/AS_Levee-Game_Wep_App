<template>
	<view >
		<view class="loginPage" :style="{height:styleSetting.windowHeight+'px'}">
			<view class="loginLogo">
				<image id="loginImage" class="loginTitle"  :style="{width:styleSetting.windowWidth+'px'}" src="/static/picture/Background_login.png" mode="aspectFill"></image>
				<view class="loginTitle">Social Experiment Platform</view>
			</view>
			<view class="loginType">
				<view class="loginTypeWord">USERNAME<input class="loginInput" type="text" placeholder="Username" placeholder-style="text-align:center" maxlength='20' v-model="data_username"></input></view>
				<view class="loginTypeWord">PASSWORD<input class="loginInput" password="true" placeholder="Password" placeholder-style="text-align:center" maxlength='20' v-model="data_password"></input></view>
			</view>
			<view class="loginPress">
				<view class="loginButton">Sign Up</view>
				<view id="loginButtonMain" class="loginButton" @click="nextStory()">Sign In</view>
			</view>
		</view>
	</view>
</template>

<script>
	import * as firebase from '@/common/app.js';
	import '@/common/firestore.min.js';
	import '@/common/function.min';
	
	var _self;
	
	export default{
		data () {
			return {
				styleSetting: {
					windowHeight: '',
					windowWidth: ''
				},
				data_password: '',
				data_username: '',
				matchId: '',
				situationId: ''
			}
		},
		onLoad() {
			_self = this;
			this.pixelCalculation();
			
			var firebaseConfig = {
				apiKey: "AIzaSyA2P0T3COrV1DHStIEBLnJoIuY4LunGmNg",
				authDomain: "fir-test-84116.firebaseapp.com",
				databaseURL: "https://fir-test-84116.firebaseio.com",
				projectId: "fir-test-84116",
				storageBucket: "fir-test-84116.appspot.com"
			};
			
			// firebase.initializeApp(firebaseConfig);
			
			//firebase 已經init過在login的時候所以用下面的方式
			if (!firebase.apps.length) {
				firebase.initializeApp(firebaseConfig);
				console.log(firebase.app().name);
			};
		},
		methods:{
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleSetting.windowHeight = phoneData.windowHeight*0.973;
				this.styleSetting.windowWidth = phoneData.windowWidth*4;
				uni.onWindowResize((res) => {
					this.pixelCalculation();
				});
			},
			// 檢查所輸入username和password是否一致或是管理員帳號，並給予反饋
			nextStory: function(){
				uni.showLoading({
					title: 'Loging...'
				})
				if(_self.data_username.trim() == '' && _self.data_password.trim() == ''){
					uni.hideLoading();
					uni.showModal({
						title: 'NOTICE',
						content: 'Username or Password cannot be empty!',
						confirmText: 'OK',
						showCancel: false
					});
				}
				var db = firebase.firestore();
				var loginConnect = db.collection('User_test').doc(_self.data_username);
				loginConnect.get().then(function(doc) {
					if(doc.exists){
						if (doc.data().password = _self.data_password){
							_self.matchId = doc.data().match;
							_self.situationId = doc.data().situation;
							uni.redirectTo({
								url: '../story/story?userId='+_self.data_username+'&matchId='+_self.matchId+'&situationId='+_self.situationId
							});
							uni.hideLoading();
						} else {
							uni.hideLoading();
							uni.showModal({
								title: 'NOTICE',
								content: 'Wrong password or username!',
								confirmText: 'OK',
								showCancel: false
							});
						}
					} else {
						// 檢查是否為管理員帳號
						if (_self.data_username == 'admin' && _self.data_password == 'admin'){
							uni.hideLoading();
							uni.redirectTo({
								url: '../backenduser/backenduser'
							});
						} else {
							uni.hideLoading();
							uni.showModal({
								title: 'NOTICE',
								content: 'Please enter the right username and password!',
								confirmText: 'OK',
								showCancel: false
							});
							_self.data_password = '';
						}
					}
				}).catch(function(error) {
					uni.hideLoading();
					console.log("Error getting document:", error);
				});

			}
		}
	}
</script>

<style lang="scss">
	@import '@/static/login.scss';
</style>
