<template>
	<view class="user-center">
		<user-center-top :enterpriseUserCount="enterpriseUserCount" :joinCourseUserCount="joinCourseUserCount" :courseCount="courseCount"></user-center-top>
		<view class="user-block" v-show="statisType===0">
			<view class="user-class-list">
				<view class="my-class-head">
					<view class="class-tip">
						<view class="class-icon">
							<uni-icon type="shuji" :size="20" color="#FFFFFF"></uni-icon>
						</view>
						<view class="txt-sross">员工学习进度</view>
					</view>
					<view class="class-more">全部{{data_total}}个></view>
				</view>
				<view class="class-list">
					<view class="list-statis fRowCenter">
						<view class="row-block fRowCenter">排名</view>
						<view class="row-block fRowCenter">学员</view>
						<view class="row-block fRowCenter">参与课程</view>
						<view class="row-block fRowCenter">通过考试</view>
						<view class="row-block fRowCenter row-progress">学习总进度</view>
					</view>
					<view class="list-row class-list-row" v-for="(obj,k) in datas" :key="k">
						<view class="list-block">
							<view class="list-statis fRowCenter">
								<view class="row-block fRowCenter">{{k}}</view>
								<view class="row-block fRowCenter">
									<img :src="sourceUrl+obj.photo" class="photo"/>
									{{obj.NAME}}
								</view>
								<view class="row-block fRowCenter">{{obj.joinCourseCount}}</view>
								<view class="row-block fRowCenter">{{obj.passExamCount}}</view>
								<view class="row-block fRowCenter row-progress">
									<progress :percent="obj.progress" stroke-width="4" activeColor="#008CEE" backgroundColor="#E0E0E0" />
									{{parseInt(obj.progress)}}%
								</view>
							</view>
						</view>
					</view>
					<uni-load-more :status="status"></uni-load-more>
				</view>
			</view>
		</view>
		<view class="user-block" v-show="statisType==1||statisType==2">
			<view class="user-class-list">
				<view class="my-class-head">
					<view class="class-tip">
						<view class="class-icon">
							<uni-icon type="shuji" :size="20" color="#FFFFFF"></uni-icon>
						</view>
						<view class="txt-sross">{{statisType==1?'课程参与情况':'课程合格情况'}}</view>
					</view>
					<view class="class-more">全部{{data_total}}个></view>
				</view>
				<view class="class-list">
					<view class="class-list">
						<view class="list-row class-list-row" v-for="(obj,k) in datas" :key="k">
							<view class="list-block" @click="navToTrain(obj.id)">
								<view class="list-more">
									<view class="list-left class-list-left">
										<view class="list-title">{{obj.name}}</view>
										<view class="class-progress">
											<view class="progress-box">
												<view class="percent">{{statisType==1?"参与度"+parseInt(obj.progress)+"%":"合格率"+parseInt(obj.progress)+"%"}}</view>
												<progress :percent="obj.progress" stroke-width="4" activeColor="#008CEE" backgroundColor="#E0E0E0" />
											</view>
										</view>
									</view>
									<view class="list-right">
										<image class="image-full" :src="obj.original_src?sourceUrl+obj.original_src:sourceUrl+'/data/image_doc/358aaf312fbb4cac05b05044b5a0e824.png'"
										 mode="aspectFill"></image>
									</view>
								</view>
							</view>
						</view>
					</view>
					<uni-load-more :status="status"></uni-load-more>
				</view>
			</view>
		</view>
		<tab-bar></tab-bar>
	</view>
</template>

<script>
	import userCenterTop from '@/components/user-center-top.vue'
	import uniGrid from '@/components/uni-grid.vue'
	import uniLoadMore from '@/components/uni-load-more.vue'
	export default {
		data() {
			return {
				UserId: "",
				statisType: 0,
				userCenterDatas: {},
				enterpriseUserCount: "0", //员工总数
				joinCourseUserCount: "0", //参与学习
				courseCount: "0", //总课程数
				datas: [],
				data_total: 0,
				pageIndex: 1,
				pageSize: 5,
				status: "more",
				interList: [{
					"inter": "personalProgresses",
					"dataFor": "personal"
				}, {
					"inter": "courseProgresses",
					"dataFor": "course"
				}, {
					"inter": "testProgresses",
					"dataFor": "test"
				}]
			}
		},
		components: {
			userCenterTop,
			uniGrid,
			uniLoadMore
		},
		computed: {},
		onLoad(e) {
			var that = this;
			let p = e.t || 0
			that.statisType = parseInt(p);
			that.$store.dispatch("cheack_page", p)
			console.log(that.statisType)
		},
		onShow() {
			var that = this;
			that.$store.dispatch('cheack_user');
			if (that.$store.state.user.userInfo == undefined) {
				return
			}
			let _user = that.$store.state.user.userInfo;
			that.UserId = _user.id || '';
			that.enterpriseUserCount = _user.subInfo.enterpriseUserCount || '0';
			that.joinCourseUserCount = _user.subInfo.joinCourseUserCount || '0';
			that.courseCount = _user.subInfo.courseCount || '0';
			that.getDatas()
			// interList.forEach((obj, key) => {
			// 	that.getDatas(obj.inter, obj.dataFor)
			// })
		},
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
		methods: {
			getDatas() {
				var that = this;
				let data = {
					"inter": that.interList[that.statisType].inter,
					"parm": `?currentPage=${that.pageIndex}&pagesize=${that.pageSize}`,
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
			navToTrain(id) {
				uni.navigateTo({
					url: `/pages/train/unitlist?id=${id}`
				})
			}
		}
	}
</script>

<style>
	@import "../../common/center.css";

	.fRowCenter {
		display: flex;
		flex-direction: row;
		align-content: center;
		align-items: center;
		justify-content: center;
		text-align: center;
	}

	.row-block {
		width: 18%;
	}

	.row-progress {
		width: 28%;
	}

	.photo {
		width: 60upx;
		height: 60upx;
		border-radius: 50%;
		display: block;
		margin-right: 4upx;
	}
</style>
