<template>
	<!-- 城市选择-->
	<view class="city-select">
		<view class="search">
			<input type="text" @input="keyInput" class="s-input" placeholder="搜索" />
		</view>
		<scroll-view :scroll-top="scrollTop" scroll-y="true" class="city-select-main" id="city-select-main" :scroll-into-view="toView">
			<!-- 预留搜索-->
			<view class="city-serach" v-if="isSearch"></view>
			<!-- 当前定位城市 -->
			<view class="hot-title" v-if="activeCity && !serachCity">当前定位城市</view>
			<view class="hot-city1" v-if="activeCity && !serachCity">
				<view class="hot-item" @click="cityTrigger(activeCity)">{{ activeCity[formatName] }}</view>
			</view>
			<!-- 热门城市 -->
			<view class="hot-title" v-if="hotCity.length > 0 && !serachCity">热门品牌</view>
			<view class="hot-city1" style="height: ;" v-if="hotCity.length > 0 && hotCity.length < 4 && !serachCity">
				<template v-for="(item, index) in hotCity">
					<view :key="index" @click="cityTrigger(item, 'hot')" class="hot-item">{{ item[formatName] }}</view>
				</template>
			</view>
			<view class="hot-city2" style="height: ;" v-if="hotCity.length > 4 && hotCity.length < 8 && !serachCity">
				<template v-for="(item, index) in hotCity">
					<view :key="index" @click="cityTrigger(item, 'hot')" class="hot-item">{{ item[formatName] }}</view>
				</template>
			</view>
			<view class="hot-city" style="height: ;" v-if="hotCity.length > 7  && !serachCity">
				<template v-for="(item, index) in hotCity">
					<view :key="index" @click="cityTrigger(item, 'hot')" class="hot-item">{{ item[formatName] }}</view>
				</template>
			</view>
			<!-- 城市列表(搜索前) -->
			<view class="citys" v-if="!serachCity">
				<view v-for="(city, index) in sortItems" :key="index" v-show="city.isCity">
					<view class="citys-item-letter" :id="'city-letter-' + (city.name === '#' ? '0' : city.name)">{{ city.name }}</view>
					<view class="citys-item" v-for="(item, inx) in city.citys" :key="inx" @click="cityTrigger(item)">{{ item.cityName }}</view>
				</view>
			</view>
			<!-- 城市列表(搜索后)  -->
			<view class="citys" v-if="serachCity">
				<view v-for="(item, index) in searchDatas" :key="index">
					<view class="citys-item" :key="inx" @click="cityTrigger(item)">{{ item.name }}</view>
				</view>
			</view>
		</scroll-view>
		<!-- 城市选择索引-->
		<view class="city-indexs-view" v-if="!serachCity">
			<view class="city-indexs">
				<view v-for="(cityIns, index) in handleCity" v-show="cityIns.isCity" :key="index" @click="cityindex(cityIns.forName)">{{ cityIns.name }}</view>
			</view>
		</view>
	</view>
</template>

