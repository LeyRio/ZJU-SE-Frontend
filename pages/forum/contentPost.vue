<template>
	<view>
		<!-- 话题详情 -->
		<view class="topic-detail">
			<!-- 话题详情 -->
			<view>
				<!-- 话题头部 -->
				<view class="detail-header">
					<view class="topic-title">{{ topic.title }}</view>
					<view class="topic-header-info">
						<text class='time-info'>•修改于{{ topic.updateTime }}•作者 {{ topic.authorName }}</text>
						<text v-if="topic.authorPhone==userPhone || authType==3" class="delete-text" @tap="deleteTopic">删除</text>
						<text v-if="topic.authorPhone==userPhone" class="edit-text" @tap="editTopic">编辑</text>
						<!-- <text v-if="topic.authorPhone!=userPhone" class="report-text" @tap="">举报</text> -->
					</view>
				</view>
				<!-- 话题内容 -->
				<view class="detail-content">
					<text>{{ topic.content }}</text>
				</view>
				<!-- 点赞/踩 -->
				<view class="detail-like">
					<image v-if="likeState==true" class="info-icon" @tap="tapLike" src="../../static/forum/赞 面性.svg"></image>
					<image v-else class="info-icon" @tap="tapLike" src="../../static/forum/赞.svg"></image>
					<text class="info-cnt">{{ topic.likeCnt }}</text>
					<image v-if='likeState==false' class="info-icon" @tap="tapDislike" src="../../static/forum/踩 面性.svg"></image>
					<image v-else class="info-icon" @tap="tapDislike" src="../../static/forum/踩.svg"></image>
					<text class="info-cnt">{{ topic.dislikeCnt }}</text>
					<image class="info-icon" src="../../static/forum/阅读 可见.svg"></image>
					<text class="info-cnt">{{ topic.viewCnt }}</text>
					<image class="info-icon" src="../../static/forum/评论.svg"></image>
					<text class="info-cnt">{{ topic.replyCnt }}</text>
					<image v-if="favoriteState && hasLogin" class="info-icon" @tap="changeFavoriteState" src="../../static/forum/星星 面性.svg"></image>
					<image v-if="!favoriteState && hasLogin" class="info-icon" @tap="changeFavoriteState" src="../../static/forum/星星.svg"></image>
				</view>
				<!-- 话题回复 -->
				<view class="detail-reply">
					<view class="reply-list">
						<block v-for="(reply, replyIndex) of replies">
							<view class="reply">
								<view class="reply-header">
									<view class="reply-info">
										<text class="reply-author">{{ reply.userName}}</text>
										<text class="reply-time">{{ reply.floor }}楼•{{ reply.lastEditTime }}</text>
									</view>
									<view class="reply-delete">
										<text class="delete-text" v-if="userPhone==reply.userPhone || authType==3" @tap="removeReply(reply.replyId)">删除</text>
										<text class="edit-text" v-if="userPhone==reply.userPhone" @tap="editReply(reply.replyId, reply.content)">编辑</text>
										<text class="report-text" v-if="userPhone!=reply.userPhone && authType!=3" @tap="reportReply(reply.replyId)">举报</text>
									</view>
								</view>
								<view class="reply-content">
									<text>{{ reply.content }}</text>
								</view>
								<view class="detail-like">
									<image v-if="replyLikeInfo.likes.indexOf(reply.replyId) > -1" class="info-icon" @tap="tapReplyLike(reply.replyId)" src="../../static/forum/赞 面性.svg"></image>
									<image v-else class="info-icon" @tap="tapReplyLike(reply.replyId)" src="../../static/forum/赞.svg"></image>
									<text class="info-cnt">{{ reply.likeCnt }}</text>
									<image v-if='replyLikeInfo.disLikes.indexOf(reply.replyId) > -1' class="info-icon" @tap="tapReplyDislike(reply.replyId)" src="../../static/forum/踩 面性.svg"></image>
									<image v-else class="info-icon" @tap="tapReplyDislike(reply.replyId)" src="../../static/forum/踩.svg"></image>
									<text class="info-cnt">{{ reply.dislikeCnt }}</text>
								</view>
							</view>
						</block>
					</view>
				</view>
			</view>
		</view>
		
		<view @tap="onFloatButton()" class="plus">回复</view>
		
		
	</view>
</template>

<script>
const moment = require('moment')
import {getPost, addViewCnt, getLikeInfo, postLike, deleteLike, getFavoriteInfo, reportPostReply, getReplyLikeInfo,
			addToFavorite, removeFromFavorite, getTopicReplies, deleteReply, deletePost,
			postReplyLike, deleteReplyLike, getAuthInfo} from '../../fetch/api.js'
