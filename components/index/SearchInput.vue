<template>
	<view class="search-input" >
		<image src="https://zs-1256645015.cos.ap-guangzhou.myqcloud.com/trace/2020/7/3/c333853f-7426-4e72-8458-2071bc1562e1.png" class="logo" mode=""></image>
		<navigator class="search" url="../../pages/HM-search/HM-search">
			 <icon type="search" class="icon" size="16"/>
			 <text class="desc">搜索商品</text>
		</navigator>
	</view>
</template>

<script>
	export default{
		methods:{
			searchInput(){
				uni.navigateTo({
					url:'../../pages/HM-search/HM-search'
				})
			}
		}
	}
</script>

<style lang="less" scoped>
.search-input {
	background-color: #fff;
	display: flex;
	align-items: center;
	box-sizing: border-box;
	padding: 14upx 32upx;
	// margin-right: 20upx;
	color: #06B720;
	.logo {
		width: 58upx;
		height: 58upx;
		margin-right: 21upx;
	}  
	.search {
			margin-left: 20upx;
		display: flex;
		align-items: center;
		justify-content: center;
		height: 58upx;
		width: 520upx;
		background-color: #F2F2F2;
		border-radius: 29upx;
		color: #808080;
		font-size: 26upx;
		.desc {
			// letter-spacing: 1upx;
			margin-left: 18upx;
		}
	}
}
</style>