<script>
import citySelect from './citySelect.js'
export default {
	props: {
		//传入要排序的名称
		formatName: {
			type: String,
			default: 'cityName'
		},
		//当前定位城市
		activeCity: {
			type: Object,
			default: () => null
		},
		//热门城市
		hotCity: {
			type: Array,
			default: () => []
		},
		//城市数据
		obtainCitys: {
			type: Array,
			default: () => []
		},
		//是否有搜索
		isSearch: {
			type: Boolean,
			default: true
		}
	},
	data() {
		return {
			toView: 'city-letter-Find', //锚链接 初始值
			scrollTop: 0, //scroll-view 滑动的距离
			cityindexs: [], // 城市索引
			activeCityIndex: '', // 当前所在的城市索引
			handleCity: [], // 处理后的城市数据
			serachCity: '', // 搜索的城市
			cityData: []
		}
	},
	computed: {
		/**
		 * @desc 城市列表排序
		 * @return  Array
		 */
		sortItems() {
			for (let index = 0; index < this.handleCity.length; index++) {
				if (this.handleCity[index].isCity) {
					let cityArr = this.handleCity[index].citys
					cityArr = cityArr.sort(function(a, b) {
						var value1 = a.unicode
						var value2 = b.unicode
						return value1 - value2
					})
				}
			}
			return this.handleCity
		},
		/**
		 * @desc 搜索后的城市列表
		 * @return Array
		 */
		searchDatas() {
			var searchData = []
			for (let i = 0; i < this.cityData.length; i++) {
				if (this.cityData[i][this.formatName].indexOf(this.serachCity) !== -1) {
					searchData.push({
						oldData: this.cityData[i],
						name: this.cityData[i][this.formatName]
					})
				}
			}
			return searchData
		}
	},
	created() {
		// 初始化城市数据
		this.cityData = this.obtainCitys
		this.initializationCity()
		this.buildCityindexs()
	},
	watch: {
		obtainCitys(newData) {
			this.updateCitys(newData)
		}
	},
	methods: {
		/**
		 * @desc 初始化
		 */
		updateCitys(data) {
			if (data && data.length) {
				this.cityData = data
				this.initializationCity()
				this.buildCityindexs()
			}
		},
		/**
		 * @desc 监听输入框的值
		 */
		keyInput(event) {
			
			this.serachCity = event.detail.value;
			this.earchDatas();
			console.log(this.serachCity)
		},
		/**
		 * @desc 初始化城市数据
		 * @return undefind
		 */
		initializationCity() {
			this.handleCity = []
			const cityLetterArr = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '#']
			for (let index = 0; index < cityLetterArr.length; index++) {
				this.handleCity.push({
					name: cityLetterArr[index],
					isCity: false, // 用于区分是否含有当前字母开头的城市
					citys: [], // 存放城市首字母含是此字母的数组
					forName: 'city-letter-' + cityLetterArr[index] //label的绑定
				})
			}
		},
		/**
		 * @desc 得到城市的首字母
		 * @param str String
		 */
		getLetter(str) {
			return citySelect.getFirstLetter(str[0])
		},
		/**
		 * @desc 构建城市索引
		 * @return undefind
		 */
		buildCityindexs() {
			this.cityindexs = []
			for (let i = 0; i < this.cityData.length; i++) {
				// 获取首字母
				const cityLetter = this.getLetter(this.cityData[i][this.formatName]).firstletter
				// 获取当前城市首字母的unicode，用作后续排序
				const unicode = this.getLetter(this.cityData[i][this.formatName]).unicode

				const index = this.cityIndexPosition(cityLetter)
				if (this.cityindexs.indexOf(cityLetter) === -1) {
					this.handleCity[index].isCity = true
					this.cityindexs.push(cityLetter)
				}

				this.handleCity[index].citys.push({
					cityName: this.cityData[i][this.formatName],
					unicode: unicode,
					oldData: this.cityData[i]
				})
			}
		},
		/**
		 * @desc 滑动到城市索引所在的地方
		 * @param id String 城市索引
		 */
		cityindex(id) {
			this.toView = id
			// //创建节点查询器
			// const query = uni.createSelectorQuery().in(this)
			// var that = this
			// that.scrollTop = 0
			// //滑动到指定位置(解决方法:重置到顶部，重新计算，影响:页面会闪一下)
			// setTimeout(() => {
			// 	query
			// 		.select('#city-letter-' + (id === '#' ? '0' : id))
			// 		.boundingClientRect(data => {
			// 			// console.log("得到布局位置信息" + JSON.stringify(data));
			// 			// console.log("节点离页面顶部的距离为" + data.top);
			// 			data ? (that.scrollTop = data.top) : void 0
			// 		})
			// 		.exec()
			// }, 0)
		},
		/**
		 * @desc 获取城市首字母的unicode
		 * @param letter String 城市索引
		 */
		cityIndexPosition(letter) {
			if (!letter) {
				return ''
			}
			const ACode = 65
			return letter === '#' ? 26 : letter.charCodeAt(0) - ACode
		},
		/** @desc 城市列表点击事件
		 *  @param Object
		 */
		cityTrigger(item) {
			// 传值到父组件
			this.$emit('cityClick', item.oldData ? item.oldData : item)
		}
	}
}
</script>

<style lang="scss">
//宽度转换vw
@function vww($number) {
	@return ($number / 375) * 750 + rpx;
}

view {
	box-sizing: border-box;
}
.search {
		width: 100%;
		height: 120upx;
		display: flex;
		justify-content: center;
		align-items: center;
}

.s-input {
	width: 700upx;
	height: 80upx;
	background: url('data:image/png;base64,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') 20upx center no-repeat #fff;
	background-size: 40upx;
	text-indent: 80upx;
}
.city-serach {
	width: 80%;
	color: #4a4a4a;
	padding: 0 vww(0);

	&-input {
		margin: vww(10) 0;
		height: vww(40);
		line-height: vww(40);
		font-size: vww(14);
		padding: 10rpx vww(5);
		border: 1px solid #4d8cfd;
		border-radius: 3px;
	}
}

