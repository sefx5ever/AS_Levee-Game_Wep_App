<template>
	<view>
		<swiper :style="{height:styleSetting.windowHeight+'px'}" :indicator-dots="true" :autoplay="false">
			<swiper-item id="ruleFlood">
				<view>
					<view class="ruleTitile":style="{height:styleSetting.otherHeight+'px'}">Situation : Flood</view>
					<view class="ruleText" :style="{height:styleSetting.textHeight+'px'}">
						<view class="ruleTextTitle">IF ?! FLoOdInG</view>
						<view class="ruleTextSubTitle">
							<view class="ruleTextSubText">1. If all the players build levees, ZERO loss will be incurs. </view>
							<view class="ruleTextSubText">2. The more people build levee, the lost will be charge increasingly!</view>
						</view>
						<view class="ruleTextBasic">
							<view class="ruleTextBasicHint" :style="{width:styleSetting.windowWidth*1.8+'px'}">HINT</view>
							<view class="ruleTextBasicText"> Game start with 1000 dollar, you only available to opt "Build" by spending 5 dollar or "Not Build"</view>	
						</view>
					</view>
				</view>
			</swiper-item>
			<swiper-item id="ruleTsunami">
				<view>
					<view class="ruleTitile" :style="{height:styleSetting.otherHeight+'px'}">Situation : Tsunami</view>
					<view class="ruleText" :style="{height:styleSetting.textHeight+'px'}">
						<view class="ruleTextTitle">IF ?!? TsUNaMi</view>
						<view class="ruleTextSubTitle">
							<view class="ruleTextSubText">1. All players will be charge with NO reason ! </view>
							<view class="ruleTextSubText">2. The more people build levee, the lost will be charge increasingly!</view>
							<view class="ruleTextSubText">3. If you build levee, the cost also will count so. </view>
						</view>
					</view>
					<button class="ruleButton" @click="nextGame()">Start Game !</button>
				</view>
			</swiper-item>
		</swiper>
		<view class="ruleShow" :style="{height:styleSetting.otherHeight*1.5+'px'}">
			<view class="ruleShowOption" :style="{width:styleSetting.windowWidth+'px'}" @click="getTabPosition(0)">Flood</view>
			<view class="ruleShowOption" :style="{width:styleSetting.windowWidth+'px'}" @click="getTabPosition(1)">Tsunami</view>
		</view>
	</view>
</template>

<script>
	import * as firebase from '@/common/app.js';
	import '@/common/firestore.min.js';
	import '@/common/function.min';
	
	var _self;
	
	export default{
		data() {
			return {
				// tabIndex: 0,
				styleSetting: {
					windowHeight: '',
					windowWidth: '',
					textHeight: ''
				},
				userId: '',
				matchId: '',
				situationId: ''
			}
		},
		onLoad(data) {
			_self = this;
			
			this.pixelCalculation();
			this.userId = data.userId;
			this.matchId = data.matchId;
			this.situationId = data.situationId;
			
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
		// onReady() {
		// 	this.getTabPosition(0);
		// },
		methods: {
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleSetting.windowHeight = phoneData.windowHeight*0.925;
				this.styleSetting.windowWidth = phoneData.windowWidth / 2;
				this.styleSetting.textHeight = phoneData.windowHeight*0.85;
				this.styleSetting.otherHeight = phoneData.windowHeight*0.05;
				uni.onWindowResize((res) => {
					this.pixelCalculation();
				});
			},
			// getTabPosition: function(option){
			// 	this.tabIndex = option;
			// },
			nextGame: function() {
				if (_self.situationId == 'SITN1') {
					uni.redirectTo({
						url: '../game/game_with_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+true
					});
				} else if (_self.situationId == 'SITN2') {
					uni.redirectTo({
						url: '../game/game_without_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+true
					});
				} else if (_self.situationId == 'SITN3') {
					uni.redirectTo({
						url: '../game/game_with_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+false
					});
				} else if (_self.situationId == 'SITN4') {
					uni.redirectTo({
						url: '../game/game_without_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+false
					});
				} else {
					uni.showModal({
						title: 'NOTICE',
						content: 'Network connection failed, please restart the game!',
						confirmText: "RESTART Now",
						showCancel: false,
						complete() {
							uni.redirectTo({
								url: '../login/login'
							})
						}
					})
				}
			}
		}
	}
</script>

<style lang="scss">
	@import "@/static/rule.scss";
</style>
