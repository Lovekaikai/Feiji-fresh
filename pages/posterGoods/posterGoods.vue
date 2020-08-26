<template>
	<view class="posterGoods">
		<view class="wucTabBox1">
			<uni-nav-bar :status-bar="true" background-color="#FFFFFF" color="#000000" left-icon="arrowleft" title="代理商模块"
			 :shadow='false' fixed="true" @click-left="back" />
		</view>
		<view class="header">
			<view class="banner">
				<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/7/b423e68d-512c-4928-bf22-7436c00b2a3a.png"
				 mode="widthFix"></image>
			</view>
		</view>
		<view class="bg" style="background-image: url(https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/11/5e0f267b-0767-4e36-983c-81561256751c.png);">
			<view class="userBox">
				<view class="userLeft">
					<view class="userImg">
						<image :src="userInfo.avatar" mode=""></image>
					</view>
					<view class="userTitle">
						<view class="userName">
							{{userInfo.nickName}}
						</view>
						<view class="userMomey">
							账户余额：{{userInfo.balance}}
						</view>
					</view>
				</view>
				<view class="shoppingCat">
					<view class="item" @tap="goShoppingCart">
						<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/11/6de0672f-b333-4f8b-9abf-5c738b1179f3.png"
						 mode=""></image>
					</view>
					<view class="item" @tap="goMyorder">
						<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/11/e87be062-1f57-40a9-a99f-9efa533873d5.png"
						 mode=""></image>
					</view>
				</view>
			</view>
			<view class="menuNav">
				<!-- 分类 -->
				<scroll-view scroll-x="true" class="navbar-wrap">
					<view class="nav-item" v-for="(item, index) in classifyTitle" :key="index">
						<view class="nav-content">
							<view class="nav-info" @tap="changeTitle(index)" :class="{navActive: index === currentIndex}">
								<view class="title">{{ item.name }}</view>
								<!-- <view class="desc">今日折扣</view> -->
							</view>
						</view>
					</view>
				</scroll-view>
				<view class="goodsList">
					<view class="goods-item" v-if="classifyGoods.length > 0" v-for="(goods,classif) in classifyGoods" :key="classif"
					 @tap="itemClick(goods.spuId)">
						<view class="goodsImg">
							<image :src="goods.image || defaultImage" mode=""></image>
						</view>
						<view class="goodsInfo">
							<view class="goodsName">
								{{goods.spuName}}
							</view>
							<view class="goodsPrice">
								<view class="oldPrice">
									<view class="jian">
										<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/11/3b998a68-7970-475e-8c0a-dc6e4bd1ef89.png"
										 mode=""></image>
									</view>
									<view class="original">
										￥{{goods.initPrice}}
									</view>
								</view>
								<view class="newPrice">
									<view class="price">
										￥{{goods.newPrice}}
									</view>
									<view class="slogan">
										代理商超值价
									</view>
									<view class="shoppingCat">
										<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/11/f0cd4cea-8661-4b57-8019-6d97b100df34.png"
										 mode=""></image>
									</view>
								</view>
							</view>
						</view>
					</view>
					<view class="noMore" v-if="classifyGoods.length > 4">
						<div class="line"></div>
						<div class="desc">没有更多数据了</div>
						<div class="line"></div>
					</view>
					<view class="empty" v-show="classifyEmpty">暂无相关数据</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import WucTab from '../../components/wuc-tab/wuc-tab.vue';
	import throttle from '../../common/throttle.js';
	export default {
		components: {
			WucTab
		},
		data() {
			return {
				currentIndex: 0,
				classifyTitle: [], // 分类标题
				classifyGoods: [], //商品列表
				classifyEmpty: false,
				categoryRequest: {
					topCategoryId: "", // 顶级分类id
					pageNum: 1,
					pageSize: 10
				},
				total: 0,
				userInfo: {}, //用户信息
				olduserInfo: {}, //用户信息
				openId: 'okY-q5f5q3QyuROea9DNuekKQYdo'
			};
		},
		onShow() {
			this.olduserInfo = this.$common.getStorage('userInfo')
			this.openId = this.$common.getStorage('userInfo').weixinOpenid
			this.getUserInfo() //代理商用户信息
		},
		created() {
			this.getClassifyTitle(); // 获取分类标题
		},
		computed: {
			defaultImage() {
				return "https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/5/15/aff1856e-27c6-4a25-8591-6ee448b9cc8d.jpg"
			}

		},
		methods: {
			// 跳转购物车
			goShoppingCart() {
				uni.navigateTo({
					url: './shoppingCart/shoppingCart'
				});
			},
			// 跳转我的订单
			goMyorder() {
				uni.navigateTo({
					url: './MeOrder/MeOrder'
				});
			},
			// 跳转商品详情
			itemClick(e) {
				this.$common.navigateTo('./product/product?spu=' + Number(e));
			},
			// 获取分类标题
			getClassifyTitle() {
				this.$common.httpGet('common_goods_agent/upMall/category/top').then(res => {
					// console.log(res)
					const {
						code,
						data
					} = res.data
					if (code === 200200) {
						this.classifyTitle = data
						this.changeTitle(0) // 初始化默认选中第一个分类title
					}
				})
			},
			// 切换分类标题触发
			changeTitle(index) {
				this.classifyGoods = [] // 每次点击要初始化classifyGoods列表值
				this.categoryRequest.pageNum = 1 // 每次点击要初始化当前页为1
				this.currentIndex = index
				this.categoryRequest.topCategoryId = this.classifyTitle[index].categoryId
				this.getClassifyGoods() // 每次点击获取对应的分类商品列表
			},
			// 获取分类商品
			getClassifyGoods() {
				this.$common.httpGet('common_goods_agent/upMall/category/includeGoods', this.categoryRequest).then(res => {
					let {
						code,
						data
					} = res.data
					if (code === 200200) {
						this.classifyEmpty = data.list.length > 0 ? false : true // 有数据则隐藏空 无则显示空
						this.total = data.total // 分类商品的总数
						// 旧数据
						let oldclassifyGoods = this.classifyGoods;
						//新数据
						const newclassifyGoods = data.list;
						// 新旧数据合并
						this.classifyGoods = [...oldclassifyGoods, ...newclassifyGoods]
						// console.log(this.classifyGoods)
					}
				})
			},
			// 获取用户信息
			getUserInfo() {
				this.$common.httpGet('user_agent/userMember/getUserAgentInfo?openId=' + this.openId).then((res) => {
					console.log(res)
					if (Number(res.data.code) === 100200) {
						if (res.data.data) {
							this.userInfo = res.data.data
						} else {
							this.userInfo = this.olduserInfo
							this.userInfo.balance = 0
							this.userInfo.userLevel = 0
						}
						let agentInfo = {
							userLevel: this.userInfo.userLevel,
							balance: this.userInfo.balance,
							nickName: this.userInfo.nickName,
							avatar: this.userInfo.avatar,
							memberId: this.userInfo.memberId
						}
						uni.setStorageSync('agentInfo', agentInfo);
					} else {
						uni.showToast({
							title: res.data.message,
							duration: 3000,
							icon: 'none'
						});
					}
				})
			},
			// 定义导航
			back: throttle.throttleFunc(function() {
				uni.switchTab({
					url: '../profile/profile'
				})		
			}, 1500)
		},
		onReachBottom() {
			// 检查是否有还有下一页
			const totalPageNum = Math.ceil(this.total / this.categoryRequest.pageSize);
			if (this.categoryRequest.pageNum < totalPageNum) {
				this.categoryRequest.pageNum++;
				this.getClassifyGoods();
			}
		}
	}
