<template>
	<view>
		<view class='my-promotion'>
			<view class="header">
				<view class='name acea-row row-center-wrapper'>
					<view>当前佣金</view>
					<navigator url='/pages/users/user_spread_money/index?type=1' hover-class="none" class='record'>提现记录<text class='iconfont icon-xiangyou'></text></navigator>
				</view>
				<view class='num'>{{spreadInfo.commissionCount}}</view>
				<view class='profit acea-row row-between-wrapper'>
					<view class='item'>
						<view>昨日收益</view>
						<view class='money'>{{spreadInfo.lastDayCount ? Number(spreadInfo.lastDayCount).toFixed(2) : 0}}</view>
					</view>
					<view class='item'>
						<view>累积已提</view>
						<view class='money'>{{spreadInfo.extractCount ? Number(spreadInfo.extractCount).toFixed(2) : 0}}</view>
					</view>
				</view>
			</view>
			<!-- #ifdef APP-PLUS || H5 -->
			<navigator url="/pages/users/user_cash/index" hover-class="none" class='bnt bg-color'>立即提现</navigator>
			<!-- #endif -->
			<!-- #ifdef MP -->
			<view @click="openSubscribe('/pages/users/user_cash/index')" class='bnt bg-color'>立即提现</view>
			<!-- #endif -->
			<view class='list acea-row row-between-wrapper'>
				<navigator url='/pages/users/user_spread_code/index' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-erweima'></text>
					<view>推广名片</view>
				</navigator>
				<navigator url='/pages/users/promoter-list/index' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-tongji'></text>
					<view>推广人统计</view>
				</navigator>
				<navigator url='/pages/users/user_spread_money/index?type=2' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-qiandai'></text>
					<view>佣金明细</view>
				</navigator>
				<navigator url='/pages/users/promoter-order/index' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-dingdan'></text>
					<view>推广人订单</view>
				</navigator>
				<navigator url='/pages/users/promoter_rank/index' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-paihang1'></text>
					<view>推广人排行</view>
				</navigator>
				<navigator url='/pages/users/commission_rank/index' hover-class="none" class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-paihang'></text>
					<view>佣金排行</view>
				</navigator>
			</view>
		</view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
		<home></home>
	</view>
</template>

<script>
	import { getSpreadInfo } from '@/api/user.js';
	import { openExtrctSubscribe } from '@/utils/SubscribeMessage.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import home from '@/components/home';
	export default {
		components: {
			// #ifdef MP
			authorize,
			// #endif
			home
		},
		data() {
			return {
				spreadInfo: {},
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getUserInfo();
					}
				},
				deep: true
			}
		},
		onLoad() {
			if (this.isLogin) {
				this.getSpreadInfo();
			} else {
				toLogin();
			}
		},
		methods: {
			onLoadFun: function() {
				this.getSpreadInfo();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			openSubscribe: function(page) {
				uni.navigateTo({
					url: page,
				});
				// uni.showLoading({
				// 	title: '正在加载',
				// })
				// openExtrctSubscribe().then(res => {
				// 	uni.hideLoading();
				// 	uni.navigateTo({
				// 		url: page,
				// 	});
				// }).catch(() => {
				// 	uni.hideLoading();
				// });
			},
			/**
			 * 获取个人用户信息
			 */
			getSpreadInfo: function() {
				let that = this;
				getSpreadInfo().then(res => {
					that.$set(that,'spreadInfo',res.data);
				});
			}
		}
	}
</script>

<style scoped lang="scss">
	.my-promotion .header {
		background-image: url("data:image/png;base64,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");
		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 100%;
		height: 375rpx;
	}

	.my-promotion .header .name {
		font-size: 30rpx;
		color: #fff;
		padding-top: 57rpx;
		position: relative;
	}

	.my-promotion .header .name .record {
		font-size: 26rpx;
		color: rgba(255, 255, 255, 0.8);
		position: absolute;
		right: 20rpx;
	}

	.my-promotion .header .name .record .iconfont {
		font-size: 25rpx;
		margin-left: 10rpx;
		vertical-align: 2rpx;
	}

	.my-promotion .header .num {
		text-align: center;
		color: #fff;
		margin-top: 28rpx;
		font-size: 90rpx;
		font-family: 'Guildford Pro';
	}

	.my-promotion .header .profit {
		padding: 0 20rpx;
		margin-top: 35rpx;
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.8);
	}

	.my-promotion .header .profit .item {
		min-width: 200rpx;
		text-align: center;
	}

	.my-promotion .header .profit .item .money {
		font-size: 34rpx;
		color: #fff;
		margin-top: 5rpx;
	}

	.my-promotion .bnt {
		font-size: 28rpx;
		color: #fff;
		width: 258rpx;
		height: 68rpx;
		border-radius: 50rpx;
		text-align: center;
		line-height: 68rpx;
		margin: -32rpx auto 0 auto;
	}

	.my-promotion .list {
		padding: 0 20rpx 50rpx 20rpx;
		margin-top: 10rpx;
	}

	.my-promotion .list .item {
		width: 345rpx;
		height: 240rpx;
		border-radius: 20rpx;
		background-color: #fff;
		margin-top: 20rpx;
		font-size: 30rpx;
		color: #666;
	}

	.my-promotion .list .item .iconfont {
		font-size: 70rpx;
		background-image: linear-gradient(to right, #fc4d3d 0%, #e93323 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		margin-bottom: 20rpx;
	}
</style>
