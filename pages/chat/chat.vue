<template>
	<view class="wholePage chat">
		<view class="topChat" :style="{height:styleSV.contentViewHeight+'px'}">
			<scroll-view id="scrollview" class="contentShowStyle" :scroll-top="scrollTop" :scroll-with-animation="false" scroll-y="true" @scroll="scroll">
				<view class="contentSaveStyle" v-for="(item,index) in contentSave" :key='index'>
					<view class="textMe" v-if="item['userId'] == userId">
						<view class="timeMe">{{item['realTime']}}</view><view class="borderMe">{{item['text']}}</view>{{item['userId']}}
					</view>
					<view class="textOther" v-if="item['userId'] != userId">
						{{item['userId']}}<view class="borderOther">{{item['text']}}</view><view class="timeOther">{{item['realTime']}}</view>
					</view>
				</view>
			</scroll-view>
		</view>
		<view class="bottomChat" :style="{height:styleBC.showHeight+'px'}">
			<textarea class="contentDec" :style="{width:styleBC.showWidth+'px'}" v-model="contentShow" maxlength="40"/>
			<view size="mini" :style="{width:styleBC.showWidthButton+'px'}" class="buttonSend" @click="submitDb">Send</view>
		</view>		
	</view>
</template>

<script>
	import * as firebase from '@/common/app.js';
	import '@/common/firestore.min.js';
	import '@/common/function.min';
	

	var _self;

	export default {
		name:'chat',
		props:{
			config: {
				type: Object,
				default() {
					return {
						var firebaseConfig = {
							apiKey: "<API KEY>",
							authDomain: "<AUTH DOMAIN>",
							databaseURL: "<DB URL>",
							projectId: "<PROJECT ID>",
							storageBucket: "STORAGE BUCKET"
						};
					};
				}
			},
			userId: {
				type: String,
				default: 'John'
			},
			match: {
				type: String,
				default: 'match2000'
			},
			turn: {
				type: String,
				default: 'turn01'
			},
			round: {
				type: String,
				default: 'round01'
			}
		},
		data() {
			return {
				styleSV : {
					contentViewHeight: 0,
					mitemHeight: 0
				},
				styleBC : {
					showHeight: 0,
					showWidth: 0,
					showWidthButton: 0
				},
				// 用於顯示
				contentSave : [],
				// 用於push並儲存聊天記錄
				contentShow : '',
				scrollTop : 0,
				chatLength : 0,
				timeSave : 0,
				old: {
					scrollTop : 0
				}
			}
		},
		components:{
			firebase
		},
		created(data) {
			_self = this;
			this.pixelCalculation(); 
			
			// 連接Firestore並啟動
			// firebase.initializeApp(this.config);

			if (!firebase.apps.length) {
				firebase.initializeApp(this.config);
				console.log(firebase.app().name);
			};
			// 將相關對應項目寫入變數，方便帶入
			// 將上頁登入賬號名稱代入本頁
			// this.userId = data.userId;	
			// console.log(_self.scrollTop);
			// this.snapOnceMessage();
			this.snapNewMessage();
		},
		methods: {
			snapNewMessage: function(){
				var db = firebase.firestore();
				var newMessageConnect = db.collection('Chatroom').doc('Game0001_'+_self.match).collection('message');
				newMessageConnect.orderBy('time','desc').where("time",'>',_self.timeSave).limit(1).onSnapshot(function(docs){
					docs.docs.forEach(function(doc){
						console.log(doc.data())
						_self.contentSave.push(doc.data());
						_self.timeSave = doc.data().time;
					});
					console.log(_self.timeSave)
				})
			},
			snapOnceMessage: function(){
				var getHistoryChatConnect = firebase.firestore().collection('Chatroom/Game0001_'+_self.match+'/message');
				getHistoryChatConnect.once('value', function(snapshot) {
					console.log(snapshot);
				});
			},
			// 登入本頁時，按照手機長寬進行有效佈局
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleBC.showHeight = phoneData.windowHeight*0.07;
				this.styleBC.showWidth = phoneData.windowWidth - (phoneData.windowWidth * 0.26)
				this.styleBC.showWidthButton = phoneData.windowWidth - (phoneData.windowWidth * 0.8);
				this.styleSV.contentViewHeight = phoneData.windowHeight*0.93;
				var scrollViewHeight = phoneData.windowHeight;
				// console.log(phoneData);
				uni.onWindowResize((res) => {
					if (res.deviceOrientation == 'portrait') {
						this.pixelCalculation();
					} else {
						this.styleBC.showHeight = res.size.windowHeight*0.13;
						this.styleBC.showWidth = res.size.windowWidth - (res.size.windowWidth * 0.24)
						this.styleBC.showWidthButton = res.size.windowWidth - (res.size.windowWidth * 0.8);
						this.styleSV.contentViewHeight = res.size.windowHeight*0.87;
					};
				});
			},
			scroll: function(e){
				_self.old.scrollTop = e.detail.scrollTop;
				console.log(_self.old.scrollTop);
				// console.log(_self.scrollTop);
			},
			// 每新增一筆資訊，將自動滑動到最新消息
			scrollToBottom: function () {
				var query = uni.createSelectorQuery();
				// selectAll 為選取所有row
				query.selectAll('.contentSaveStyle').boundingClientRect();
				query.select('#scrollview').boundingClientRect();	
				// 執行時針對現屏幕進行半段做出置底
				query.exec(function (res) {
					_self.styleSV.mitemHeight = 0;
					_self.styleSV.mitemHeight = res[1].height;	
					// console.log(res);
					res[0].forEach(function (rect) {
						_self.styleSV.mitemHeight = _self.styleSV.mitemHeight + rect.height;
					});
					if (_self.styleSV.mitemHeight > (_self.styleSV.contentViewHeight - 100)) {
						_self.$nextTick(function(){
							// _self.scrollTop = _self.styleSV.mitemHeight + 44
							// _self.old.scrollTop = _self.scrollTop;
							_self.scrollTop = _self.styleSV.mitemHeight + 44
						})
						
						// _self.scrollTop = _self.styleSV.mitemHeight + 44;
						// console.log(_self.styleSV.mitemHeight + 44);
					}
				});
			},
			// Local端顯示及儲存，再推送到資料庫
			submitDb : function(){
				// 先把格式（Dic）設定好
				var checkSpace = this.contentShow.trim();
				if (checkSpace == '' || checkSpace == null) {
					uni.showModal({
						title: 'NOTICE',
						content: 'Message cannot be empty !',
						confirmText: 'OK',
						showCancel: false
					})
				}else{
					var message = {
						text : this.contentShow,
						userId : this.userId,
						time : new Date().getTime(),
						// realDate : new Date().toDateString()
						realTime : new Date().toLocaleTimeString(),
						round : this.round,
						turn : this.turn
					};
					// 發送信息時先上傳firestore
					this.uploadDB(message);
				};
				//完成一切作業後將textarea清空
				_self.contentShow = '';
			},
			uploadDB : function(message){
				var db = firebase.firestore();
				var chatConnect = db.collection('Chatroom').doc('Game0001_'+_self.match).collection('message');
				chatConnect.add(message).then(function(docRef) {
					// 上傳firestore後console相關id
					console.log("Document written with ID: ", docRef.id);
					// _self.snapNewMessage();
					_self.scrollToBottom();
				})
				.catch(function(error) {
					console.error("Error adding document: ", error);
				});
			}
		}
	}
</script>

<style lang="scss">
	@import "@/static/chat.scss" ;
</style>