</script>

<style lang="less">
	.posterGoods {
		.wucTabBox1 {
			position: fixed;
			left: 0;
			/* #ifdef MP-WEIXIN */
			top: 0upx;
			/* #endif */
			width: 100%;
			z-index: 100;
		}
		.header {
			margin-top: 176rpx;
			.banner {
				width: 100%;

				image {
					width: 100%;
					display: block;
				}
			}
		}

		.bg {
			margin-top: -1px;
			width: 100%;
			min-height: 69.5vh;
			padding: 20upx 32upx 30upx 32upx;
			box-sizing: border-box;

			.userBox {
				box-sizing: border-box;
				width: 100%;
				height: 144upx;
				background-color: #FFFFFF;
				display: flex;
				justify-content: space-between;
				align-items: center;
				border-radius: 25upx;
				padding: 30upx 40upx;

				.userLeft {
					display: flex;

					.userImg {
						width: 84upx;
						height: 84upx;

						image {
							width: 100%;
							height: 100%;
						}
					}

					.userTitle {
						margin-left: 20upx;
						display: flex;
						flex-direction: column;
						justify-content: space-between;

						.userName {
							font-weight: 600;
							font-size: 30upx;
						}

						.userMomey {
							font-size: 24upx;
							color: #CFA56B;
						}
					}
				}

				.shoppingCat {
					display: flex;

					.item {
						margin: 0 20upx;
						width: 40upx;
						height: 40upx;

						image {
							width: 100%;
							height: 100%;
						}
					}
				}
			}

			.menuNav {
				box-sizing: border-box;
				width: 100%;
				border-radius: 25upx;
				background-color: #FFFFFF;
				margin-top: 30upx;

				.navbar-wrap {
					white-space: nowrap;
					padding-bottom: 16upx;
					background-color: #FFFFFF;
					// height: 60upx;
					border-top-left-radius: 25upx;
					border-top-right-radius: 25upx;
					padding-top: 20upx;

					.nav-item {
						display: inline-block;
						text-align: center;

						.nav-content {
							display: flex;
							align-items: center;

							.nav-info {
								margin: 0 30upx;
								// padding-bottom: 20upx;
								border-bottom: 1px solid #FFFFFF;

								.title {
									font-size: 28upx;
									color: #808080;
									margin-bottom: 6upx;
									font-weight: bold;
									padding-bottom: 20upx;
								}
							}

							.navActive {
								.title {
									font-size: 32upx;
									color: #CFA56B;
									border-bottom: 1px solid #CFA56B;
								}
							}
						}


					}
				}

				.goodsList {
					padding-bottom: 20upx;

					// min-height: 400upx;
					.goods-item {
						margin: 20upx 20upx;
						display: flex;
						padding-bottom: 20upx;
						border-bottom: 1px solid #D8D8D8;

						.goodsImg {
							width: 240upx;
							height: 240upx;

							image {
								width: 100%;
								height: 100%;
							}
						}

						.goodsInfo {
							width: 400upx;
							box-sizing: border-box;
							padding-left: 15upx;
							display: flex;
							flex-direction: column;
							justify-content: space-between;

							.goodsName {
								font-size: 28upx;
								font-weight: 600;
							}

							.goodsPrice {
								.oldPrice {
									font-size: 26upx;
									color: #808080;
									display: flex;

									.jian {
										width: 15upx;
										height: 80upx;
										margin-top: 10upx;

										image {
											width: 100%;
											height: 100%;
										}
									}

									.original {}
								}

								.newPrice {
									display: flex;
									justify-content: space-between;
									align-items: center;

									.price {
										font-size: 35upx;
										color: #CFA56B;
										font-weight: 600;
									}

									.slogan {
										font-size: 22upx;
										color: #CFA56B;
									}

									.shoppingCat {
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
					}

					.noMore {
						display: flex;
						justify-content: center;
						align-items: center;
						padding: 40upx 0 20upx 0;

						.line {
							width: 180upx;
							height: 1px;
							background-color: #eee;
						}

						.desc {
							font-size: 24upx;
							color: #ccc;
							padding: 0 20upx;
						}

					}

					.empty {
						color: #999;
						text-align: center;
						font-size: 26rpx;
						width: 100%;
						padding: 100rpx 0;
					}
				}
			}
		}
	}
</style>