import store from "@/common/store.js"
export default {
	data() {
		return {
			// 加载层
			loading: false,
			// 话题ID
			topicId: null,
			topic: null,
			likeState: null,
			favoriteState: null,
			hasLogin : store.state.hasLogin,
			userPhone: store.state.uerInfo.userPhone,
			pageSize : 20,
			pageNo : 1,
			replies : [],
			replyLikeInfo: null,
			authType : store.state.uerInfo.authType
		}
	},
	methods: {
		onFloatButton() {
			uni.navigateTo({
				'url': './createReply?id=' + this.topicId
			})
		},
		onNavigationBarButtonTap(e) {
			uni.navigateTo({
				'url': './createReply?id=' + this.topicId
			})
		},
		// 获取话题详情
		async handleGetTopicDetail() {
			var topic = await getPost(this.topicId);
			this.topic = topic.data;
			this.topic.updateTime = moment(this.topic.updateTime * 1000).format('YYYY-MM-DD HH:mm:ss')
		},
		async loadLikeInfo() {
			const params = {
				"userPhone" : this.userPhone,
			};
			var likeState = await getLikeInfo(this.topicId, params)
			this.likeState = likeState.data
		},
		async loadReplyLikeInfo() {
			const params = {
				"userPhone" : this.userPhone,
				"pageSize" : 2147483647,
				"pageNo" : 1
			}
			var replyLikeInfo = await getReplyLikeInfo(this.topicId, params)
			this.replyLikeInfo = replyLikeInfo.data
		},
		async LoadFavoriteInfo() {
			const params = {
				"userPhone" : this.userPhone,
			};
			var favoriteState = await getFavoriteInfo(this.topicId, params)
			this.favoriteState = favoriteState.data
		},
		async signalLike() {
			const params = {
				"userPhone" : this.userPhone,
				"like" : 1
			};
			await postLike(this.topicId, params)
		},
		async signalDislike() {
			const params = {
				"userPhone" : this.userPhone,
				"like" : 0
			};
			await postLike(this.topicId, params)
		},		
		async signalClear() {
			const params = {
				"userPhone" : this.userPhone,
			};
			await deleteLike(this.topicId, params)
		},
		async tapLike() {
			if(this.hasLogin) {
				await this.signalClear()
				if (this.likeState != true)
					await this.signalLike()
				this.loadLikeInfo()
				this.handleGetTopicDetail()
			}
		},
		async tapDislike() {
			if(this.hasLogin) {
				await this.signalClear()
				if (this.likeState != false)
					await this.signalDislike()
				this.loadLikeInfo()
				this.handleGetTopicDetail()
			}
		},
		async signalReplyLike(replyId) {
			const params = {
				"userPhone" : this.userPhone,
				"like" : 1
			};
			await postReplyLike(replyId, params)
		},
		async signalReplyDislike(replyId) {
			const params = {
				"userPhone" : this.userPhone,
				"like" : 0
			};
			await postReplyLike(replyId, params)
		},		
		async signalReplyClear(replyId) {
			const params = {
				"userPhone" : this.userPhone,
			};
			await deleteReplyLike(replyId, params)
		},
		async tapReplyLike(replyId) {
			if(this.hasLogin) {
				await this.signalReplyClear(replyId)
				if (this.replyLikeInfo.likes.indexOf(replyId) == -1)
					await this.signalReplyLike(replyId)
				await this.loadReplyLikeInfo()
				await this.getReplies()
			}
		},
		async tapReplyDislike(replyId) {
			if(this.hasLogin) {
				await this.signalReplyClear(replyId)
				if (this.replyLikeInfo.disLikes.indexOf(replyId) == -1)
					await this.signalReplyDislike(replyId)
				await this.loadReplyLikeInfo()
				await this.getReplies()
			}
		},
		async changeFavoriteState() {
			const params = {
				"userPhone" : this.userPhone,
			};
			if (!this.favoriteState)
				await addToFavorite(this.topicId, params)
			else
				await removeFromFavorite(this.topicId, params)
			this.LoadFavoriteInfo()
		},
		async getReplies() {
			const params = {
				"pageSize" : this.pageSize,
				"pageNo" : this.pageNo
			}
			var replies = await getTopicReplies(this.topicId, params)
			this.replies = replies.data.postReplys
			replies = []
			for(var r in this.replies) {
				if(this.replies[r].content != "") {
					this.replies[r].lastEditTime = moment(this.replies[r].lastEditTime * 1000).format('YYYY-MM-DD HH:mm:ss')
					replies.push(this.replies[r])
				}
			}
			this.replies = replies
		},
		async removeReply(replyId) {
			await deleteReply(replyId)
			this.getReplies()
		},
		editReply(replyId, content) {
			uni.navigateTo({
				'url': './editReply?id=' + replyId + '&content=' + content
			})
		},
		async reportReply(replyId) {
			await reportPostReply(replyId)
			this.$util.toast('举报成功')
		},
		async deleteTopic() {
			await deletePost(this.topicId)
			var pages = getCurrentPages();
			var beforePage = pages[pages.length - 2];
			beforePage.refresh();
			uni.navigateBack({
				animationDuration: 500,
				animationType: 'pop-out'
			})
		},
		async editTopic() {
			uni.navigateTo({
				'url': './editPost?id=' + this.topicId + '&title=' + this.topic.title + '&content=' + this.topic.content
			})
		}
	},
	async onLoad(params) {
		if (params.id) {
			this.topicId = params.id
			console.log('Loading ' + this.topicId)
			await this.getReplies()
			await addViewCnt(this.topicId)
			this.LoadFavoriteInfo()
			this.loadLikeInfo()
			this.loadReplyLikeInfo()
			this.handleGetTopicDetail()
			console.log('Userinfo: ', store.state.uerInfo)
		}
	}
}
</script>

