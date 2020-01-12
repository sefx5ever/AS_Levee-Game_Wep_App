<template>
	<view class="story">
		<view class="storySection">
			<view class="storyTitle">Abstract</view>
			<view class="storyText">
				Peace before the storm, you may not know what's happen unpredictable !'
				It can be flooding or tsunami, but the only way to avoid is co-operation,
				it can minimize the lost.
			</view>
			<view class="ruleTextBasic">
				<view class="ruleTextBasicHint" :style="{width:styleSetting.windowWidth*0.9+'px'}">HINT</view>
				<view class="ruleTextBasicText" :style="{width:styleSetting.windowWidth*0.85+'px'}">The game will happen in two situation, which are FLOODING and TSUNAMI</view>	
			</view>
			<view>
				<button class="storyButton" :style="{width:styleSetting.windowWidth*0.9+'px'}" @click="nextRule()">Next: Rule Declaration</button>
			</view>
		</view>
		<img class="storyImage" :style="{height:styleSetting.windowHeight+'px'}" src="~@/static/picture/Background_story.jpg" mode="aspectFit">
	</view>
</template>

<script>
	export default {
		data () {
			return {
				styleSetting: {
					windowHeight: '',
					windowWidth: ''
				},
				userId: '',
				matchId: '',
				situationId: ''
			}
		},
		onLoad(data) {
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
		methods: {
			pixelCalculation: function() {
				const phoneData = uni.getSystemInfoSync();
				this.styleSetting.windowHeight = phoneData.windowHeight;
				this.styleSetting.windowWidth = phoneData.windowWidth;
				uni.onWindowResize((res) => {
					this.pixelCalculation();
				});
			},
			nextRule: function(){
				uni.redirectTo({
					url: '../rule/rule?userId='+this.userId+'&matchId='+this.matchId+'&situationId='+this.situationId
				})
			}
		}
	}
</script>

<style lang="scss">
	@import '@/static/story.scss';
</style>
