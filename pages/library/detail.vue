<template>
	<view class="article-detail">
		<block v-if="datas.id">
			<view class="banner">
				<image class="banner-img" :src="sourceUrl+datas.original_src"></image>
				<!-- <view class="banner-title">{{datas.name}}</view> -->
			</view>
			<view class="banner-title-block">{{datas.name}}</view>
			<view class="article-meta">
				<text class="article-time">{{datas.add_time}}</text>
				<text class="article-author">{{datas.author_name?datas.author_name:"管理员"}}</text>
			</view>
			<view class="article-content">
				<rich-text :nodes="datas.detail"></rich-text>
			</view>
		</block>
		<fix-button btnType="fbtn-big">
			<view class="fbtns collect" :class="isCollect?'collected':''" @click="collect(datas.id)">
				<view>
					<uni-icon :type="isCollect?'shoucang1':'shoucang'" size="25" :color="isCollect?'#008CEE':'#929292'"></uni-icon>
					<view>收藏</view>
				</view>
			</view>
		</fix-button>
	</view>
</template>
<script>
	import fixButton from '@/components/fix-button.vue'
	export default {
		data() {
			return {
				article_id: "",
				isCollect: false,
				datas: []
			}
		},
		onLoad(e) {
			var that = this;
			that.article_id = e.id;
		},
		onShow() {
			var that = this;
			that.$store.dispatch('cheack_user')
			that.getDatas()
		},
		components: {
			fixButton
		},
		methods: {
			getDatas() {
				var that = this;
				/*文库*/
				let data = {
					"inter": "support",
					"parm": `?article_id=${that.article_id}`,
					"header": {
						"token": that.$store.state.user.token || ""
					}
				}
				data["fun"] = function(res) {
					if (res.success) {
						let _data = res.data;
						that.datas = _data;
						if (_data.favorites === 1) {
							that.isCollect = true;
						} else {
							that.isCollect = false;
						}
						uni.setNavigationBarTitle({
							title: _data.name
						});
					}
				}
				that.$store.dispatch("getData", data)
			},
			collect(id) {
				var that = this;
				/*加入收藏*/
				let data = {
					"inter": "favorite",
					"method": "POST",
					"data": {
						"article_id": that.article_id
					},
					"header": {
						"Content-Type": "application/json",
						"token": that.$store.state.user.token || ""
					}
				}
				data["fun"] = function(res) {
					if (res.success) {
						var toast_title = !that.isCollect ? "已加入收藏" : "取消收藏";
						uni.showToast({
							title: toast_title,
							icon: "success"
						});
						that.isCollect = !this.isCollect
					}
				}
				that.$store.dispatch("getData", data)
			}
		}
	}
</script>

<style>
	.banner {
		height: 360upx;
		overflow: hidden;
		position: relative;
		background-color: #ccc;
	}

	.banner-img {
		width: 100%;
	}

	.banner-title {
		max-height: 84upx;
		overflow: hidden;
		position: absolute;
		left: 30upx;
		bottom: 30upx;
		width: 90%;
		font-size: 32upx;
		font-weight: 400;
		line-height: 42upx;
		color: white;
		z-index: 11;
	}

	.banner-title-block {
		padding: 30upx 3% 10upx;
		font-size: 44upx;
		font-weight: 500;
		color: #222222;
	}

	.article-meta {
		padding: 20upx 40upx;
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
		color: gray;
	}

	.article-text {
		font-size: 26upx;
		line-height: 50upx;
		margin: 0 20upx;
	}

	.article-author,
	.article-time {
		font-size: 30upx;
		padding-right: 20upx;
	}

	.article-content {
		padding: 0 30upx;
		overflow: hidden;
		font-size: 30upx;
		margin-bottom: 30upx;
	}
</style>
