<template>
	<view>
		<view class="topPage" >
			<scroll-view class="scrollview" :scroll-x="true" :show-scrollbar="false" :scroll-into-view="scroll_into_view">
				<view class="scrollviewOption" :style="{height:styleSetting.topHeight+'px'}">
					<view id="game" class="scrollviewSelect" :class="[isActive == 'game' ? 'colorValueTrue' : '']" data-option="game" @click="switchtab">Game</view>
					<view id="grouping" class="scrollviewSelect"  :class="[isActive == 'grouping' ? 'colorValueTrue' : '']" data-option="grouping" @click="switchtab">Grouping</view>
					<view id="user" class="scrollviewSelect" :class="[isActive == 'user' ? 'colorValueTrue' : '']" data-option="match" @click="switchtab">User</view>
					<view id="match" class="scrollviewSelect" :class="[isActive == 'match' ? 'colorValueTrue' : '']" data-option="user" @click="switchtab">Match</view>
					<view class="scrollviewSelect" @click="logout()">Log Out</view>
				</view>
			</scroll-view>
		</view>
		<view>
			<swiper :style="{height:styleSetting.bottomHeight+'px'}" :indicator-dots="false" :autoplay="false" :current="tabIndex">
				<swiper-item id="game">
					<scroll-view scroll-x="true">
						<view class="centerGame">
							<view class="titleGame">Game Setting:</view>
							<view class="topGame">				
								<view class="detailGame">Tsunami Cost:<input class="inputGame" maxlength="5" type="text" placeholder="100" placeholder-style="text-align:center" v-model="tsunamiCost"></view>
								<view class="detailGame">Flooding Cost:<input class="inputGame" maxlength="5" type="text" placeholder="10" placeholder-style="text-align:center" v-model="floodingCost"></view>
								<view class="detailGame">Time Duration:<input class="inputGame" maxlength="5" type="text" placeholder="30" placeholder-style="text-align:center" v-model="timeDuration"></view>
								<view class="detailGame">Player Per Group:</view><slider class="matchRihtSlider" show-value="true" @change="playerNumberChange" min="3" max="5" current="4" indicator-dots="true" display-multiple-items="5" :style="{width:styleSwiper.witdh+'px'}"/>
							</view>
							<view class="bottomGame">
								<button size="mini" type="primary" @click="saveMatch()">Save</button>
								<button size="mini" type="primary" @click="resetMatch()">Reset</button>
							</view>
						</view>
					</scroll-view>
				</swiper-item>
				<swiper-item id="grouping"  class="swiperClassification" :style="{height:styleSetting.bottomHeight+'px'}">
					<scroll-view scroll-x="true">
						<view class="topPageClassification">
<!-- 							<view class="dateInput">
								<view class="date">From Date:</view>
								<input class="input" type="text" placeholder="Year" maxlength="4" placeholder-style="text-align:center">
								<input class="input" type="text" placeholder="Month" maxlength="2" placeholder-style="text-align:center">
								<input class="input" type="text" placeholder="Day" maxlength="2" placeholder-style="text-align:center">
							</view>
							<view class="dateInput">
								<view class="date">To Date:</view>
								<input class="input" type="text" placeholder="Year" maxlength="4" placeholder-style="text-align:center">
								<input class="input" type="text" placeholder="Month" maxlength="2" placeholder-style="text-align:center">
								<input class="input" type="text" placeholder="Day" maxlength="2" placeholder-style="text-align:center">
							</view> -->
							<view class="dateInput">
								<view class="date">Url:</view>
								<input class="input" type="text" placeholder="Upload For Classification" placeholder-style="text-align:center" v-model="urlConnect">
							</view>
							<view class="buttonInput">
								<view>
									<button size="mini" type="primary" @click="uploadNlist()">Upload</button>
								</view>
								<view v-if="isReady == false">
									<button size="mini" type="primary" disabled="true" @click="editNlist()">Play</button>
								</view>
								<view v-if="isReady == true">
									<button size="mini" type="primary" @click="editNlist()">Play</button>
								</view>
								
								<!-- <button size="mini" type="primary">Find</button> -->
							</view>
						</view>
						<scroll-view class="bottomScrollview" scroll-y="true">
							<view class="bottomPageClassification">
