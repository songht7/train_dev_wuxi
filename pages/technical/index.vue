<template>
	<view class="library">
		<view class="page-main">
			<block v-for="(obj,index) in datas" :key="index">
				<view class="lib-row" @click="goDetail(obj.id)">
					<view class="lib-block">
						<view class="lib-more">
							<view class="lib-m-left">
								<view class="lib-title">{{obj.name}}</view>
								<view class="lib-ov">{{obj.overview}}</view>
							</view>
							<view class="lib-m-right">
								<image class="image-full" :src="obj.original_src?obj.original_src:sourceUrl+'/data/image_doc/358aaf312fbb4cac05b05044b5a0e824.png'"
								 mode="aspectFill"></image>
							</view>
						</view>
					</view>
				</view>
			</block>
			<uni-load-more :status="status"></uni-load-more>
		</view>
		<fix-button>
			<view class="fbtns fbtns-clr-full btn-totest" @click="$store.dispatch('makePhoneCall')">
				<uni-icon type="dianhua1" size="25" color="#fff"></uni-icon>咨询我们
			</view>
		</fix-button>
	</view>
</template>

<script>
	import uniLoadMore from '@/components/uni-load-more.vue'
	import fixButton from '@/components/fix-button.vue'
	export default {
		data() {
			return {
				ctgId: "",
				datas: [],
				data_total: 0,
				pageIndex: 1,
				pageSize: 5,
				status: "more"
			}
		},
		onLoad: function(e) {
			var that = this;
			that.ctgId = e.ctg_id || 16;
		},
		onShow() {
			var that = this;
			that.$store.dispatch('cheack_user')
			that.getDatas()
		},
		onReady: function(res) {},
		onPullDownRefresh() {
			var that = this;
			that.pageIndex = 1;
			that.getDatas()
		},
		onReachBottom() {
			var that = this;
			if (that.status === "noMore") {
				return;
			}
			if (that.datas.length >= that.data_total || that.data_total <= 0) {
				that.status = "noMore";
				return;
			}
			that.pageIndex = that.pageIndex + 1;
			that.getDatas()
		},
		components: {
			fixButton,
			uniLoadMore
		},
		computed: {},
		methods: {
			getDatas() {
				var that = this;
				that.status = "loading";
				/*技术支持*/
				let data = {
					"inter": "supports",
					"parm": `?cat_id=${that.ctgId}&currentPage=${that.pageIndex}&pagesize=${that.pageSize}`,
					"header": {
						"token": that.$store.state.user.token || ""
					}
				}
				data["fun"] = function(res) {
					that.status = "more";
					uni.stopPullDownRefresh()
					if (res.success) {
						var _data = res.data.list;
						if (_data) {
							if (that.pageIndex == 1) {
								that.datas = _data;
							} else {
								console.log(_data)
								//that.datas.push(_data);
								_data.forEach(item => {
									that.datas.push(item);
								});
							}
							that.data_total = res.data.total;
						}
						if (that.datas.length >= res.data.total || res.data.total <= 0) {
							that.status = "noMore";
							return;
						}
					}
				}
				that.$store.dispatch("getData", data)
			},
			goDetail: function(id) {
				uni.navigateTo({
					url: "/pages/technical/detail?id=" + id
				})
			},
		}
	}
</script>

<style>
	.lib-row {
		background: #FFFFFF;
	}

	.lib-block {
		padding: 30upx 20upx;
	}

	.lib-block,
	.lib-more {
		display: flex;
		justify-content: flex-start;
		align-content: center;
		flex-direction: column;
	}

	.lib-more {
		flex-direction: row-reverse;
		justify-content: space-between;
		color: #929292;
		font-size: 26upx;
		line-height: 1.4;
	}

	.lib-title {
		color: #222222;
		font-size: 34upx;
		line-height: 1.6;
		padding-bottom: 10upx;
	}


	.lib-ov {
		line-height: 1.4;
		padding-bottom: 20upx;
	}

	.lib-m-left {
		display: flex;
		align-content: space-between;
		flex-direction: column;
		flex: 1;
		padding-left: 40upx;
	}

	.lib-m-right {
		width: 175upx;
		height: 175upx;
		border-radius: 50%;
		overflow: hidden;
	}
</style>
