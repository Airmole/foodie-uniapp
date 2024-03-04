<template>
	<scroll-view scroll-y class="bg-gradual-blue" :style="{height: `${screenHeight}px`}">
		<cu-custom bgColor="bg-gradual-blue" :isBack="false">
			<block slot="content">Airmole | 干饭日记</block>
		</cu-custom>
		
		<attend-calendar @on-click="checkDate" @on-monthchanged="monthChanged" :lateddates="eatedDates"></attend-calendar>
		
		<view class="cu-list menu-avatar cu-card" v-if="dataList.length > 0">
		    <view class="cu-item" v-for="(item,index) in dataList" :key="index" @click="clickItem(item)">
		        <view class="cu-avatar lg">
					<image @click.native.stop="clickImage(index)" class="cu-avatar lg" :src="item.thumb ? item.thumb : (item.image+'?imageMogr2/auto-orient/strip%7CimageView2/2/w/360')" mode="scaleToFill" :show-menu-by-longpress="true"></image>
				</view>
		        <view class="content">
		            <view class="text-grey text-sm flex">
		                <view class="">
		                    <text>{{item.desc}}</text>
		                </view>
		            </view>
		        </view>
		    </view>
		</view>
		<view class="padding bg-white margin-lr-sm radius text-center" v-else>
			<image class="xl bg-white cu-avatar" src="@/static/nothing.png" mode="widthFix"></image>
			<view class="margin-sm"><text>这一天不知道吃了什么东西~</text></view>
		</view>
		
		<view class="foot text-center margin-top-xl">
			<image class="cu-avatar round" src="@/static/avatar.png" mode="widthFix"></image>
			<view class="">Airmole</view>
		</view>
		
	</scroll-view>
</template>

<script>
	export default {
		data() {
			return {
				screenHeight: 600,
				dataList: [],
				eatedDates: []
			}
		},
		onLoad(options) {
			const sysInfo = uni.getSystemInfoSync()
			this.screenHeight = sysInfo.screenHeight
			this.fetchFood()
			this.fetchMonth()
			console.log('options', options)
		},
		methods: {
			checkDate (e) {
				const date = e.split('-')
				const month = date[1].padStart(2, '0')
				const day = date[2].padStart(2, '0')
				const dateString = `${date[0]}${month}${day}`
				this.fetchFood(dateString)
			},
			clickImage (index) {
				const urls = []
				for (let item of this.dataList) {
					urls.push(item.image)
				}
				wx.previewImage({
				  current: this.dataList[index].image,
				  urls: urls
				})
			},
			clickItem (item) {
				window.location.href = item.url
			},
			monthChanged (month) {
				const monthValue = month.replace('-', '')
				this.fetchMonth(monthValue)
			},
			fetchMonth(month = '') {
				let url = 'https://workers-api.airmole.net/api/foodie/month'
				if (month !== '') url = `${url}?month=${month}`
				uni.request({
					url: url,
					success: (res) => {
						const eatedDates = []
						for (let item of res.data) {
							const month = Number(item.month) < 10 ? item.month.replace('0', '') : item.month
							const day = Number(item.date) < 10 ? item.date.replace('0', '') : item.date
							eatedDates.push(`${item.year}-${month}-${day}`)
						}
						this.eatedDates = eatedDates
					}
				})
			},
			fetchFood(date = '') {
				let url = 'https://workers-api.airmole.net/api/foodie'
				if (date !== '') url = `${url}?date=${date}`
				uni.request({
					url: url,
					success: (res) => {
						this.dataList = res.data
					}
				})
			}
		}
	}
</script>

<style>
</style>