<!-- 								<view class="bottomScroll" v-for="(item,index) in 6" :key="index">
									<uni-card class="unicardStyle" title="Leevee Game" thumbnail="../../static/Leevee.png" @click='clickCard' note="Group: 4">
										<view v-for="(item,index) in 4" :key="index">
											<view>Name</view>
										</view>
									</uni-card>
								</view> -->
							</view>
						</scroll-view>
					</scroll-view>
				</swiper-item>
				<swiper-item id="user">
					<scroll-view scroll-x :style="{height:styleSetting.bottomHeight+'px'}">
						<view class="userTitle">Player Setting</view>
						<view class="topPageUser">
							<view class="detailId">Search For ID:<input class="idInput" type="text" placeholder="UserID" placeholder-style="text-align:center" v-model="userNumber"></view>
							<button size="mini" type="primary" @click="userSearch()">Search</button>
						</view>
						<view class="userTitle">User Detail:</view>
						<view class="bottomPageUser">
							<view class="userDetail">
								<view class="userLeft">Player:<view class="userRight">{{playerName}}</view></view>
 								<view class="userLeft">Age:<view class="userRight">{{playerAge}}</view></view>
								<view class="userLeft">Race:<view class="userRight">{{playerRace}}</view></view>
								<view class="userLeft">Game Status:<view class="userRight">{{playerGameStatus}}</view></view>
							</view>
							<view class="userDetailButton">
								<button size="mini" type="primary" @click="userSetting()">User Setting</button>
							</view>
							<uni-popup :show="isChangeUser === true" position="middle" mode="fixed" msg=userNumber>
								<view class="unipopup">
									<view class="popupLeft">Player:</view><input class="popupRight" v-model="playerName">
									<view class="popupLeft">Age:</view><input class="popupRight" v-model="playerAge">
									<view class="popupLeft">Race:</view><input class="popupRight"  v-model="playerRace">
									<view class="popupLeft">Game Status:</view><input class="popupRight" v-model="playerGameStatus">
									<button style="margin-top: 5%;" type="primary" @click="userDataSubmit()">Submit</button>
								</view>
							</uni-popup>
						</view>
					</scroll-view>
				</swiper-item>
				<swiper-item id="matching" :style="{height:styleSetting.bottomHeight+'px'}">
					<scroll-view scroll-x>
						<view class="matchTitle">Join Match</view>
						<view class="topPageMatch">
							<view class="matchId">Match ID:<input class="idInput" type="text" placeholder="MatchID" placeholder-style="text-align:center" v-model="matchId"></view>
							<view class="matchButton">
								<button size="mini" type="primary" @click="joinGame()">Join</button>
								<button size="mini" type="primary" @click="stopGame()">Stop</button>
							</view>
						</view>
						<view class="bottomPageMatch">
							<view class="matchTitle">Match Setting</view>
							<view class="matchDetail">
								<view class="detailMatch">Tsunami Cost:<input class="inputMatch" maxlength="5" type="text" placeholder="100" v-model="tsunamiCost"></view>
								<view class="detailMatch">Flooding Cost:<input class="inputMatch" maxlength="5" type="text" placeholder="10" v-model="floodingCost"></view>
								<view class="detailMatch">Time Duration:<input class="inputMatch" maxlength="5" type="text" placeholder="30" v-model="timeDuration"></view>
								<view class="detailMatch">Player Per Group:</view><slider class="matchRihtSlider" show-value="true" @change="playerNumberChange" min="3" max="5" current="4" indicator-dots="true" display-multiple-items="5" :style="{width:styleSwiper.witdh+'px'}"/>
							</view>
							<view class="matchDetailButton">
								<button size="mini" type="primary" @click="saveMatch()">Save</button>
								<button size="mini" type="primary" @click="resetMatch()">Reset</button>
							</view>
						</view>
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
	</view>
