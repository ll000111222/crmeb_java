<template>
	<view>
		<form @submit="formSubmit" report-submit='true'>
			<view class='evaluate-con'>
				<view class='goodsStyle acea-row row-between'>
					<view class='pictrue'>
						<image :src='productInfo.image'></image>
					</view>
					<view class='text acea-row row-between'>
						<view class='name line2'>{{productInfo.storeName}}</view>
						<view class='money'>
							<view>￥{{productInfo.price}}</view>
							<view class='num'>x{{cart_num}}</view>
						</view>
					</view>
				</view>
				<view class='score'>
					<view class='item acea-row row-middle' v-for="(item,indexw) in scoreList" :key="indexw">
						<view>{{item.name}}</view>
						<view class='starsList'>
							<text @click="stars(indexn, indexw)" v-for="(itemn, indexn) in item.stars" :key="indexn" class='iconfont' :class="item.index >= indexn? 'icon-shitixing font-color':'icon-kongxinxing'"></text>
						</view>
						<text class='evaluate'>{{item.index === -1 ? "" : item.index + 1 + "分"}}</text>
					</view>
					<view class='textarea'>
						<textarea placeholder='商品满足你的期待么？说说你的想法，分享给想买的他们吧~' name="comment" placeholder-class='placeholder'></textarea>
						<view class='list acea-row row-middle'>
							<view class='pictrue' v-for="(item,index) in picsPath" :key="index">
								<image :src='item'></image>
								<text class='iconfont icon-guanbi1 font-color' @click='DelPic(index)'></text>
							</view>
							<view class='pictrue acea-row row-center-wrapper row-column' @click='uploadpic' v-if="picsPath.length < 8">
								<text class='iconfont icon-icon25201'></text>
								<view>上传图片</view>
							</view>
						</view>
					</view>
					<button class='evaluateBnt bg-color' formType="submit">立即评价</button>
				</view>
			</view>
		</form>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
	</view>
</template>

<script>
	import {
		orderProduct,
		orderComment
	} from '@/api/order.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
		},
		data() {
			return {
				pics: [],
				picsPath: [],
				scoreList: [{
						name: "商品质量",
						stars: ["", "", "", "", ""],
						index: -1
					},
					{
						name: "服务态度",
						stars: ["", "", "", "", ""],
						index: -1
					}
				],
				orderId: '',
				productId: 0, //产品id
				evaluateId: 0, //评价id
				unique: '',
				productInfo: {},
				cart_num: 0,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				id: 0//订单id
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getOrderProduct();
					}
				},
				deep: true
			}
		},
		onLoad(options) {
			if (!options.unique || !options.uni || !options.id) return this.$util.Tips({
				title: '缺少参数'
			}, {
				tab: 3,
				url: 1
			});
			this.unique = options.unique;
			this.orderId = options.uni;
			this.id = options.id;
			this.evaluateId = options.id;
			if (this.isLogin) {
				this.getOrderProduct();
			} else {
				toLogin();
			}
		},
		methods: {
			onLoadFun() {
				this.getOrderProduct();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			/**
			 * 获取某个产品详情
			 * 
			 */
			getOrderProduct: function() {
				let that = this;
				orderProduct({
					orderId: that.id,
					uni: that.unique
				}).then(res => {
					that.$set(that, 'productInfo', res.data.productInfo);
					that.$set(that, 'cart_num', res.data.cartNum);
					that.$set(that, 'productId', res.data.productId);
				});
			},
			stars: function(indexn, indexw) {
				this.scoreList[indexw].index = indexn;
			},
			/**
			 * 删除图片
			 * 
			 */
			DelPic: function(index) {
				let that = this,
					pic = this.picsPath[index];
				that.picsPath.splice(index, 1);
				that.$set(that, 'picsPath', that.picsPath);
			},

			/**
			 * 上传文件
			 * 
			 */
			uploadpic: function() {
				let that = this;
				that.$util.uploadImageOne({
					url: 'user/upload/image',
					name: 'multipart',
					model: "product",
					pid: 1
				}, function(res) {
					that.pics.push(res.data.url);
					that.picsPath.push(res.data.localPath);
					that.$set(that, 'pics', that.pics);
					that.$set(that, 'picsPath', that.picsPath);
				});
			},

			/**
			 * 立即评价
			 */
			formSubmit: function(e) {
				let formId = e.detail.formId,
					value = e.detail.value,
					that = this,
					product_score = that.scoreList[0].index + 1 === 0 ? "" : that.scoreList[0].index + 1,
					service_score = that.scoreList[1].index + 1 === 0 ? "" : that.scoreList[1].index + 1;
				if (!value.comment) return that.$util.Tips({
					title: '请填写你对宝贝的心得！'
				});
				value.productScore = product_score;
				value.serviceScore = service_score;
				value.pics = that.pics.length>0?JSON.stringify(that.pics):'';
				value.productId = that.productId;
				value.oid = that.evaluateId;
				value.unique = that.unique;
				value.sku = that.productInfo.attrInfo.suk;
				uni.showLoading({
					title: "正在发布评论……"
				});
				orderComment(value).then(res => {
					uni.hideLoading();
					return that.$util.Tips({
						title: '感谢您的评价!',
						icon: 'success'
					}, '/pages/order_details/index?order_id=' + that.orderId);
				}).catch(err => {
					uni.hideLoading();
					return that.$util.Tips({
						title: err
					});
				});
			}
		}
	}
