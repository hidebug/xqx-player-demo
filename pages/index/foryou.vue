<template>
	<view class="content">
		<xqx-player ref="playPanelRef" :panelMode="1" :episodesList="episodesList" :episodesMode="1" :overShadeMode="1"
			:overShadeBack="0" :navBarHeight="navBarHeight" :supplyEpisodes="supplyEpisodes"
			@onPalyletDetail="onPalyletDetail" @onSupplyEpisodes="onSupplyEpisodes" @onEpisodesStart="onEpisodesStart"
			@onEpisodesForbit="onEpisodesForbit" @onPlayerEvent="onPlayerEvent">

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
					<view class="tips">当前列表已经播放完毕</view>
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

		props: {
			fowyouShow: {
				type: Boolean,
				default: false
			},
		},

		data() {
			return {
				title: 'discover',

				//当前播放哪一集的信息，由组件通知获得
				currentEpisodes: {},

				// 推荐的短剧列表
				episodesList: [],

				supplyEpisodes: {
					episodes: null,
					url: null
				},

				navBarHeight: 0
			}
		},

		created() {
			let me = this;

			me.initViewInfo();
			me.playletQueryForyou();
		},

		watch: {
			//页面不显示要暂停播放，延时处理确保时机合适
			fowyouShow: {
				handler(newShow, oldShow) {
					let me = this;

					setTimeout(() => {
						if (!me.$refs.playPanelRef) {
							return;
						}

						if (newShow) {
							me.$refs.playPanelRef.playVideo();
						} else {
							me.$refs.playPanelRef.pauseVideo();
						}
					}, 1000)
				},
				// 强制立即执行回调
				immediate: true
			}
		},

		methods: {
			//根据不同运行环境，计算高度适配
			initViewInfo() {
				let me = this;

				// pages.json 中tabBar 设置了高度
				// #ifdef APP-PLUS
				me.navBarHeight = 0;
				// #endif
				// #ifndef APP-PLUS
				me.navBarHeight = 60;
				// #endif
			},

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
				}, 3000)
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
			//注意：推荐模式时候，playletId 是在episodesItem对象中
			onPalyletDetail(episodes, episodesItem) {
				let me = this;

				console.log("onPalyletDetail: episodes=" + episodes + " ,  episodesItem=" + JSON.stringify(episodesItem));

				uni.navigateTo({
					url: "/pages/index/detail?playetId" + episodesItem.playetId
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
				console.log("toShare");
			},

			//收藏视频
			toFavor() {
				let me = this;

				//me.currentEpisodes 处理当前集收藏逻辑
				console.log("toFavor");
			},

			playletQueryForyou() {
				let me = this;

				me.episodesList = [{
						"playletId": 53,
						"title": "1-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_007.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
					},
					{
						"playletId": 50,
						"title": "2-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_006.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo1.mp4"
					},
					{
						"playletId": 42,
						"title": "3-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_003.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo2.mp4"
					},
					{
						"playletId": 48,
						"title": "4-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_004.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
					},
					{
						"playletId": 47,
						"title": "5-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_005.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo1.mp4"
					},
					{
						"playletId": 46,
						"title": "6-短剧很好看",
						"brief": "隐居山林的战主沐辰在属下的召唤下出山，这个时间变得多么的型芯试试的故事和期待。",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_006.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo2.mp4"
					},
					{
						"playletId": 44,
						"title": "7-短剧很好看",
						"brief": "一次事故失忆后的黎雾，被四个帅哥团团包围，瞬间成为抢手团宠。",
						"covurl": "https://mmpt.oss-ap-southeast-1.aliyuncs.com/img/2024-06-17/7c53a39ae89f4c4bb382eab0639df89e.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
					},
					{
						"playletId": 43,
						"title": "8-短剧很好看",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_007.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo1.mp4"
					},
					{
						"playletId": 41,
						"title": "9-短剧Demo",
						"brief": "短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo短剧Demo",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_001.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo2.mp4"
					},
					{
						"playletId": 40,
						"title": "10-短剧Demo",
						"brief": "无疑进入修仙儿，却发现发生了许许多多的变故，他决心改变这个现状。",
						"covurl": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/small_002.jpg",
						"canPlay": 1,
						"url": "https://env-00jxhb2rkow0.normal.cloudstatic.cn/xqx-playlet/demo3.mp4"
					}
				]
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
		padding: 0rpx;
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
	}
</style>