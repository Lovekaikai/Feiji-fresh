<template>
	<view class="menu-nav">
		<!-- 导航 -->

			<scroll-view scroll-x="true" class="navbar-wrap1" v-if="positionNarBar" :class="{positionNarBar:positionNarBar}" :style="{isHeight:isHeight}"
		 id="main">
			<view class="nav-item" v-for="(item, index) in classifyTitle" :key="index">
				<view class="nav-content">
					<view class="nav-info" @tap="handleNav(index)" :class="{navActive: index === currentIndex}">
						<image :src="imgArr[index]" mode="widthFix"></image>
						<view class="title">{{ item.name }}</view>
						<view class="desc">{{item.description}}</view>
					</view>
				</view>
			</view>
		</scroll-view>
		<div class="navbar-wrap">
			<view class="nav-item" v-for="(item, index) in classifyTitle" :key="index">
				<view class="nav-content">
					<view class="nav-info" @tap="handleNav(index)" :class="{navActive: index === currentIndex}">
						<image :src="imgArr[index]" mode="widthFix"></image>
						<view class="title">{{ item.name }}</view>
						<view class="desc">{{item.description}}</view>
						<!-- <view class="desc">今日折扣</view> -->
					</view>
					<!-- <view class="line"></view> -->
				</view>
			</view>
		</div>


		<!-- 主体 -->
		<view v-if="list.length > 0 &&currentIndex!==0" class="goodsList-wrap">
			<goods-list :goods="list"></goods-list>
			<view class="noMore" v-if="list.length > 4">
				<div class="line"></div>
				<div class="desc">没有更多数据了</div>
				<div class="line"></div>
			</view>
		</view>
		<joint-item v-if="currentIndex===0" @tap.native="goGoodsDetail(item)" v-for="(item,index) in list" :key="index" :item="item"
		 :navIndex="navIndex" :hideLine="index == (jointList.length-1)">
		</joint-item>

		<view class="empty" v-show="classifyEmpty">暂无相关数据</view>
	</view>
</template>

<script>
	import throttle from '@/common/throttle.js';
	import GoodsList from './goods/GoodsList.vue'
	import JointItem from "../joint-item/JointItem.vue";
	export default {
		data() {
			return {
				imgArr: [
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/11b075fc-745d-42a0-ba46-f71eca598acd.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/c333853f-7426-4e72-8458-2071bc1562e1.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/f93ce1de-6cbd-4c76-86e7-502583f9fb4e.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/20a71b40-6819-42fe-8097-18bf2568a4af.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/419cf3c8-2ae3-4c9f-b751-9ade5b70972c.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/f83acbbb-1825-4b86-9765-9d0973e81c78.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/9bf21b62-85db-4116-ae9a-b4f64eb275f2.png',
					'https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/41196948-696d-4c9a-bfa3-b22bdd5f5221.png'
				],
				NarTop: '',
				positionNarBar: false
			}
		},
		props: {
			// 分类标题
			classifyTitle: {
				type: Array,
				default () {
					return []

				}
			},
			NavTopMuch: {
				type: Number
			},
			// 分类商品列表
			list: {
				type: Array,
				default () {
					return []
				}
			},

			// 标题当前索引
			currentIndex: {
				type: Number,
				default: 0
			},

			// 控制是否为空
			classifyEmpty: {
				type: Boolean,
				default: false
			}
		},
		components: {
			GoodsList,
			JointItem
		},
		watch: {
			classifyTitle: {
				handler() {
					setTimeout(() => {
						const query = uni.createSelectorQuery().in(this);
						query.select('.menu-nav').boundingClientRect(data => {
							this.NarTop = data.top
							// this.$emit("navTop",this.NarTop)
						}).exec();
					}, 800)
				},
				deep: true
			},
			NavTopMuch(val) {

				if (Math.ceil(val) - (this.NarTop + 200) > 0) {
					this.positionNarBar = true
				} else {
					this.positionNarBar = false
				}
			}
		},
		computed: {
			isHeight() {
				let result = this.classifyTitle.some(v => v.description.length > 0)
				return result ? uni.upx2px(100) + 'px' : uni.upx2px(160) + 'px'
			}
		},

		methods: {
			goGoodsDetail: throttle.throttleFunc(function(item) {
				if (this.navIndex === 1) return
				this.$common.navigateTo('../groupGoodsDetails/groupGoodsDetails?spuId=' + item.spuId);
			}, 500),
			// 点击切换菜单
			handleNav(index) {
				this.$emit('change', index)
			}
		}
	};
</script>

<style lang="less" scoped>
	.menu-nav {
		margin-top: 20upx;
		background-color: #fff;
		min-height: 500upx;

		.positionNarBar {
			z-index: 100;
			position: fixed;
			top: 0;
			left: 0;
			height: 60px;
		}
	
		.navbar-wrap1{
			overflow-x: visible;
			white-space: nowrap;
			background-color: #f2f2f2;
			height: 170upx;
			.nav-item {
				display: inline-block;
				// display: flex;
				width: 25%;
				text-align: center;
				padding-bottom: 30upx;
				.nav-content {
					display: flex;
					align-items: center;
			
					.nav-info {
						padding: 0 20upx;
			
						.title {
							font-size: 26upx;
							color: #1F1F1F;
							margin-bottom: 6upx;
							font-weight: bold;
						}
			
						image {
							height: 96upx;
							width: 86upx;
						}
			
						.desc {
							padding: 2upx 12upx;
							font-size: 22upx;
							color: #7F7F7F;
						}
					}
			
					.navActive {
						.title {
							color: #06B720;
						}
			
						.desc {
							background: #06B720;
							color: #fff;
							border-radius: 18upx;
						}
					}
				}
			
				&:not(:last-child) {
					.line {
						width: 2upx;
						height: 32upx;
						background-color: #B1B1B1;
					}
				}
			
			}
		}
		.navbar-wrap {
			white-space: nowrap;
			background-color: #f2f2f2;
			// height: 170upx;
			width: 100vw;
			flex-wrap: wrap;
			display: flex;
			.nav-item {
				display: inline-block;
				// display: flex;
				width: 25%;
				text-align: center;
				padding-bottom: 30upx;
				.nav-content {
					display: flex;
					align-items: center;

					.nav-info {
						padding: 0 20upx;

						.title {
							font-size: 26upx;
							color: #1F1F1F;
							margin-bottom: 6upx;
							font-weight: bold;
						}

						image {
							height: 96upx;
							width: 86upx;
						}

						.desc {
							padding: 2upx 12upx;
							font-size: 22upx;
							color: #7F7F7F;
						}
					}

					.navActive {
						.title {
							color: #06B720;
						}

						.desc {
							background: #06B720;
							color: #fff;
							border-radius: 18upx;
						}
					}
				}

				&:not(:last-child) {
					.line {
						width: 2upx;
						height: 32upx;
						background-color: #B1B1B1;
					}
				}

			}
		}

		.goodsList-wrap {
			background-color: #fff;
			min-height: 400upx;
			
			.noMore {
				display: flex;
				justify-content: center;
				align-items: center;
				padding: 160upx 0 20upx 0;

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
		}

		.empty {
			color: #999;
			text-align: center;
			font-size: 26rpx;
			width: 100%;
			padding: 100rpx 0;
		}
	}
</style>
