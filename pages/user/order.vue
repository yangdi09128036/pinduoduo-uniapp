<template>
	<view class="container">
		<!-- 固定头部区域 -->
		<view class="fixed-header">
			<!-- 状态栏占位 -->
			<view class="status-bar"></view>

			<!-- 顶部导航栏 -->
			<view class="nav-header">
				<view class="back-icon" @click="navigateBack">
					<image src="/static/left.png" class="icon-back"></image>
				</view>
				<text class="page-title">我的订单</text>
			</view>

			<!-- 选项卡导航 -->
			<scroll-view scroll-x="true" class="tab-scroll" :scroll-into-view="'tab'+currentTab"
				:scroll-with-animation="true" @scroll="handleScroll">
				<view class="tabs">
					<view v-for="(tab, index) in tabs" :key="index" :id="'tab'+index" class="tab-item"
						:class="{ active: currentTab === index }" @click="switchTab(index)">
						{{ tab }}
						<view class="tab-line" v-if="currentTab === index"></view>
					</view>
				</view>
			</scroll-view>
		</view>

		<!-- 内容区域 -->
		<swiper class="swiper-content" :current="currentTab" @change="handleSwiperChange">
			<!-- 全部 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[0] && orderList[0].length > 0">
							<view v-for="(order, orderIndex) in orderList[0]" :key="orderIndex" class="order-item">
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">{{ getOrderStatus(order) }}</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="order-actions">
											<!-- 待付款状态 -->
											<template v-if="order.paymentStatus === 0">
												<button class="action-btn cancel"
													@click="cancelOrder(order)">取消订单</button>
												<button class="action-btn primary" @click="goToPay(order)">去支付</button>
											</template>
											<!-- 待分享状态 -->
											<template v-else-if="order.paymentStatus === 1 && order.shareStatus === 0">
												<button class="action-btn cancel"
													@click="cancelOrder(order)">取消订单</button>
												<button class="action-btn primary"
													@click="shareOrderWithDelay(order)">去分享</button>
											</template>
											<!-- 待收货状态 -->
											<template
												v-else-if="order.shippingStatus === 1 && order.deliveryStatus === 0">
												<button class="action-btn primary"
													@click="confirmReceipt(order)">确认收货</button>
											</template>
											<!-- 未评价状态 -->
											<template
												v-else-if="order.deliveryStatus === 1 && order.reviewStatus === 0">
												<button class="action-btn primary"
													@click="showReviewPopup(order)">去评价</button>
											</template>
										</view>
									</view>
								</view>

							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有相关的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 待付款 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[1] && orderList[1].length > 0">
							<view v-for="(order, orderIndex) in orderList[1]" :key="orderIndex" class="order-item">
								<!-- 待付款订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">待付款</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="order-actions">
											<button class="action-btn cancel" @click="cancelOrder(order)">取消订单</button>
											<button class="action-btn primary" @click="goToPay(order)">去支付</button>
										</view>
									</view>
								</view>

							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有待付款的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 待分享 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[2] && orderList[2].length > 0">
							<view v-for="(order, orderIndex) in orderList[2]" :key="orderIndex" class="order-item">
								<!-- 待分享订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">待分享</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="order-actions">
											<button class="action-btn cancel" @click="cancelOrder(order)">取消订单</button>
											<button class="action-btn primary"
												@click="shareOrderWithDelay(order)">去分享</button>
										</view>
									</view>
								</view>



							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有待分享的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 待发货 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[3] && orderList[3].length > 0">
							<view v-for="(order, orderIndex) in orderList[3]" :key="orderIndex" class="order-item">
								<!-- 待发货订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">待发货</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
									</view>
								</view>
							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有待发货的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 待收货 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[4] && orderList[4].length > 0">
							<view v-for="(order, orderIndex) in orderList[4]" :key="orderIndex" class="order-item">
								<!-- 待收货订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">待收货</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage" mode="aspectFill" class="product-image"
											@click="navigateToProductDetails(order.productId)">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="order-actions">
											<button class="action-btn primary"
												@click="confirmReceipt(order)">确认收货</button>
										</view>
									</view>
								</view>

							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有待收货的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 已评价 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[5] && orderList[5].length > 0">
							<view v-for="(order, orderIndex) in orderList[5]" :key="orderIndex" class="order-item">
								<!-- 已评价订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">已评价</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="review-content">
											<text class="review-label">我的评价：</text>
											<text class="review-text">{{ order.review }}</text>
										</view>
									</view>
								</view>
							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您还没有已评价的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<!-- 未评价 -->
			<swiper-item>
				<scroll-view scroll-y="true" class="order-list">
					<view class="orders-section">
						<template v-if="orderList[6] && orderList[6].length > 0">
							<view v-for="(order, orderIndex) in orderList[6]" :key="orderIndex" class="order-item">
								<!-- 未评价订单内容 -->
								<view class="store-info">
									<image class="store-icon" src="/static/avatar-default.png" mode="aspectFit"></image>
									<text class="store-name">拼多多官方旗舰店</text>
									<text class="order-status">未评价</text>
								</view>
								<view class="product-info">
									<view class="product-details">
										<image :src="order.productImage"
											@click="navigateToProductDetails(order.productId)" mode="aspectFill"
											class="product-image">
										</image>
										<text class="product-name">{{ order.productName }}</text>
										<text class="product-desc">{{ order.description }}</text>
										<view class="price-info">
											<text class="quantity">x{{ order.quantity }}</text>
											<text class="price">¥{{ order.amount }}</text>
										</view>
										<view class="order-actions">
											<button class="action-btn primary"
												@click="showReviewPopup(order)">去评价</button>
										</view>
									</view>
								</view>

							</view>
						</template>
						<view v-else class="empty-state">
							<image src="/static/empty-order.png" mode="aspectFit" class="empty-icon"></image>
							<text>您没有未评价的订单</text>
						</view>
					</view>
				</scroll-view>
			</swiper-item>
		</swiper>

		<!-- 评价弹窗 -->
		<uni-popup ref="reviewPopup" type="bottom">
			<view class="review-popup">
				<view class="popup-header">
					<image src="/static/left.png" class="icon-back-popup" @click="hideReviewPopup"></image>
					<text class="popup-title">商品评价</text>
				</view>
				<view class="review-content-popup">
					<textarea v-model="reviewContent" placeholder="请输入您的评价内容" class="review-textarea" />
					<button class="submit-review" @click="submitReview">提交评价</button>
				</view>
			</view>
		</uni-popup>

		<!-- 分享提示弹窗 -->
		<uni-popup ref="sharePopup" type="center">
			<view class="share-popup">
				<view class="share-popup-content">
					<text class="share-popup-title">分享成功</text>
					<text class="share-popup-text">3秒后将自动为您发货</text>
					<text class="share-popup-countdown">{{ shareCountdown }}s</text>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import {
		store
	} from '@/uni_modules/uni-id-pages/common/store.js'

	export default {
		computed: {
			userInfo() {
				return store.userInfo
			}
		},
		data() {
			return {
				tabs: ['全部', '待付款', '待分享', '待发货', '待收货', '已评价', '未评价'],
				currentTab: 0,
				orderList: {
					0: [], // 全部
					1: [], // 待付款
					2: [], // 待分享
					3: [], // 待发货
					4: [], // 待收货
					5: [], // 已评价
					6: [] // 新增：未评价
				},
				orderCounts: {
					1: 0, // 待付款
					2: 0, // 待分享
					3: 0, // 待发货
					4: 0, // 待收货
					5: 0, // 已评价
					6: 0 // 新增：未评价
				},
				randomGoods: [], // 推荐商品列表
				reviewContent: '', // 评价内容
				currentOrderForReview: null, // 当前要评价的订单
				shareCountdown: 5, // 分享倒计时
				shareTimer: null // 分享倒计时定时器
			}
		},
		onLoad(options) {
			// 根据传入的type参数设置当前选项卡
			if (options.type) {
				const typeMap = {
					'pending': 1,
					'share': 2,
					'shipping': 3,
					'receiving': 4,
					'review': 5
				}
				this.currentTab = typeMap[options.type] || 0
			}
			this.loadOrderData()
			this.getRandomGoods()
		},
		onShow() {
			// 每次页面显示时刷新订单数据
			this.loadOrderData()
		},
		onUnload() {
			// 清除定时器
			if (this.shareTimer) {
				clearInterval(this.shareTimer)
			}
		},
		methods: {
			async loadOrderData() {
				try {
					if (!this.userInfo || !this.userInfo._id) {
						uni.showToast({
							title: '请先登录',
							icon: 'none'
						})
						return
					}

					const db = uniCloud.database()

					// 重置订单计数
					for (let i = 1; i <= 5; i++) {
						this.orderCounts[i] = 0
					}

					// 加载当前标签页的订单
					let query = {
						userId: this.userInfo._id
					}

					// 根据当前标签页添加查询条件
					switch (this.currentTab) {
						case 1: // 待付款
							query.paymentStatus = 0
							break
						case 2: // 待分享
							query.paymentStatus = 1
							query.shareStatus = 0
							break
						case 3: // 待发货
							query.paymentStatus = 1
							query.shippingStatus = 0
							break
						case 4: // 待收货
							query.shippingStatus = 1
							query.deliveryStatus = 0
							break
							// 在loadOrderData方法中，修改case 5的查询条件
						case 5: // 已评价
							query.deliveryStatus = 1
							query.reviewStatus = 1
							break

							// 新增case 6的查询条件
						case 6: // 未评价
							query.deliveryStatus = 1
							query.reviewStatus = 0
							break
					}

					const result = await db.collection('order')
						.where(query)
						.orderBy('createdAt', 'desc')
						.get()

					if (result.result.data) {

						this.orderList[this.currentTab] = result.result.data

						// 只有在待发货状态且有订单数据时才执行自动发货
						if (this.currentTab === 3 && this.orderList[this.currentTab].length > 0) {
							this.orderList[this.currentTab].forEach(order => {
								if (order.shippingStatus === 0) {
									this.startAutoShipping(order)
								}
							})
						}
					} else {
						this.orderList[this.currentTab] = []
					}

					// 如果是全部选项卡，加载所有订单
					if (this.currentTab === 0) {
						const allResult = await db.collection('order')
							.where({
								userId: this.userInfo._id
							})
							.orderBy('createdAt', 'desc')
							.get()

						if (allResult.result.data) {
							this.orderList[0] = allResult.result.data
						} else {
							this.orderList[0] = []
						}
					}

					// 获取各状态的订单数量
					await this.getOrderCounts()

					// 将订单数量存储到本地，供用户页面使用
					uni.setStorageSync('orderCounts', this.orderCounts)

				} catch (error) {
					console.error('加载订单数据失败:', error)
					uni.showToast({
						title: '加载订单数据失败',
						icon: 'none'
					})
				}
			},
			async getOrderCounts() {
				try {
					const db = uniCloud.database()

					const queries = [
						// 待付款
						db.collection('order').where({
							userId: this.userInfo._id,
							paymentStatus: 0
						}).count(),

						// 待分享
						db.collection('order').where({
							userId: this.userInfo._id,
							paymentStatus: 1,
							shareStatus: 0
						}).count(),

						// 待发货
						db.collection('order').where({
							userId: this.userInfo._id,
							paymentStatus: 1,
							shippingStatus: 0
						}).count(),

						// 待收货
						db.collection('order').where({
							userId: this.userInfo._id,
							shippingStatus: 1,
							deliveryStatus: 0
						}).count(),

						// 已评价
						db.collection('order').where({
							userId: this.userInfo._id,
							deliveryStatus: 1,
							reviewStatus: 1
						}).count(),

						// 新增：未评价
						db.collection('order').where({
							userId: this.userInfo._id,
							deliveryStatus: 1,
							reviewStatus: 0
						}).count()
					]

					const results = await Promise.all(queries)

					for (let i = 0; i < results.length; i++) {
						this.orderCounts[i + 1] = results[i].result.total
					}

				} catch (error) {
					console.error('获取订单数量失败:', error)
				}
			},
			async getRandomGoods() {
				try {
					const {
						result
					} = await uniCloud.database().collection('mall-goods').limit(8).get()
					this.randomGoods = this.shuffleArray(result.data || [])
				} catch (err) {
					console.error('获取商品数据失败:', err)
				}
			},
			shuffleArray(array) {
				if (!Array.isArray(array)) {
					return []
				}
				const newArray = [...array]
				for (let i = newArray.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[newArray[i], newArray[j]] = [newArray[j], newArray[i]]
				}
				return newArray
			},
			getOrderStatus(order) {
				if (order.paymentStatus === 0) return '待付款'
				if (order.paymentStatus === 1 && order.shareStatus === 0) return '待分享'
				if (order.paymentStatus === 1 && order.shippingStatus === 0) return '待发货'
				if (order.shippingStatus === 1 && order.deliveryStatus === 0) return '待收货'
				if (order.deliveryStatus === 1 && order.reviewStatus === 0) return '未评价' // 修改这里
				return '已完成'
			},
			switchTab(index) {
				this.currentTab = index;
				this.loadOrderData();
				console.log(`切换到选项卡 ${index}，数据:`, this.orderList[index]);
			},
			handleSwiperChange(e) {
				this.currentTab = e.detail.current;
				this.loadOrderData();
				console.log(`切换到选项卡 ${this.currentTab}，数据:`, this.orderList[this.currentTab]);
			},
			async goToPay(order) {
				const paymentData = {
					amount: order.amount,
					userId: this.userInfo._id,
					username: this.userInfo.username,
					mobile: this.userInfo.mobile,
					avatar: this.userInfo.avatar_file?.path || '/static/avatar-default.png',
					productId: order.productId,
					productName: order.productName,
					productImage: order.productImage,
					quantity: order.quantity
				}
				uni.setStorageSync('paymentData', paymentData)
				uni.setStorageSync('currentOrderIds', [order._id]) // 使用数组存储单个订单ID
				uni.navigateTo({
					url: '/pages/wallet/pay'
				})
			},
			async cancelOrder(order) {
				try {
					const db = uniCloud.database()
					await db.collection('order').doc(order._id).update({
						paymentStatus: 2 // 取消状态
					})
					uni.showToast({
						title: '订单已取消',
						icon: 'success'
					})
					this.loadOrderData()
				} catch (error) {
					console.error('取消订单失败:', error)
					uni.showToast({
						title: '取消订单失败',
						icon: 'none'
					})
				}
			},
			shareOrderWithDelay(order) {
				// 显示分享倒计时弹窗
				this.shareCountdown = 3
				this.$refs.sharePopup.open()

				// 设置倒计时
				this.shareTimer = setInterval(() => {
					this.shareCountdown--
					if (this.shareCountdown <= 0) {
						clearInterval(this.shareTimer)
						this.$refs.sharePopup.close()
						this.shareOrder(order)
					}
				}, 1000)
			},
			async shareOrder(order) {
				try {
					const db = uniCloud.database()
					await db.collection('order').doc(order._id).update({
						shareStatus: 1 // 已分享状态
					})
					uni.showToast({
						title: '分享成功',
						icon: 'success'
					})
					this.loadOrderData()

					// 3秒后自动发货
					setTimeout(async () => {
						try {
							await db.collection('order').doc(order._id).update({
								shippingStatus: 1 // 已发货状态
							})
							console.log('订单自动发货成功:', order._id)
							this.loadOrderData()
						} catch (error) {
							console.error('自动发货失败:', error)
						}
					}, 3000)
				} catch (error) {
					console.error('分享订单失败:', error)
					uni.showToast({
						title: '分享失败',
						icon: 'none'
					})
				}
			},
			async confirmReceipt(order) {
				try {
					const db = uniCloud.database()
					await db.collection('order').doc(order._id).update({
						deliveryStatus: 1, // 已收货状态
						reviewStatus: 0 // 设置为未评价状态
					})

					// 显示评价弹窗
					this.showReviewPopup(order)

					this.loadOrderData()
				} catch (error) {
					console.error('确认收货失败:', error)
					uni.showToast({
						title: '确认收货失败',
						icon: 'none'
					})
				}
			},
			showReviewPopup(order) {
				this.currentOrderForReview = order
				this.reviewContent = ''
				// 确保popup组件存在
				if (this.$refs.reviewPopup) {
					this.$refs.reviewPopup.open()
				} else {
					console.error('评价弹窗组件不存在')
					uni.showToast({
						title: '评价功能暂时不可用',
						icon: 'none'
					})
				}
			},
			hideReviewPopup() {
				if (this.$refs.reviewPopup) {
					this.$refs.reviewPopup.close()
				}
				// 不需要在这里做任何状态更新，保持订单为"未评价"状态
				this.currentOrderForReview = null
				this.reviewContent = ''
			},
			async submitReview() {
				if (!this.reviewContent.trim()) {
					uni.showToast({
						title: '请输入评价内容',
						icon: 'none'
					});
					return;
				}

				try {
					const db = uniCloud.database();
					const timestamp = new Date().getTime();

					// 1. 更新订单表
					await db.collection('order').doc(this.currentOrderForReview._id).update({
						review: this.reviewContent,
						reviewStatus: 1, // 设置为已评价
						updatedAt: timestamp
					});

					// 2. 更新商品表（追加到评价数组）
					let productId = this.currentOrderForReview.productId;
					console.log('商品id', productId);
					// 处理 productId 可能是数组的情况
					if (Array.isArray(productId)) {
						productId = productId[0];
					}

					// 获取商品当前信息
					const goodsRes = await db.collection('mall-goods').doc(productId).get();
					console.log('已经存在的商品数据', goodsRes);
					if (!goodsRes.result.data) {
						throw new Error('商品不存在');
					}

					// 确保 reviews 是数组
					let currentReviews = goodsRes.result.data[0].reviews || [];
					console.log('已经存在的商品的评论数据', currentReviews);

					// 添加新评价
					currentReviews = [...currentReviews, this.reviewContent];

					console.log('更新后的的商品的评论数据', currentReviews);

					// 更新商品评价
					await db.collection('mall-goods').doc(productId).update({
						reviews: currentReviews
					});

					// 获取更新后的商品数据
					const endgoodsRes = await db.collection('mall-goods').doc(productId).get();
					console.log('更新后的商品数据:', endgoodsRes);

					uni.showToast({
						title: '评价成功',
						icon: 'success'
					});
					this.hideReviewPopup();
					this.loadOrderData();
				} catch (error) {
					console.error('提交评价失败:', error);
					uni.showToast({
						title: '提交评价失败: ' + (error.message || '请重试'),
						icon: 'none'
					});
				}
			},
			// 计算平均评分的方法
			calculateAverageRating(reviews) {
				if (!reviews || reviews.length === 0) return 5
				const sum = reviews.reduce((total, review) => total + (review.rating || 5), 0)
				return Math.round(sum / reviews.length * 10) / 10
			},
			navigateToProduct(item) {
				if (!item) return
				uni.setStorage({
					key: 'currentProduct',
					data: item,
					success: () => {
						uni.navigateTo({
							url: '../search/mall-details'
						})
					}
				})
			},
			navigateBack() {
				uni.switchTab({
					url: '/pages/user/user'
				})
			},
			handleScroll() {
				// 滚动事件处理（如需要）
			},
			startAutoShipping(order) {
				// 3秒后自动发货
				setTimeout(async () => {
					try {
						const db = uniCloud.database()
						await db.collection('order').doc(order._id).update({
							shippingStatus: 1 // 已发货状态
						})
						console.log('订单自动发货成功:', order._id)
						this.loadOrderData()
					} catch (error) {
						console.error('自动发货失败:', error)
					}
				}, 5000)
			},
			navigateToProductDetails(productId) {
				if (!productId) {
					uni.showToast({
						title: '商品信息不完整',
						icon: 'none'
					})
					return
				}

				// 获取商品详情
				console.log("跳转商品详情页的商品id", productId);

				// 处理可能为Proxy(Array)的情况
				let idString = productId;
				if (typeof productId === 'object' && productId[0]) {
					idString = productId[0];
				}

				const db = uniCloud.database()
				db.collection('mall-goods').doc(idString).get().then(res => {
					if (res.result.data) {
						console.log("跳转商品详情页的商品数据", res.result.data);
						// 确保存储的是单个商品对象而不是数组
						const goodsData = Array.isArray(res.result.data) ? res.result.data[0] : res.result.data;
						uni.setStorage({
							key: 'currentProduct',
							data: goodsData,
							success: () => {
								uni.navigateTo({
									url: '../search/mall-details'
								})
							}
						})
					} else {
						uni.showToast({
							title: '商品不存在或已下架',
							icon: 'none'
						})
					}
				}).catch(err => {
					console.error('获取商品详情失败:', err)
					uni.showToast({
						title: '获取商品详情失败',
						icon: 'none'
					})
				})
			}
		}
	}
