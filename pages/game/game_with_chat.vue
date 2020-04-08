<template>
	<view class="gamePage" :style="{height:styleSetting.windowHeight+'px'}">
		<view class="gameBackground" :style="{width:styleSetting.windowWidth+'px',height:styleSetting.windowHeight*0.485+'px'}">
			<view class="gameTitle" :style="{height:styleSetting.windowHeight*0.083+'px'}">{{currentRound+" "+currentTurn}}</view>
			<view>
				<view class="gameHistory" v-if='showHistory == "true"' @click='showPop()'>
					<view class="gameHistoryBorder">
						<uni-popup :show="showPopup" position="middle" mode="fixed">
							<view class="historyBoard" :style="{width:styleSetting.windowWidth*0.6+'px',height:styleSetting.windowHeight*0.65+'px'}">
								<view class="historyTitle">HISTORY</view>
								<view class="historySubTitle">Please choose to check player's record.</view>
								<view class="historyPlayer">{{playerList[1]}}</view>
								<uni-collapse>
									<uni-collapse-item>
										<view v-if="currentTurn != 'turn01'" v-for="(score,index) in 10" :key='index'>
											Round {{index+1}} : {{player1[0][index]}}
										</view>
									</uni-collapse-item>
								</uni-collapse>
								<view class="historyPlayer">{{playerList[2]}}</view>
								<uni-collapse>
									<uni-collapse-item>
										<view v-if="currentTurn != 'turn01'" v-for="(score,index) in 10" :key='index'>
											Round {{index+1}} : {{player2[0][index]}}
										</view>
									</uni-collapse-item>
								</uni-collapse>
								<view class="historyPlayer">{{playerList[3]}}</view>
								<uni-collapse>
									<uni-collapse-item>
										<view v-if="currentTurn != 'turn01'" v-for="(score,index) in 10" :key='index'>
											Round {{index+1}} : {{player3[0][index]}}
										</view>
									</uni-collapse-item>
								</uni-collapse>
								<button type="primary" @click="showPop()">Close</button>
							</view>
						</uni-popup>
					</view>
				</view>
				<view class="gameDetail" :style="{height:styleSetting.windowHeight*0.25+'px'}">
					<view class="gameDetailLayout">
						<view class="gameDetailSection">
							<view v-for="(item,index) in playerList" :key='index'>
								<view v-if="item == 'Player'">
									<view class="gameDetailSectionTitle">{{item}}</view>
								</view>
								<view v-if="item != 'Player'">
									{{item}}
								</view>
							</view>
						</view>
						<view class="gameDetailSection">
							<view v-for="(item,index) in moneyList" :key='index'>
								<view v-if="item == 'Money'">
									<view class="gameDetailSectionTitle">{{item}}</view>
								</view>
								<view v-if="item != 'Money'">
									{{item}}
								</view>
							</view>
						</view>
					</view>
					<view class="gameButton" :style="{width:styleSetting.windowWidth*0.75+'px'}">
						<view class="gameButtonFrame" :class="[isActive == true ? '.gameButtonHover' : '']" @click="sendBuild()">
							<view class="gameButtonBuild">Build</view>
							<view class="gameButtonBuild">( 5 Dollar )</view>
						</view>
						<view class="gameButtonFrame" :class="[isActive == true ? '.gameButtonHover' : '']" @click="sendNotBuild()">
							<view class="gameButtonNotBuild">Not Build</view>
						</view>
					</view>
				</view>
				<view class="gameCenter" :style="{height:styleSetting.windowHeight*0.15+'px'}">
					<view class="gameSeq">
						<view>{{emptySeq}}</view>
					</view>
					<view class="gameMoney">
						<view>Money: {{money}}</view>
					</view>
				</view>
			</view>
		</view>
		<uni-popup :show='showMessage' type="center" mode="fixed">
			<view class="scriptTitle">RESULT</view>
			<view class="scriptDecision">
				<view>You Choose {{currentDecision}} !</view>
				<view>[{{currentWeather}}]</view>
			</view>
			<view class="scriptDecoration">
				<view v-if="showScript[0]">You are SAFE ! Luckily you choose to build, you just spend {{lostAmount}} dollar.</view>
				<view v-if="showScript[1]">You have a bad luck, you lost {{lostAmount}} dollar</view>
				<view v-if="showScript[2]">OMG! This is unpredictable, you suddenly lost {{lostAmount}} dollar</view>
				<view v-if="showScript[3]">OMG! I was fall on evil days, I had gone {{lostAmount}} dollar</view>
			</view>
			<view class="scriptDecision">LOST: {{lostAmount}}</view>
		</uni-popup>
		<view class="gameChat" :style="{height:styleSetting.windowHeight*0.45+'px'}">
			<chat  :userId='userId' :match='matchId' :turn='currentTurn' :round='currentRound'></chat>
		</view>
	</view>
