<template>
	<view class="goods-list-item" @tap="itemClick">
		<view class="image-wrap">
			<view class="loadig-wrap" v-if="isLoading">
				<image src="https://s1.ax1x.com/2018/12/06/F1mHTs.gif" class="loading"></image>
			</view>
			<image :src="product.image || defaultImage" class="good-image" @load="imageLoad"></image>
		</view>
		<view class="good-info">
			<view class="good-name">{{product.spuName || product.skuName}}</view>
			<view class="good-price">
				<view class="left">￥{{product.initPirce || product.initPrice}}</view>
				<view class="right" @tap.stop="addCart(product,$event)" :data-img="product.image">
					<image src="../../../static/images/add.png" mode=""></image>
				</view>
			</view>
		</view>
		<shopCarAnimation ref="carAnmation" cartx="0.45" carty="1.1"></shopCarAnimation>
	</view>
</template>

<script>
	import throttle from '../../../common/throttle.js';
	import shopCarAnimation from '@/components/fly-in-cart/fly-in-cart.vue'
	export default {
		components: {
			shopCarAnimation
		},
		props: {
			product: {
				type: Object,
				default () {
					return {
						specSelected: []
					}
				}
			}
		},
		data() {
			return {
				isLoading: true
			}
		},
		computed: {
			defaultImage() {
				return "https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/5/15/aff1856e-27c6-4a25-8591-6ee448b9cc8d.jpg"
			}

		},
		methods: {

			// 加入购物车
			addShopCar(e) {
				
				console.log('加入购物车');
				// 成功的话，调用加入购物车动画
				this.$refs.carAnmation.touchOnGoods(e);
			},
			addCart(item,$event) {
					this.addShopCar($event)
				console.log(item)
				this.getProductDetail(item.spuId)
			},
			getProductDetail(spId) {
				this.$common
					.httpGet('common-goods/common/spuDetail', {
						spuId: spId,
						system: 'UPMALL'
					}, false)
					.then(res => {

						if (res.data.code === 200200) {
							this.specChildList = this.returnSpec(res.data.data.spec, 2);


							this.specSelected = []
							this.specChildList.forEach((item, idx) => {
								item.forEach((sonItem, sonIdx) => {
									this.specSelected.push(sonItem.specValueId)
								})
							})
							this.getStandardInfo(spId)
						} else {}
					});
			},
			getStandardInfo(spId) {
				console.log(this.specSelected)
				let specId = this.specSelected[0];
				// for (let i = 0; i < this.specSelected.length; i++) {
				// 	specId.push(this.specSelected[i].specValueId);
				// }

				this.$common
					.httpGet('common-goods/common/skuSpec', {
						spuId: spId,
						specValueIdList: String(specId),
						system: 'UPMALL'
					}, false)
					.then(res => {
						if (res.data.code === 200200) {
							this.specDetail = res.data.data;
							// this.optionInfo.skuId = this.specDetail.skuId;
							this.addShoppingCart(this.specDetail.skuId)
							// this.getSkuPrice(res.data.data.skuId);
							// this.$common.httpGet('common-goods/upMall/userPrice/sku/'+this.specDetail.skuid).then(res => {

							// })
						} else {
							this.$common.showToast(res.data.desc, 'none');
						}
					});
			},
			// 加入购物车
			addShoppingCart: throttle.throttleFunc(function(skuId) {
				// let shareId = this.options.share ? this.options.share : '';
				// this.$common.passUserauth(shareId).then(res => {
				// 	if (res) {
				// setTimeout(() => {
				console.log(skuId)
				this.$common.httpGet('common-goods/common/shoppingCart/addition', {
					skuId: skuId,
					increase: true,
					count: 1,
					system: 'UPMALL'
				}, false).then(data => {
					// this.$common.showToast(res.data.desc);
					if (data.data.code === 200200) {
						// this.$common.showToast(data.data.desc)
					
						// let timeid=	setTimeout(()=>{
						// this.$common.hideLoading()
						// clearTimeout(timeid)
						// },1000)
					}
				})
				// }, 1000)

				// 	} else {
				// 		this.$common.navigateTo('../auth/auth?share=' + shareId)
				// 	}
				// })
			}, 1800),
			// 把规格拆分出来
			returnSpec(speList, type) {
				let specList = [];
				let specChildList = [];
				if (type === 1) {
					speList.forEach(item => {
						specList.push({
							id: item.specId,
							name: item.specName
						});
					});
					return specList;
				} else {
					speList.forEach(speitem => {
						let _specChildList = []
						speitem.value.forEach(childItem => {
							_specChildList.push({
								specValueId: childItem.specValueId,
								specValue: childItem.specValue
							});
						});
						specChildList.push(_specChildList)
						// for (let childItem of speitem) {
						// 	specChildList.push({
						// 		specValueId: childItem.specValueId,
						// 		specValue: childItem.specValue
						// 	})
						// }
					});
					return specChildList;
				}
			},
			itemClick() {
				this.$common.navigateTo('../product/product?spu=' + this.product.spuId);
			},
			// 监听图片的加载事件
			imageLoad() {
				this.isLoading = false
			}
		}
	}
</script>

<style lang="less" scoped>
	.goods-list-item {
		display: flex;
		flex: 1;
		width: calc(100vw - 40rpx);
		// flex-direction: column;
		background: #ffffff;
		border-radius: 20upx;
		box-shadow: 0px 5upx 1px 0px #bdbdbd; 
		margin-bottom: 30upx;

		.image-wrap {
			position: relative;
		}

		.good-image {
			width: 250upx;
			height: 250upx;
			margin: 10upx;
		}

		.loadig-wrap {
			position: absolute;
			top: 0;
			left: 0;
			display: flex;
			justify-content: center;
			align-items: center;
			width: 100%;
			height: 327upx;

			.loading {
				width: 80upx;
				height: 80upx;
			}
		}

		.good-info {
			padding: 15upx 15upx 25upx;
			flex: 1;
			display: flex;
			flex-direction: column;
			justify-content: space-between;

			.good-name {
				font-size: 26upx;
				color: #000;
				margin-bottom: 20upx;
				overflow: hidden;
				display: -webkit-box;
				-webkit-line-clamp: 2;
				-webkit-box-orient: vertical;
				// height: 34upx;
			}

			.good-price {
				font-size: 28upx;
				color: #FF731D;
				font-weight: bold;
				display: flex;
				justify-content: space-between;
				width: 100%;

				.right {
					width: 48upx;
					height: 48upx;

					image {
						width: 100%;
						height: 100%;
					}
				}
			}
		}
	}
</style>