</script>

<style scoped>
	.status-bar {
		height: 35px;
		background-color: #ffffff;
	}

	.review-popup {
	  background: #fff;
	  border-radius: 20rpx 20rpx 0 0;
	  /* 移除外层 padding，改为内部控制 */
	}
	
	.popup-header {
	  display: flex;
	  align-items: center;
	  padding: 30rpx 30rpx 0 30rpx; /* 标题区域单独设置 padding */
	}
	
	.popup-title{
		margin-left: 10rpx;
		font-size: 32rpx;
		font-weight: 600;
	}
	.review-content-popup {
	  padding: 20rpx 30rpx 30rpx 30rpx; /* 内容区域单独设置 padding */
	}
	
	.review-textarea {
	  height: 500rpx;
	  border: 1rpx solid #000;
	  border-radius: 10rpx;
	  margin-bottom: 30rpx;
	  padding: 20rpx; /* 调整内边距 */
	  width: 100%; /* 确保宽度填满父容器 */
	  box-sizing: border-box; /* 防止边框和 padding 影响宽度 */
	}

	.submit-review {
		background-color: #e02e24;
		color: #fff;
		border-radius: 10rpx;
		height: 88rpx;
		line-height: 88rpx;
		font-size: 32rpx;
	}

	/* 分享弹窗样式 */
	.share-popup {
		background: #fff;
		border-radius: 12rpx;
		padding: 40rpx;
		width: 80%;
		max-width: 500rpx;
	}

	.share-popup-content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.share-popup-title {
		font-size: 36rpx;
		font-weight: bold;
		margin-bottom: 20rpx;
	}

	.share-popup-text {
		font-size: 28rpx;
		color: #666;
		margin-bottom: 20rpx;
	}

	.share-popup-countdown {
		font-size: 48rpx;
		color: #e02e24;
		font-weight: bold;
	}

	.action-btn.primary {
		background-color: #e02e24;
		color: #fff;
		border-radius: 10rpx;
		border: none;
	}

	.action-btn.cancel {
		background-color: #fff;
		color: #666;
		border-radius: 10rpx;
		border: 1rpx solid #ddd;
	}

	.action-btn.disabled {
		background-color: #ccc;
		color: #fff;
		border-radius: 10rpx;
		border: none;
	}

	.recommend-title {
		text-align: center;
		color: #666;
		font-size: 28rpx;
		margin: 30rpx 0;
	}

	.container {
		display: flex;
		flex-direction: column;
		height: 100vh;
		background-color: #f8f8f8;
	}

	.fixed-header {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		background-color: #fff;
	}

	.nav-header {
		display: flex;
		align-items: center;
		justify-content: center;
		background-color: #fff;
	}

	.back-icon {
		padding: 20rpx;
		margin-left: -20rpx;
	}

	.icon-back {
		margin-left: 40rpx;
		width: 50rpx;
		height: 50rpx;
	}

	.icon-back-popup {
		margin-left: 10rpx;
		width: 50rpx;
		height: 50rpx;
	}

	.page-title {
		flex: 1;
		text-align: center;
		font-size: 35rpx;
		font-weight: 700;
	}

	.tab-scroll {
		width: 100%;
		white-space: nowrap;
		background-color: #fff;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.tabs {
		display: flex;
		height: 88rpx;
	}

	.tab-item {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 0 30rpx;
		font-size: 32rpx;
		font-weight: 600;
		position: relative;
	}

	.tab-item.active {
		color: #e02e24;
	}

	.tab-line {
		position: absolute;
		bottom: 0;
		left: 30rpx;
		right: 30rpx;
		height: 6rpx;
		background-color: #e02e24;
	}

	.swiper-content {
		flex: 1;
		margin-top: 260rpx;
		background-color: #f8f8f8;
	}

	/* 隐藏滚动条和灰色背景 */
	::-webkit-scrollbar {
		display: none;
	}

	.uni-swiper-wrapper {
		background-color: transparent !important;
	}

	.order-list {
		height: 100%;
		background-color: #fff;
	}

	.orders-section {
		padding: 5rpx;
	}

	.order-item {
		background-color: #f5f5f5;
		border-radius: 10rpx;
		margin: 10rpx;
		padding: 15rpx;
		box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.01);
	}

	.store-info {
		display: flex;
		align-items: center;
		border-bottom: 1rpx solid #f5f5f5;
		margin-bottom: 10rpx;
	}

	.product-image {
		border-radius: 8rpx;
		/* 圆角图片 */
	}

	.store-icon {
		width: 40rpx;
		height: 40rpx;
		margin-right: 10rpx;
	}

	.store-name {
		flex: 1;
		font-size: 30rpx;
		font-weight: 700;
	}

	.order-status {
		color: #e02e24;
		font-size: 30rpx;
	}


	.product-image {
		float: left;
		width: 180rpx;
		height: 180rpx;
		border-radius: 8rpx;
		margin-right: 10rpx;
	}

	.product-details {
		flex: 1;
	}

	.product-name {
		font-size: 28rpx;
		margin-bottom: 10rpx;
	}

	.product-desc {
		font-size: 24rpx;
		color: #eee;
		margin-bottom: 10rpx;
	}

	.price-info {
		margin-top: 10rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.quantity {
		color: #999;
		font-size: 26rpx;
	}

	.order-actions {
		display: flex;
		justify-content: flex-end;
		padding-top: 10rpx;
		gap: 15rpx;
	}

	.action-btn {
		font-size: 24rpx;
		padding: 8rpx 20rpx;
		border-radius: 30rpx;
		margin: 0;
		white-space: nowrap;
		line-height: normal;
		height: auto;
	}

	.action-btn.primary {
		background-color: #e02e24;
		color: #fff;
		border: none;
	}

	.action-btn.cancel {
		background-color: #fff;
		color: #666;
		border: 1rpx solid #ddd;
	}

	.empty-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 60rpx 0;
		color: #999;
		font-size: 28rpx;
	}

	.empty-icon {
		width: 160rpx;
		height: 160rpx;
		margin-bottom: 20rpx;
		opacity: 0.5;
	}

	.content {
		padding: 20rpx;
		background-color: #fff;
	}

	.tab-content {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
	}

	.content-item {
		width: 48%;
		margin-bottom: 20rpx;
	}

	.product-card {
		background: #ffffff;
		border-radius: 12rpx;
		overflow: hidden;
		box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.05);
	}

	.item-image {
		width: 100%;
		height: 340rpx;
		object-fit: cover;
	}

	.product-info {
		display: flex;
		flex-direction: column;
		margin-bottom: 10rpx;
	}

	.item-title {
		font-size: 28rpx;
		color: #000000;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
		overflow: hidden;
		margin-bottom: 10rpx;
	}

	.service-tags {
		display: flex;
		gap: 8rpx;
		margin-bottom: 10rpx;
	}

	.tag {
		font-size: 22rpx;
		padding: 2rpx 8rpx;
		border-radius: 4rpx;
		white-space: nowrap;
	}

	.pay-later {
		color: #00aa00;
		background: rgba(76, 175, 80, 0.1);
	}

	.quick-refund {
		color: #666;
		background: #f5f5f5;
	}

	.price-and-sales {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-top: 10rpx;
	}

	.price {
		font-size: 30rpx;
		color: #e02e24;
		font-weight: bold;
	}

	.sales {
		font-size: 24rpx;
		color: #999999;
	}

	.loading {
		text-align: center;
		padding: 40rpx;
		color: #999;
		font-size: 28rpx;
	}

	/* 修改swiper的默认样式 */
	.wx-swiper-wrapper {
		background-color: transparent !important;
	}

	.wx-swiper-dots {
		display: none !important;
	}

	/* 修改scroll-view的默认样式 */
	::-webkit-scrollbar {
		width: 0;
		height: 0;
		color: transparent;
	}

	.review-content {
		float: right;
		width: 70%;
		margin: 10rpx 0;
		background-color: #fff;
		padding: 6rpx;
		border-radius: 8rpx;
	}

	.review-label {
		font-size: 26rpx;
		color: #666;
		margin-right: 20rpx;
	}

	.review-text {
		font-size: 26rpx;
		color: #333;
	}
</style>