</script>

<style lang="scss" scoped>
	.evaluate-con .score {
		background-color: #fff;
		border-top: 1rpx solid #f5f5f5;
		font-size: 28rpx;
		color: #282828;
		padding: 48rpx 30rpx 65rpx 30rpx;
	}

	.evaluate-con .score .item~.item {
		margin-top: 30rpx;
	}

	.evaluate-con .score .item .starsList {
		padding: 0 35rpx 0 40rpx;
	}

	.evaluate-con .score .item .starsList .iconfont {
		font-size: 40rpx;
		color: #aaa;
	}

	.evaluate-con .score .item .starsList .iconfont~.iconfont {
		margin-left: 20rpx;
	}

	.evaluate-con .score .item .evaluate {
		color: #aaa;
		font-size: 24rpx;
	}

	.evaluate-con .score .textarea {
		width: 690rpx;
		background-color: #fafafa;
		border-radius: 10rpx;
		margin-top: 48rpx;
	}

	.evaluate-con .score .textarea textarea {
		font-size: 28rpx;
		padding: 38rpx 30rpx 0 30rpx;
		width: 100%;
		box-sizing: border-box;
		height: 160rpx;
	}

	.evaluate-con .score .textarea .placeholder {
		color: #bbb;
	}

	.evaluate-con .score .textarea .list {
		margin-top: 25rpx;
		padding-left: 5rpx;
	}

	.evaluate-con .score .textarea .list .pictrue {
		width: 140rpx;
		height: 140rpx;
		margin: 0 0 35rpx 25rpx;
		position: relative;
		font-size: 22rpx;
		color: #bbb;
	}

	.evaluate-con .score .textarea .list .pictrue:nth-last-child(1) {
		border: 1rpx solid #ddd;
		box-sizing: border-box;
	}

	.evaluate-con .score .textarea .list .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 3rpx;
	}

	.evaluate-con .score .textarea .list .pictrue .icon-guanbi1 {
		font-size: 45rpx;
		position: absolute;
		top: -20rpx;
		right: -20rpx;
	}

	.evaluate-con .score .textarea .list .pictrue .icon-icon25201 {
		color: #bfbfbf;
		font-size: 50rpx;
	}

	.evaluate-con .score .evaluateBnt {
		font-size: 30rpx;
		color: #fff;
		width: 690rpx;
		height: 86rpx;
		border-radius: 43rpx;
		text-align: center;
		line-height: 86rpx;
		margin-top: 45rpx;
	}
</style>
