<template>
	<view>
		<view class="bgColor" :style="skinMode?'':'background-color: rgba(0,0,0,0.2)'"></view>
		<view v-for="(item,index) in data_list" :key="item.news_id">
			<view v-if="item.delete_mark==0">
				<view class="p-2 animated fast fadeIn">
					<!-- 标题 -->
					<view class="my-1" style="line-height: 1.4;" :class="fontSizeTitle" @click="openDetail(item.news_id)">{{item.title}}</view>
					<view class="news_info flex align-center">
						<view class="author_time flex justify-start" @click="openDetail(item.news_id)">
							<view class="mr-2" style="color: #999999;line-height: 1.3;" :class="fontSizeDetail">{{item.media_name}}</view>
							<text style="color: #9D9589;line-height: 1.3;" :class="fontSizeDetail">{{timestampToTime_(item.publish_time)}}</text>
						</view>
						<view class="star_digg flex">
							<view class="star" @click="collect_new(item)">
								<image :src="item.is_star==1?'../../static/comment_bar/star_1.png':'../../static/comment_bar/star_0.png'" mode="aspectFit"
								 class="star_img"></image>
							</view>
							<view class="digg" @click="digg_new(item)">
								<image :src="item.is_digg == 1?'../../static/comment_bar/like_1.png':'../../static/comment_bar/like_0.png'" mode="aspectFit" 
								class="digg_img"></image>
								<text :class="['comments_digg_count',item.is_digg == 1?'change_color':'']">{{item.digg_count}}</text>
							</view>
						</view>
					</view>
				</view>
			</view>
			<divider></divider>
		</view>
	</view>
</template>

<script>
	import divider from '@/components/custom/divider/divider.vue'
	export default {
		components: {
			divider,
		},
		data() {
			return {
				userid: 0,
				data_list: [],
				skinMode: true,
				fontSize: "normal"
			}
		},
		computed: {
			fontSizeTitle() {
				return (this.fontSize == 'big') ? 'font-md' : ((this.fontSize == 'normal') ? 'font' : 'font-sm')
			},
			fontSizeDetail() {
				return (this.fontSize == 'big') ? 'font' : ((this.fontSize == 'normal') ? 'font-sm' : 'font-small')
			}
		},
		onLoad() {
			//加载skinMode的值
			try {
				const value = uni.getStorageSync('skin_mode');
				if (value !== '') {
					this.skinMode = value
				}
			} catch (e) {
				// error
			}
			//加载fontSize的值
			try {
				const value = uni.getStorageSync('font-size');
				//console.log(value);
				if (value !== '') {
					this.fontSize = (value == 0) ? 'big' : ((value == 1) ? 'normal' : 'small')
				}
			} catch (e) {
				// error
			}
		},
		onShow() {
			//获取用户id
			this.data_list = []
			const user_id = uni.getStorageSync('user_id');
			this.userid = user_id;

			//获取新闻id列表
			uni.request({
				url: "https://af1o32.toutiao15.com/getCollectByUid2",
				data: {
					user_id: this.userid,
				},
				method: 'POST',
				success: (res) => {
					this.data_list = res.data.result;
					console.log(this.data_list);
					for(var i = 0;i < data_list.length;i++)
					{
						this.data_list[i].publish_time = "123432";
					}
				}
			});

			
		},
		onPullDownRefresh() {
			uni.redirectTo({
				url: '../myCollect/myCollect'
			});
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 1000);
		},
		methods: {
			//时间转换
			timestampToTime_(timestamp) {
				var date = new Date(timestamp * 1000); //timestamp 为10位需*1000，timestamp 为13位的话不需乘1000
				var Y = date.getFullYear() + '-';
				var M = (date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1) + '-';
				var D = (date.getDate() < 10 ? '0' + date.getDate() : date.getDate()) + ' ';
				var h = (date.getHours() < 10 ? '0' + date.getHours() : date.getHours()) + ':';
				var m = (date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes()) + ':';
				var s = (date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds());
				return Y + M + D + h + m + s;
			},

			//点击进入详情页面
			openDetail(news_id) {
				uni.navigateTo({
					url: "../../pages/info/info?news_id=" + news_id
				})
			},
			//点击收藏按钮
			collect_new(item) {
				uni.request({
					url: 'https://af1o32.toutiao15.com/collection',
					method: "POST",
					data: {
						news_id: item.news_id,
						user_id: this.userid
					},
					success: res => {
						//console.log(res);
					}
				});
				item.delete_mark = 1;
			},

			//点击点赞按钮
			digg_new(item) {
				if (item.is_digg == 1) {
					item.is_digg = 0;
					item.digg_count = item.digg_count - 1;
					uni.request({
						url: 'https://af1o32.toutiao15.com/digg_cancel',
						method: "POST",
						data: {
							type: "news",
							id: item.news_id,
							user_id: this.userid
						}
					});
				} else {
					item.is_digg = 1;
					item.digg_count = item.digg_count + 1;
					uni.request({
						url: 'https://af1o32.toutiao15.com/digg',
						method: "POST",
						data: {
							type: "news",
							id: item.news_id,
							user_id: this.userid
						}
					});
				}
			},

		}
	}
</script>

<style>
	.bgColor {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		z-index: -1;
	}
	.news_info{
		justify-content: space-between;
	}
	.star_digg{
		margin-right: 10rpx;
	}
	.star_img {
		width: 55rpx;
		height: 55rpx;
	}
	.digg{
		margin-left: 10rpx;
	}
	.digg_img {
		position: relative;
		width: 55rpx;
		height: 55rpx;
	}
	.comments_digg_count {
		font-size: 18rpx;
		position: relative;
		bottom:40rpx;
		left: -10rpx;
		color: #707070;
	}
	.change_color {
		color: #F45858;
	}
</style>
