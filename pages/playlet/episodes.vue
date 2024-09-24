<template>
	<view class="content">
		<xqx-player ref="playPanelRef" :panelMode="2" :playletDetail="playletDetail" :episodesList="episodesList"
			:episodesStart="episodesStart" :episodesMode="2" :playPanelBack="1" :overShadeMode="2"
			:supplyEpisodes="supplyEpisodes" @onPalyletDetail="onPalyletDetail" @onSupplyEpisodes="onSupplyEpisodes"
			@onEpisodesStart="onEpisodesStart" @onEpisodesForbit="onEpisodesForbit" @onPlayerEvent="onPlayerEvent">

			<template v-slot:funcs>
				<view class="funcitem" @click="toFavor()">
					<image class="img" src="../../static/img/share.png"> </image>
				</view>
				<view class="funcitem" @click="toShare()">
					<image v-if="currentEpisodes.favor == 1" class="img" src="../../static/img/favor.png">
					</image>
					<image v-else class="img" src="../../static/img/favor_no.png"></image>
				</view>
			</template>

			<template v-slot:overshade>
				<view class="shade">
					<view class="tips">短剧所有的集数已经播放完毕</view>
					<view class="gotips"> 1. 点击按钮观看更多短剧</view>
					<view class="button" @click="toBack">去观看</view>
					<view class="gotips mt30">2. 点击去赚取更多的积分</view>
					<view class="button" @click="toEarnScore">去赚取</view>
				</view>
			</template>
		</xqx-player>
	</view>
</template>

<script>
	import XqxPlayer from 'uni_modules/xqx-player/components/xqx-player/xqx-player.vue';

	export default {
		components: {
			XqxPlayer
		},

		data() {
			return {
				title: 'discover',

				supplyEpisodes: {
					episodes: null,
					url: null
				},

				//当前播放哪一集的信息，由组件通知获得
				currentEpisodes: {},

				//短剧详情
				playletDetail: {},

				// 短剧剧集列表
				episodesList: [],

				episodesStart: 0,
			}
		},
		onLoad(option) {
			let me = this;

			if (option.episodesStart) {
				me.episodesStart = Number(option.episodesStart);
			}

			me.playletQueryDetail();
		},

		onShow() {
			let me = this;

			//页面初始化时，第一次调用me.$refs.playPanelRef 可能为空
			if (me.$refs.playPanelRef) {
				me.$refs.playPanelRef.playVideo();
			}
		},

		onHide() {
			let me = this;
			me.$refs.playPanelRef.pauseVideo();
		},

		methods: {
			//只是通知当前播放哪一集，即使当前集的视频需要先购买才能播放也会通知
			onEpisodesStart(episodes, episodesItem) {
				let me = this;
				console.log("onEpisodesStart: episodes=" + episodes + " ,episodesItem=" + JSON.stringify(episodesItem));

				me.currentEpisodes = episodesItem;

				//触发加载额外的短剧播放列表
				if (episodes > me.episodesList.length - 10) {
					//me.episodesList = me.episodesList.concat(extraEpisodesList)
					console.log("onEpisodesStart: 触发加载额外的短剧播放列表");
				}
			},

			//如果当前集不允许播放，回调让使用方决定怎么做，比如弹窗购买提示视图
			onEpisodesForbit(episodes, episodesItem) {
				let me = this;
				console.log("onEpisodesForbit: episodes=" + episodes + " , episodesItem=" + JSON.stringify(episodesItem));

				//实现参考，实际可能弹窗购买或者做任务提示框，用户完成要求后设置canPay为1同时触发刷新播放单集视频
				setTimeout(() => {
					episodesItem.canPlay = 1;
					//只要变化就会触发刷新，使用方自由控制
					me.$refs.playPanelRef.flushVideo();
				}, 5000)
			},


			//需要提供哪一集的视频地址
			onSupplyEpisodes(episodes, episodesItem) {
				let me = this;

				console.log("onSupplyEpisodes: episodes=" + episodes + " ,  episodesItem=" + JSON.stringify(episodesItem));

				me.supplyEpisodes = {
					playletId: episodesItem.playetId,
					episodes: episodes,
					url: episodesItem.url
				}
			},

			//点击短剧详情展示
			//注意：剧集模式时候，playletId 是在playletDetail对象中
			onPalyletDetail(episodes, episodesItem) {
				let me = this;

				console.log("onPalyletDetail: episodes=" + episodes + " ,  episodesItem=" + JSON.stringify(episodesItem));

				uni.navigateTo({
					url: "/pages/index/detail?playetId" + me.playletDetail.playetId
				})
			},

			//实例播放器事件
			onPlayerEvent(eventData) {
				let me = this;

				console.log("onPlayerEvent: eventData" + JSON.stringify(eventData));
			},

			//分享视频
			toShare() {
				let me = this;

				//me.currentEpisodes 处理当前集分享逻辑
			},

			//收藏视频
			toFavor() {
				let me = this;

				//me.currentEpisodes 处理当前集收藏逻辑
			},

			//返回上一个页面
			toBack() {
				let me = this;
				uni.navigateBack();
			},

			//进入赚积分页面
			toEarnScore() {
				let me = this;
				uni.navigateBack();
			},

			playletQueryDetail() {
				let me = this;

				//短剧信息
				me.playletDetail = {
					"playletId": 53,
					"title": "Short play demo",
					"favor": 1, //收藏表示，0：未收藏，1：已收藏					
					"brief": "在不久的未来，人类文明进入了量子科技时代，量子计算和人工智能彻底改变了社会结构。科学家们发现了一个能够操纵",
					"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_006.jpg",
					"episodes": 7, //已经上传多少集
					"totalEpisodes": 100 //总集数
				}

				//短剧关联的剧集列表
				me.episodesList = [{
					"canPlay": 1,
					"episodes": 1,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
				}, {
					"canPlay": 1,
					"episodes": 2,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo2.mp4"
				}, {
					"canPlay": 1,
					"episodes": 3,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo1.mp4"
				}, {
					"canPlay": 1,
					"episodes": 4,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
				}, {
					"canPlay": 1,
					"episodes": 5,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo2.mp4"
				}, {
					"canPlay": 1,
					"episodes": 6,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
				}, {
					"canPlay": 1,
					"episodes": 7,
					"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo1.mp4"
				}];
			}
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}


	.funcitem {
		display: flex;
		flex-direction: column;
		margin-top: 40rpx;
		justify-content: center;
		align-items: center;

		.img {
			width: 65rpx;
			height: 65rpx;
		}
	}

	.shade {
		.tips {
			padding: 20rpx;
			color: #FFFFFF;
			font-size: 38rpx;
		}

		.gotips {
			padding: 20rpx 0rpx 0rpx 50rpx;
			color: #FFFFFF;
			font-size: 32rpx;
			text-align: left;
			width: 100%;
		}

		.button {
			width: 30%;
			height: 60rpx;
			line-height: 60rpx;
			font-size: 30rpx;
			padding-left: 10rpx;
			padding-right: 10rpx;
			border-radius: 20rpx;
			margin-top: 20rpx;
			margin-left: 100rpx;
			text-align: center;
			background-color: #f4ce15;
			color: #000000;
		}
	}
</style>