</template>

<script>
	import uniCollapse from '@/components/uni-collapse/uni-collapse.vue'
	import uniCollapseItem from '@/components/uni-collapse-item/uni-collapse-item.vue'
	import UniPopup from '@/components/uni-popup/uni-popup.vue'
	import chat from '@/pages/chat/chat.vue';
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
				showPopup: false,
				showMessage: false,
				showHistory: "true",
				showScript: [false, false, false, false],
				matchId: '', 
				userId: '' ,
				situationId: '',
				currentRound: 'round1',
				currentTurn: 'turn1',
				currentWeather: '',
				currentDecision: '',
				currentSeq: '',
				player1: '',
				player2: '',
				player3: '',
				money: 1000,
				index: 0,
				playerList: ['Player','user','user','user'],
				moneyList: ['Money','0','0','0'],
				emptySeq: ['-', '-', '-', '-', '-', '-', '-', '-', '-', '-'],
				seq1: ['F','T','T','F','F','F','F','T','T','F'],
				seq2: ['F','T','F','T','F','T','F','T','F','F'],
				isActive: false,
				buildCount: '',
				notbuildCount: '',
				decisionList: '',
				lostAmount: 0
			}
		},
		components: {
			firebase,
			UniPopup,
			chat,
			uniCollapse,
			uniCollapseItem
		},
		onLoad(data) {
			_self = this;
			
			this.pixelCalculation();
			this.matchId = data.matchId;
			this.userId = data.userId;
			this.situationId = data.situationId;
			_self.showHistory = data.switch;
			
			var firebaseConfig = {
				apiKey: "<API KEY>",
				authDomain: "<AUTH DOMAIN>",
				databaseURL: "<DB URL>",
				projectId: "<PROJECT ID>",
				storageBucket: "STORAGE BUCKET"
			};
			
			// firebase.initializeApp(firebaseConfig);
			
			//firebase 已經init過在login的時候所以用下面的方式
			if (!firebase.apps.length) {
				firebase.initializeApp(firebaseConfig);
				console.log(firebase.app().name);
			};
			this.checkResult();
			this.getHistoryResult();
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
			sendBuild: function() {
				_self.isActive = true;
				_self.currentDecision = 'build';
				var db = firebase.firestore();
				var decisionConnect = db.collection('PlayAndScore').doc(_self.matchId);//.collection('play_data').doc('play_data'+_self.matchId.split('h')[1]);
				var message = {
					play_choose: {
						[_self.userId]: 'build'
					},
				};
				decisionConnect.set(message, {
					merge: true
				}).then(function() {
					console.log('correctly write')
				});
				uni.showLoading({
					mask: false,
					title: "Waiting For Others' Response"
				});
			},
			sendNotBuild: function() {
				_self.isActive = true;
				_self.currentDecision = 'notbuild';
				var db = firebase.firestore();
				var decisionConnect = db.collection('PlayAndScore').doc(_self.matchId);//.collection('play_data').doc('play_data'+_self.matchId.split('h')[1]);
				var message = {
					play_choose: {
						[_self.userId]: 'notbuild'
					},
				};
				decisionConnect.set(message, {
					merge: true
				}).then(function() {
					console.log('correctly write')
				});
				uni.showLoading({
					mask: false,
					title: "Waiting For Others' Response"
				});
			},
			checkResult: function(){
				var db = firebase.firestore();
				var resultConnect = db.collection('PlayAndScore').doc(_self.matchId);//.collection('play_data').doc('play_data'+String(_self.matchId.slice(5,9)));
				resultConnect.onSnapshot(function(doc){
					_self.currentRound = doc.data().status.processing.slice(0,7);
					_self.currentTurn = doc.data().status.processing.slice(8,14);
					var playerObject = Object.keys(doc.data()[_self.currentRound+'_score']);
					var moneyObject = Object.values(doc.data()[_self.currentRound+'_score']);
					_self.playerList = ['Player'];
					_self.moneyList = ['Money'];
					
					playerObject.forEach(function(item){
						_self.playerList.push(item)
					})
					moneyObject.forEach(function(item){
						_self.moneyList.push(item)
					})
					
					_self.money = doc.data()[_self.currentRound+'_score'][_self.userId];
					_self.decisionList = Object.values(doc.data()['play_choose']);
					_self.buildCount = _self.decisionList.filter(x => x == 'build').length;
					_self.notbuildCount = _self.decisionList.filter(x => x == 'notbuild').length;
					
					if (_self.buildCount+_self.notbuildCount == 3) {
						if (_self.currentTurn == 'turn01') { 
							_self.getCurrentSeqList()
						}
						if ((_self.currentRound=='round02' || _self.currentRound == 'round01' )&& _self.currentTurn == 'turn10') {
							uni.hideLoading();
							_self.checkSeq(); 
							_self.checkStatus();
							_self.showMessage = true;
							setTimeout(function(){
								_self.showMessage = false;
								_self.checkSeq();
								uni.redirectTo({
									url: '../final/final?userId='+_self.userId+'&matchId='+_self.matchId+'&situationId='+_self.situationId+'&path='+'Round2'
								})
							},5000);
						} else {
							uni.hideLoading();
							_self.checkSeq();
							_self.checkStatus();
							_self.showMessage = true;
							_self.index = _self.showScript.findIndex(index => index == true);
							setTimeout(function(){
								_self.isActive = false;
								_self.showMessage = false;
								_self.showScript[_self.index] = false;
								_self.getHistoryResult();
							},10000);
						}
					}
				});
			},
			getCurrentSeqList: function(){
				if (_self.currentRound == 'round01') {
					_self.currentSeq = _self.seq1;
				} else {
					_self.currentSeq = _self.seq2;
				};
			},
			checkStatus: function() {
				_self.currentWeather = _self.currentSeq[Number(_self.currentTurn.slice(4,6))];
				console.log(_self.currentWeather);
				//發生海嘯
				if (_self.currentWeather == 'T') {
					_self.currentWeather = "Tsunami"
					if (_self.currentDecision == 'build') {
						_self.showScript[2] = true
						_self.lostAmount = 40
					} else {
						_self.showScript[3] = true
						if (_self.buildCount == 0) {
							_self.lostAmount = 70 + 40 - 150
						} else if (_self.buildCount == 1) {
							_self.lostAmount = 80 + 40 - 150
						} else if (_self.buildCount == 2) {
							_self.lostAmount = 90 + 40 - 150
						} else {
							_self.lostAmount = 100 + 40 - 150
						}
					}
				//發生水災
				} else {
					_self.currentWeather = "Flood"
					if (_self.currentDecision == 'build') {
						_self.showScript[0] = true
					} else {
						_self.showScript[1] = true
						if (_self.buildCount == 0) {
							_self.lostAmount = 70 + 40 - 150
						} else if (_self.buildCount == 1) {
							_self.lostAmount = 77.5 + 40 - 150
						} else if (_self.buildCount == 2) {
							_self.lostAmount = 85 + 40 - 150
						} else {
							_self.lostAmount = 92.5 + 40 - 150
						}
					}
				};
			},
			checkSeq: function() {
				_self.emptySeq = ['-', '-', '-', '-', '-', '-', '-', '-', '-']
				_self.emptySeq.splice(0,_self.currentTurn.split('n')[1],_self.currentSeq.slice(0,_self.currentTurn.split('n')[1]))
				console.log(_self.emptySeq);
				_self.emptySeq = _self.emptySeq.join(' ')
			},
			getHistoryResult: function(){
				var db = firebase.firestore();
				var historyConnect = db.collection('PlayAndScore').doc(_self.matchId);//collection('play_data').doc('play_data'+String(_self.matchId.slice(5,9)));
				_self.player1 = new Array
				_self.player2 = new Array
				_self.player3 = new Array
				historyConnect.onSnapshot(function(doc){
					_self.getCurrentSeqList();
					if (_self.currentRound == 'round02') {
						_self.player1.push(Object.values(doc.data()['play_history'][_self.playerList[1]]).slice(10,Number(_self.currentTurn.split('n')[1])+9))
						_self.player2.push(Object.values(doc.data()['play_history'][_self.playerList[2]]).slice(10,Number(_self.currentTurn.split('n')[1])+9))
						_self.player3.push(Object.values(doc.data()['play_history'][_self.playerList[3]]).slice(10,Number(_self.currentTurn.split('n')[1])+9))
					} else {
						_self.player1.push(Object.values(doc.data()['play_history'][_self.playerList[1]]))
						_self.player2.push(Object.values(doc.data()['play_history'][_self.playerList[2]]))
						_self.player3.push(Object.values(doc.data()['play_history'][_self.playerList[3]]))
					}
				})
			},
			showPop: function(){
				if (_self.showPopup == false) {
					_self.showPopup = true
				} else {
					_self.showPopup = false
				}
			}
		}
	}
</script>

<style lang="scss">
	@import '@/static/game_with_chat.scss';
</style>