</template>

<script>
	import uniCard from '@/components/uni-card/uni-card.vue'
	import UniPopup from '@/components/uni-popup/uni-popup.vue'
	import * as firebase from '@/common/app.js';
	import '@/common/firestore.min.js';
	import '@/common/auth.min.js';

	var _self;

	export default {
		data() {
			return {
				cacheTab : [],
				urlConnect : "https://docs.google.com/spreadsheets/d/1sefxKcrFGhfJztdeBOwx9l58HR1kAruzEO3JN5zMKKk/edit#gid=0",
				tabIndex : "",
				idList : [],
				matchId : "",
				scroll_into_view : "",
				tsunamiCost : 0,
				floodingCost : 0,
				timeDuration : 0,
				playerNumber : 0,
				userNumber : "",
				number : "",
				currentColorId : "game",
				playerName : "",
				playerAge : "",
				playerRace : "",
				playerGameStatus : "",
				playerSituation : "",
				isActive : "game",
				isReady : false,
				isChangeUser : false,
				styleSetting : {
					topHeight : 0,
					bottomHeight : 0
				},
				styleSwiper : {
					width : 0
				}
			}
		},
		components: {
			uniCard,
			UniPopup
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
			
			// 將相關對應項目寫入變數，方便帶入
			var db = firebase.firestore();
			var statusConnect = db.collection('Features').doc('BackAndFront');
			statusConnect.onSnapshot(async function(doc){
				_self.isReady = doc.data().RFI_status === "take";
				if(_self.isReady === true){
					uni.hideLoading();
				}
				if(doc.data().CM_status === "confirm"){
					uni.hideLoading();
					_self.playNlist();
				}
				// console.log(_self.isReady);
				
			})
		},
		// 完成加載後監聽，方便switch tab
		onReady() {
			this.getIdPosition();
		},
		methods: {
			// 監聽手機比例
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleSetting.bottomHeight = phoneData.windowHeight*0.92;
				this.styleSetting.topHeight = phoneData.windowHeight*0.08;
				this.styleSwiper.width = phoneData.windowWidth*0.2;
				// console.log(phoneData);
				uni.onWindowResize((res) => {
					this.pixelCalculation()
				});
			},
			// 登出賬號
			logout: function() {
				uni.showModal({
					title: "Notice",
					content: "Conform Sign Out?",
					confirmText: "YES",
					cancelText: "No",
					success(res) {
						if (res.confirm) {
							uni.reLaunch({
								url: "../login/login"
							})
						}
					}
				})
			},
			// 用於雙向進行頁面轉換
			switchtab: function(id) {
				this.tabIndex = _self.idList.indexOf(id.target.id);
				this.scroll_into_view = id.target.id;
				this.isActive = id.target.id;
			},
			// 加載完後，讀取選項名稱
			getIdPosition: function() {
				var query = uni.createSelectorQuery();
				query.selectAll(".scrollviewSelect").boundingClientRect();
				query.exec(function(idName) {
					for (let x = 0; x < idName[0].length; x++) {
						_self.idList.push(idName[0][x].id);
					};
					// console.log(_self.idList);
				})
			},
			uploadNlist: function(){
				var db = firebase.firestore();
				var excelConnect = db.collection("Features").doc("BackAndFront");
				excelConnect.update({
					RFI_status : "upload",
				});
				this.urlConnect = "";
				uni.showLoading({
					title: 'Uploading...'
				});
			},
			editNlist: function(){
				var db = firebase.firestore();
				var excelConnect = db.collection("Features").doc("BackAndFront");
				excelConnect.update({
					CM_status : "initialize",
				});
				uni.showLoading({
					title: 'Initialzing...'
				});
			},
			playNlist: function(){
				var db = firebase.firestore();
				var excelConnect = db.collection("Features").doc("BackAndFront");
				uni.showModal({
					title: "Notice",
					content: "Initialize successfully. Do you need to edit default setting?",
					confirmText: "YES",
					cancelText: "NO",
					success: (res) => {
						console.log(res);
						if (res.confirm) {
							this.tabIndex = "3";
							this.scroll_into_view = "match";
						} else if (res.cancel) {
							excelConnect.update({
								CM_status : "create"
							})
							.then(function() {
								console.log("Document successfully updated!");
							})
							.catch(function(error) {
								// The document probably doesn't exist.
								console.error("Error updating document: ", error);
							});
						} else {
							
						}
					}
				});
			},
			// 搜尋相關玩家
			userSearch: function(e) {
				var db = firebase.firestore();
				var userConnect = db.collection("User_test").doc(_self.userNumber);
				var matchingConnect = userConnect.collection('play_data').doc('now_matching');				
				userConnect.get().then(function(doc) {
					if (doc.exists) {
						console.log(doc.data());
						_self.playerName = doc.data().account;
						_self.playerAge = doc.data().age;
						_self.playerRace = doc.data().race;
					} else {
						// doc.data() will be undefined in this case
						console.log("No such document!");
					}
				}).catch(function(error) {
					console.log("Error getting document:", error);
				});
				matchingConnect.get().then(function(doc){
					if (doc.exists) {
						_self.playerGameStatus = doc.data()["play_data"]["now_matching"];
					}
				})
			},
			userSetting: function(e){
				this.isChangeUser = true;
				console.log(this.isChangeUser);
			},
			userDataSubmit: function(e){
				var db = firebase.firestore();
				var userConnect = db.collection("User_test").doc(this.userNumber);
				userConnect.update({
					account : _self.playerName,
					age : _self.playerAge,
					race : _self.playerRace
				})
				.then(function() {
					console.log("Document successfully updated!");
				})
				.catch(function(error) {
					// The document probably doesn't exist.
					console.error("Error updating document: ", error);
				});
				_self.userNumber = ""
				_self.isChangeUser = false;
			},
			playerChange: function(playerNumber) {
				console.log(playerNumber);
			},
			// 加入match
			joinGame: function(e) {
				uni.navigateTo({
					url: "../viewvergame/viewvergame?matchId="+this.matchId,
					animationType: 'pop-in',
					animationDuration: 200
				})
			},
			// 停止相關match
			stopGame: function(e) {
				var db = firebase.firestore();
				var gameConnect = db.collection("PlayAndScore").doc(this.matchId);
				gameConnect.update({
					status : "break"
				})
				.then(function() {
					console.log("Document successfully updated!");
				})
				.catch(function(error) {
					// The document probably doesn't exist.
					console.error("Error updating document: ", error);
				});
			},
			// 設定match條件
			saveMatch: function(e) { // bug
				if (this.tsunamiCost == "" & this.floodingCost == "" & this.match == "") {
					uni.showModal({
						title : "Notice",
						content : "Value cannot be empty !",
						confirmText : "OK",
						showCancel: false
					})
				} else {
					var db = firebase.firestore();
					var matchConnect = db.collection("Games").doc("Game0001");
					matchConnect.set({
						FH_value : this.tsunamiCost,
						FL_value : this.floodingCost,
						PlayerAmount : this.playerNumber,
						matchtime : this.timeDuration*100,
						game : "Game0001"
					},{merge:true})
					.then(function() {
						console.log("Document successfully updated!");
					})
					.catch(function(error) {
						console.error("Error updating document: ", error);
					});
				}
			},
			// 重設match條件
			resetMatch: function(e) {
				this.tsunamiCost = 100;
				this.floodingCost = 10;
				this.playerNumber = 4;
				this.timeDuration = 5;
			},
			playerNumberChange: function(e) {
				this.playerNumber = e.detail.value;
			}
		}
	}
</script>

<style lang="scss">
	@import "@/static/backenduser.scss";
</style>