<style lang="scss">
	.plus{
		position: fixed;
		right: 50rpx;
		/* #ifdef H5 */
		bottom: 80px;
		/* #endif */
		/* #ifndef H5 */
		bottom: calc(var(--window-bottom) + 50rpx);
		/* #endif */
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100rpx;
		height: 100rpx;
		color: #333;
		background-color: #fff;
		box-shadow: 0 20rpx 60rpx 20rpx rgba(0, 0, 0, 0.2);
		font-size: 30rpx;
		border-radius: 50%;
		z-index: 999;
	}
// 话题详情
.topic-detail {
	width: 730rpx;
	margin: 0 15rpx 25rpx;
	background-color: #fff;
	border-radius: 6rpx;
	// 话题头部
	.detail-header {
		width: 690rpx;
		padding: 20rpx;
		border-bottom: #e5e5e5 solid 2rpx;
		.topic-title {
			color: #333;
			font-size: 22px;
			font-weight: 700;
		}
		.topic-header-info {
			font-size: 12px;
			color: #838383;
			.time-info {
				padding: 0rpx 10rpx 0rpx 0rpx;
			}
			.delete-text {
				padding: 0rpx 10rpx 0rpx 10rpx;
				color: #ff0000;
			}
			.edit-text {
				padding: 0rpx 10rpx 0rpx 10rpx;
				color: #ffaa00;
			}
			.report-text {
				padding: 0rpx 10rpx 0rpx 10rpx;
				color: #b4b4b4;
			}
		}
	}
	// 话题内容
	.detail-content {
		width: 690rpx;
		padding: 20rpx;
		border-bottom: #e5e5e5 solid 2rpx;
	}
	// 话题赞踩
	.detail-like {
		display: flex;
		flex-direction: row;
		width: 690rpx;
		height: 40rpx;
		padding: 20rpx;
		// margin-left: auto;
		margin-right: auto;
		border-bottom: #e5e5e5 solid 2rpx;
		
		.info-cnt {
			height: 40rpx;
			width: 80rpx;
			font-size: 30rpx;
			vertical-align: middle;
			line-height: 40rpx;
			margin: 0 0 0 10rpx;
		}
		
		.info-icon {
			height: 40rpx;
			width: 40rpx;
		}
	}
	// 话题回复
	.detail-reply {
		width: 730rpx;
		// 回复标题
		.reply-title {
			width: 690rpx;
			padding: 20rpx;
			height: 40rpx;
			line-height: 40rpx;
			font-size: 14px;
			color: #333;
			background-color: #f6f6f6;
		}
		// 回复列表
		.reply-list {
			width: 690rpx;
			padding: 0 20rpx 0 20rpx;
			.reply {
				padding: 15rpx 0 15rpx 0;
				border-bottom: #f0f0f0 solid 2rpx;
				.reply-header {
					height: 88rpx;
					display: flex;
					flex-flow: row nowrap;
					justify-content: flex-start;
					align-items: center;
					.reply-author-avatar {
						margin-right: 15rpx;
						image {
							width: 60rpx;
							height: 60rpx;
						}
					}
					.reply-info {
						.reply-author {
							margin-right: 15rpx;
							color: #666;
							font-size: 12px;
							font-weight: 700;
						}
						.reply-time {
							font-size: 11px;
							color: #08c;
						}
					}
					.reply-delete {
						margin-left: 20rpx;
						.delete-text {
							padding: 3rpx 10rpx 3rpx 10rpx;
							font-size: 11px;
							color: #ff0000;
						}
						.edit-text {
							padding: 3rpx 10rpx 3rpx 10rpx;
							font-size: 11px;
							color: #ffaa00;
						}
						.report-text {
							padding: 3rpx 10rpx 3rpx 10rpx;
							font-size: 11px;
							color: #b4b4b4;
						}
					}
				}
				.reply-content {
				}
			}
		}
	}
}
</style>
