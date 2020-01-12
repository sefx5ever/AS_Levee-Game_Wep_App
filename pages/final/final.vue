<template>
	<view class="finalFrame" :style="{height:styleSetting.windowHeight+'px'}">
		<view class="finalLayout">
			<view class="finalTitle">ROUND 1</view>
			<view class="finalTopPage" :style="{height:styleSetting.windowHeight*0.35+'px'}">
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in rankList" :key='index'>
						{{item}}
					</view>
				</view>
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in winnerList1" :key='index'>
						{{item}}
					</view>
				</view>
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in pointList1" :key='index'>
						{{item}}
					</view>
				</view>
			</view>
			<view class="finalTitle">ROUND 2</view>
			<view class="finalBottomPage" :style="{height:styleSetting.windowHeight*0.35+'px'}">
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in rankList" :key='index'>
						{{item}}
					</view>
				</view>
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in winnerList2" :key='index'>
						{{item}}
					</view>
				</view>
				<view class="finalSection" :style="{width:styleSetting.windowWidth/3.5+'px'}">
					<view v-for="(item,index) in pointList2" :key='index'>
						{{item}}
					</view>
				</view>
			</view>
		</view>
		<view class="finalButton" @click="backLogin()">
			<view>DONE GAME</view>
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
				rankList: ['Ranking','Top1 :','Top2 :','Top3 :'],
				winnerList1: ['Player'],
				winnerList2: ['Player'],
				pointList1: ['Amount'],
				pointList2: ['Amount'],
				matchId: '',
				nextRound: '',
				userId: '',
				situationId: '',
				styleSetting: {
					windowHeight: '',
					windowWidth: ''
				}
			}
		},
		onLoad(data) {
			_self = this;
			
			this.pixelCalculation();
			// 獲取matchId以進行之後程序
			this.matchId = data.matchId;
			this.nextRound = data.nextRound;
			this.userId = data.userId;
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
			// 加載完成後，獲取成績
			this.checkResult();
		},
		methods: {
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleSetting.windowHeight = phoneData.windowHeight;
				this.styleSetting.windowWidth = phoneData.windowWidth;
				uni.onWindowResize((res) => {
					this.pixelCalculation();
				});
			},
			getKeyByValue: function(winner,object, value) { 
			    var key = Object.keys(object).find(key => object[key] === value); 
				if(winner == 'winnerList1'){
					_self.winnerList1.push(key)
				} else {
					_self.winnerList2.push(key)
				}
			},
			checkResult: function() {
				var db = firebase.firestore();
				var resultConnect = db.collection('PlayAndScore').doc(_self.matchId);
				resultConnect.get().then(function(doc) {
					if (doc.exists) {
						// 將round1及round2成績拆開
						var round1_result = doc.data().round01_score;
						var round2_result = doc.data().round02_score;
						_self.rankList = ['Ranking','Top1 :','Top2 :','Top3 :'];
						_self.winnerList1 = ['Player'];
						_self.winnerList2 = ['Player'];
						_self.pointList1 = ['Amount'];
						_self.pointList2 = ['Amount'];
						// 針對round1進行成績排序
						var round1_arrange = Object.values(round1_result).sort().reverse();
						var round2_arrange = Object.values(round2_result).sort().reverse();
						round1_arrange.forEach(function(key) {
							_self.pointList1.push(key);
							_self.getKeyByValue('winnerList1',round1_result,key)
						});
						// 針對round2進行成績排序
						round2_arrange.forEach(function(key) {
							_self.pointList2.push(key);
							_self.getKeyByValue('winnerList2',round2_result,key)
						});
					} else {
						// doc.data() will be undefined in this case
						console.log("No such document!");
					}w
				}).catch(function(error) {
					console.log("Error getting document:", error);
				});
			},
			backLogin: function() {
				if (_self.pointList2.length > 1) {
					uni.redirectTo({
						url: '../login/login'
					});
				} else if (_self.situationId == 'SITN1') {
					uni.redirectTo({
						url: '../game/game_with_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+false
					});
				} else if (_self.situationId == 'SITN2') {
					uni.redirectTo({
						url: '../game/game_without_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+false
					});
				} else if (_self.situationId == 'SITN3') {
					uni.redirectTo({
						url: '../game/game_with_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+true
					});
				} else if (_self.situationId == 'SITN4') {
					uni.redirectTo({
						url: '../game/game_without_chat?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&switch='+true
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
	@import '@/static/final.scss';
</style>