.city-select-main {
	position: relative;
	// overflow: scroll;
	// -webkit-overflow-scrolling: touch;
	width: 100%;
	height: 100%;
	background: #f6f5fa;
	// overflow-y: auto;
}

.city-select {
	position: relative;
	width: 100vw;
	height: 100vh;
	background: #f6f5fa;

	// 热门城市
	.hot-title {
		padding-left: vww(23);
		width: 100vw;
		font-size: 14px;
		line-height: vww(30);
		color: #9b9b9b;
	}
	.hot-city1 {
		padding-left: 20rpx;
		padding-right: 20rpx;
		// overflow: hidden;
		width: 100%;
		height: 85rpx;
		.hot-item {
			float: left;
			padding: 5rpx 20rpx 50rpx 20rpx;
			margin-right: 16rpx;
			margin-bottom: 20rpx;
			overflow: hidden;
			width: auto;
			height: vww(30);
			font-size: 14px;
			text-align: center;
	
			display: -webkit-box;
			-webkit-box-orient: vertical;
			-webkit-line-clamp: 1;
	
			line-height: vww(31);
			color: #4a4a4a;
			background: #fff;
			border: 1px solid #ebebf0;
	
			// &:nth-child(3n) {
			// 	margin-right: 0;
			// }
		}
	
		.hot-hidden {
			display: none;
			margin-right: 0;
		}
	}
	.hot-city2 {
		padding-left: 10rpx;
		padding-right: 10rpx;
		// overflow: hidden;
		width: 100%;
		height: 150rpx;
		.hot-item {
			float: left;
			padding: 5rpx 20rpx 50rpx 20rpx;
			margin-right: 16rpx;
			margin-bottom: 20rpx;
			overflow: hidden;
			width: auto;
			height: vww(30);
			font-size: 14px;
			text-align: center;
	
			display: -webkit-box;
			-webkit-box-orient: vertical;
			-webkit-line-clamp: 1;
	
			line-height: vww(31);
			color: #4a4a4a;
			background: #fff;
			border: 1px solid #ebebf0;
	
			// &:nth-child(3n) {
			// 	margin-right: 0;
			// }
		}
	
		.hot-hidden {
			display: none;
			margin-right: 0;
		}
	}
	.hot-city {
		padding-left: 10rpx;
		padding-right: 10rpx;
		// overflow: hidden;
		width: 100%;
		height: 220rpx;
		.hot-item {
			float: left;
			padding: 5rpx 20rpx 50rpx 20rpx;
			margin-right: 16rpx;
			margin-bottom: 20rpx;
			overflow: hidden;
			width: auto;
			height: vww(30);
			font-size: 14px;
			text-align: center;

			display: -webkit-box;
			-webkit-box-orient: vertical;
			-webkit-line-clamp: 1;

			line-height: vww(31);
			color: #4a4a4a;
			background: #fff;
			border: 1px solid #ebebf0;

			// &:nth-child(3n) {
			// 	margin-right: 0;
			// }
		}

		.hot-hidden {
			display: none;
			margin-right: 0;
		}
	}

	.citys {
		> view {
			padding-left: vww(18);
			margin-top: vww(10);
			width: 100%;
			font-size: 14px;
			background: #fff;

			.citys-item-letter {
				margin-left: vww(-18);
				padding-left: vww(18);
				margin-top: 5rpx;
				width: 100vw;
				line-height: vww(20);
				color: #9b9b9b;
				background: #f6f5fa;
				border-top: none;
			}

			.citys-item {
				width: 100%;
				line-height: vww(50);
				color: #4a4a4a;
				border-bottom: 1px solid #ebebf0;

				&:last-child {
					border: none;
				}
			}
		}
	}

	.city-indexs-view {
		position: absolute;
		right: 0;
		top: 0;
		z-index: 999;
		display: flex;
		width: vww(20);
		height: 100%;
		text-align: center;

		.city-indexs {
			width: vww(20);
			text-align: center;
			vertical-align: middle;
			align-self: center;

			> view {
				margin-bottom: vww(0.5);
				width: vww(20);
				font-size: 12px;
				color: #4d8cfd;

				&:last-child {
					margin-bottom: 0;
				}
			}
		}
	}
}
</style>
