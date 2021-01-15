<template>
	<view class="slide-scroll" :id="scrollId" @touchstart="touchStart($event)" @touchend="touchEnd($event)" @touchmove.stop.prevent="touchMove($event)">
		<view class="slide-scroll-box" :style="{ transform: 'translate3d(0, ' + scrollY + 'px,  0)' }">
			<slot />
		</view>
	</view>
</template>

<script>
	let timely = null
	export default {
		name: 'poput-scroll',
		onShow() {
			// 获取可视区域高度和滑动区域高度
			this.touchStart()
		},
		computed: {
			maxScrollY() {
				return this.scrollHeight - this.boxHeight
			},
			isOut(){
				let maxScrollY = this.maxScrollY
				if(this.scrollY > 0 || Math.abs(this.scrollY) > this.maxScrollY){
					return true
				}
				return false
			}
		},
		data() {
			return {
				scrollId: "scroll-" + Math.random().toString().replace(".","").substr(0, 8),
				boxHeight: 0, // 可视区域高度
				boxTop: 0, // 
				boxBottom: 0, // 

				scrollHeight: 0, // 可滑动区域高度
				scrollTop: 0, // 
				scrollBottom: 0, // 

				scrollY: 0, // 滑动的位置
				startY: 0, // 手指开始时的位置
				moveTime: 0,
				status: "", // start、 move、end
			};
		},
		methods: {
			// 滑动开始
			touchStart(e) {
				this.status = "start"
				clearInterval(timely)
				const selector = uni.createSelectorQuery()
				//#ifdef MP-WEIXIN
					.in(this)
				//#endif
				selector.select("#" + this.scrollId).boundingClientRect(res => {
					if (res != null) {
						this.boxHeight = res.height
						this.boxBottom = res.bottom
						this.boxTop = res.top
					}
				})

				selector.select(`#${this.scrollId} .slide-scroll-box`).boundingClientRect(res => {
					if (res != null) {
						this.scrollHeight = res.height
						this.scrollBottom = res.bottom
						this.scrollTop = res.top
					}
				}).exec();
				// 记录手指所处位置
				if (e) {
					this.startY = e.touches[0].pageY;
				}

			},
			// 滑动中
			touchMove(e) {
				this.status = "move"
				this.compY = (e.touches[0].pageY - this.startY) * 1.1
				this.startY = e.touches[0].pageY
				// 内容区域滚动
				if(this.scrollY <= 0 && this.scrollY >= -this.maxScrollY){
					this.scrolling()
				} else {
					// 超过内容区域滚动
					this.scrolling(0.4)
				}
			},
			// 滑动结束
			touchEnd(e) {
				this.status = "end"
				clearInterval(timely)
				const endTime = e.timeStamp;
				let count = 1
				if (Math.abs(this.compY) > 4 && !this.isOut) {
					timely = setInterval(() => {
						// 继续滚动
						this.scrolling(count)
						count -= 0.02
						// 停止
						if (count <= 0) {
							clearInterval(timely)
						}
					}, 20)
				} else {
					this.homing()
				}
			},
			// 内容区域滑动
			scrolling(count = 1){
				if (this.compY > 0) {
					let scrollY = this.scrollY + this.compY * 1.1 * count
					if(this.status === "move"){
						this.scrollY = scrollY
					} else {
						this.scrollY = scrollY > 0 ? 0 : scrollY
					}
				} else if (this.compY < 0) {
					let scrollY = this.scrollY + this.compY * 1.1 * count
					if(this.status === "move"){
						this.scrollY = scrollY
					} else {
						this.scrollY = scrollY < -this.maxScrollY ? -this.maxScrollY : scrollY
					}
				} else {
					timely && clearInterval(timely)
				}
			},
			// 滑动超出返回
			homing(){
				if (this.scrollY > 0) {
					timely = setInterval(() => {
						let scrollY = this.scrollY -= 10
						if(scrollY > 0){
							this.scrollY = scrollY
						} else {
							this.scrollY = 0
							clearInterval(timely)
						}
					}, 15)
				} else if(this.scrollY < -this.maxScrollY) {
					timely = setInterval(() => {
						let scrollY = this.scrollY += 10
						if(scrollY < -this.maxScrollY){
							this.scrollY = scrollY
						} else {
							this.scrollY = -this.maxScrollY
							clearInterval(timely)
						}
					}, 15)
				} 
			}
		}
	};
</script>

<style lang="scss" scoped>
	.slide-scroll {
		height: 100%;
		max-height: 100%;
		width: 100%;
		overflow: hidden;
	}
</style>
