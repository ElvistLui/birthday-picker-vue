<template>
	<view class="birthday-picker" v-if="md.done">
		<!-- 半透明遮罩 -->
		<view class="mask" :class="{ show: md.open }" :style="{ backgroundColor: props.maskColor }"
			@touchmove.stop.prevent catchtouchmove="true" @click="md.open = false">
		</view>
		<!-- 容器 -->
		<view class="wrap" :class="{ show: md.open }">
			<!-- 头部视图 -->
			<view class="picker-header" @touchmove.stop.prevent catchtouchmove="true">
				<!-- 公历/农历切换 -->
				<view class="change" :style="{ border: '1px solid ' + props.themeColor }">
					<view :class="!md.isLunar ? 'active' : ''"
						:style="!md.isLunar ? { backgroundColor: props.themeColor } : {}" @click="_onLunar(false)">公历
					</view>
					<view :class="md.isLunar ? 'active' : ''"
						:style="md.isLunar ? { backgroundColor: props.themeColor } : {}" @click="_onLunar(true)">农历
					</view>
				</view>
				<!-- 标题 -->
				<text>{{ props.title }}</text>
				<!-- 确定按钮 -->
				<view class="submit">
					<view />
					<text :style="{ color: props.themeColor }" @click="_onSubmit()">确 定</text>
				</view>
			</view>
			<!-- 公历选择器 -->
			<view class="picker-body" v-if="!md.isLunar">
				<picker-view :value="md.value" @change="_onChange" indicator-class="select-line">
					<!-- 年 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.cYears" :key="item" :class="
                md.selYear.num == item.num ? 'select-item' : 'column-item'
              ">
							{{ item.num }}年
						</view>
					</picker-view-column>
					<!-- 月 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.selYear.months" :key="item" :class="
                md.selMonth.num == item.num ? 'select-item' : 'column-item'
              ">
							{{ zeroPadding(item.num) }}月
						</view>
					</picker-view-column>
					<!-- 日 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.selMonth.days" :key="item" :class="
                md.selDay == item ? 'select-item' : 'column-item'
              ">
							{{ zeroPadding(item) }}日
						</view>
					</picker-view-column>
				</picker-view>

			</view>
			<!-- 农历选择器 -->
			<view class="picker-body" v-if="md.isLunar">
				<picker-view :value="md.value" @change="_onChange" indicator-class="select-line">
					<!-- 年 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.lYears" :key="item" :class="
			  md.selYear.num == item.num ? 'select-item' : 'column-item'
			">
							{{ item.num + item.name }}年
						</view>
					</picker-view-column>
					<!-- 月 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.selYear.months" :key="item" :class="
			  md.selMonth.num == item.num ? 'select-item' : 'column-item'
			">
							{{ zeroPadding(item.name) }}
						</view>
					</picker-view-column>
					<!-- 日 -->
					<picker-view-column class="picker-view-column">
						<view v-for="(item, index) in md.selMonth.days" :key="item" :class="
			  md.selDay == item ? 'select-item' : 'column-item'
			">
							{{ zeroPadding(md.selMonth.names[index]) }}
						</view>
					</picker-view-column>
				</picker-view>
			</view>
		</view>
	</view>
</template>

<script setup>
	import {
		ref,
		reactive,
		onMounted,
		watch
	} from "vue";
	import {
		onReady
	} from "@dcloudio/uni-app";
	import calendar from 'js-calendar-converter';

	const props = defineProps({
		// 选择器标题
		title: {
			type: String,
			default: "出生时间",
		},
		// 主题色
		themeColor: {
			type: String,
			default: "#FA5132",
		},
		// 起始时间（公历，不早于1901年）
		startDate: {
			type: Date,
			default: new Date(1901, 0, 1),
		},
		// 结束时间（公历，不晚于2100年）
		endDate: {
			type: Date,
			default: new Date(),
		},
		// 默认选中时间（公历）
		defaultDate: {
			type: Date,
			default: new Date(),
		},
		// 蒙版颜色
		maskColor: {
			type: String,
			default: "rgba(49, 47, 47, 0.4)",
		},
		// 是否是农历，默认公历
		isLunar: {
			type: Boolean,
			default: false,
		},
	});
	const emit = defineEmits(["submit"]);
	defineExpose({
		show,
		hide,
	});

	const md = reactive({
		// 是否显示
		open: false,
		done: false,
		// 是否为农历
		isLunar: false,
		// 默认选中日期
		defaultDate: new Date(),
		// 选择器选中的索引
		value: [0, 0, 0],

		// 公历年数据[{"num":1990,"months":[{"num":1,days:[1,2]}]}]
		cYears: [],
		// 农历年数据[{"num":1990,"name":"庚子",months":[{"num":"1","name":"正月","isLeap":false,days:[1,2],names:["初一","初二"]}]}]
		lYears: [],

		// 选中的年{"num":1990,"months":[...]}
		selYear: {},
		// 选中的月{"num":1,"days":[...]}
		selMonth: {},
		// 选中的日num
		selDay: "",
	});

	onMounted(() => {
		init();
	});

	// 初始化默认值
	function init() {
		if (md.cYears.length == 0 && md.lYears.length == 0) {
			// 首次打开才赋值
			md.isLunar = props.isLunar;
			md.defaultDate = props.defaultDate;
		}
		initYears();
		setSelectValue();
		md.done = true;
	}

	// 初始化年份
	function initYears() {
		// 设置的开始年份、结束年份
		var startYear, endYear;
		// 选择器开始的年、月、日
		var sy, sm, sd;
		// 选择器结束的年、月、日
		var ey, em, ed;
		// 默认选中的年、月、日
		var dy, dm, dd;

		// 下面可以优化为，选中年时，先获取月列表，有则展示，无责创建；日列表同理
		if (md.isLunar) {
			// 农历年
			var defaultObj = calendar.solar2lunar(md.defaultDate.getFullYear(), md.defaultDate.getMonth() + 1,
				md
				.defaultDate.getDate());
			dy = defaultObj.lYear;
			dm = defaultObj.lMonth;
			dd = defaultObj.lDay;
			var dl = defaultObj.isLeap;

			if (md.lYears.length > 0) {
				// 已经创建好了，只需要找到已选中日期的索引
				var stop = false;
				for (var i = 0; i < md.lYears.length; i++) {
					var year = md.lYears[i];
					if (year.num === dy) {
						for (var j = 0; j < year.months.length; j++) {
							var month = year.months[j];
							if (month.num === dm &&
								month.isLeap == dl) {
								for (var k = 0; k < month.days.length; k++) {
									if (month.days[k] === dd) {
										md.value[0] = i;
										md.value[1] = j;
										md.value[2] = k;
										stop = true;
										break;
									}
								}
							}
						}
						if (stop) {
							break;
						}
					}
					if (stop) {
						break;
					}
				}
				return;
			}
			// 开始日期，转农历
			var startObj = calendar.solar2lunar(props.startDate.getFullYear(), props.startDate.getMonth() + 1, props
				.startDate.getDate());
			sy = startObj.lYear;
			sm = startObj.lMonth;
			sd = startObj.lDay;
			var sl = startObj.isLeap;

			// 结束日期，转农历
			var endObj = calendar.solar2lunar(props.endDate.getFullYear(), props.endDate.getMonth() + 1, props
				.endDate.getDate());
			ey = endObj.lYear;
			em = endObj.lMonth;
			ed = endObj.lDay;
			var el = endObj.isLeap;

			startYear = Math.max(sy, 1900);
			endYear = Math.min(ey, 2099);

			const years = [];
			for (var i = startYear; i <= endYear; i++) {
				if (i === dy) {
					// 默认选中年
					md.value[0] = i - startYear;
				}
				// 获取本年的闰月，0为没有闰月
				var leapMonth = calendar.leapMonth(i);
				const months = [];
				for (var j = 1; j <= 12; j++) {
					if (i === dy &&
						j === dm) {
						// 默认选中月（本年无闰月||月份小于闰月||（闰月数字相同&&默认选中不是闰月））
						md.value[1] = j - (leapMonth <= 0 || j < leapMonth || (j === leapMonth && !dl) ? 1 : 0);
					}
					var k = 1;
					var dayCount = calendar.monthDays(i, j);
					if (i === startYear) {
						// 不能早于最小日期
						if (j < sm) {
							continue;
						} else if (j === sm) {
							k = sd;
						}
					}
					if (i === endYear) {
						// 不能晚于最大日期
						if (j > em) {
							break;
						} else if (j === em) {
							dayCount = ed;
						}
					}
					const days = [];
					const names = [];
					for (k; k <= dayCount; k++) {
						if (i === dy &&
							j === dm &&
							k === dd &&
							!dl) {
							// 默认选中日（非闰月）
							md.value[2] = k - 1;
						}
						days.push(k);
						names.push(calendar.toChinaDay(k));
					}
					months.push({
						num: j,
						name: calendar.toChinaMonth(j),
						days: days,
						names: names,
						isLeap: false,
					});
					if (j === leapMonth) {
						// 是闰月，再添加一次
						var k = 1;
						var dayCount = calendar.leapDays(i);
						const days = [];
						const names = [];
						for (k; k <= dayCount; k++) {
							if (i === dy &&
								j === dm &&
								k === dd &&
								dl) {
								// 默认选中日（闰月）
								md.value[2] = k - 1;
							}
							days.push(k);
							names.push(calendar.toChinaDay(k));
						}
						months.push({
							num: j,
							name: "闰" + calendar.toChinaMonth(j),
							days: days,
							names: names,
							isLeap: true,
						});
					}
				}
				years.push({
					num: i,
					name: calendar.toGanZhiYear(i),
					months: months,
					lunar: true, // 标识是农历数据
				});
			}
			md.lYears = years;
		} else {
			// 公历年
			dy = md.defaultDate.getFullYear();
			dm = md.defaultDate.getMonth() + 1;
			dd = md.defaultDate.getDate();

			if (md.cYears.length > 0) {
				// 已经创建好了，只需要找到已选中日期的索引
				var stop = false;
				for (var i = 0; i < md.cYears.length; i++) {
					var year = md.cYears[i];
					if (year.num === dy) {
						for (var j = 0; j < year.months.length; j++) {
							var month = year.months[j];
							if (month.num === dm) {
								for (var k = 0; k < month.days.length; k++) {
									if (month.days[k] === dd) {
										md.value[0] = i;
										md.value[1] = j;
										md.value[2] = k;
										stop = true;
										break;
									}
								}
							}
						}
						if (stop) {
							break;
						}
					}
					if (stop) {
						break;
					}
				}
				return;
			}

			sy = props.startDate.getFullYear();
			sm = props.startDate.getMonth() + 1;
			sd = props.startDate.getDate();

			ey = props.endDate.getFullYear();
			em = props.endDate.getMonth() + 1;
			ed = props.endDate.getDate();

			startYear = Math.max(sy, 1900);
			endYear = Math.min(ey, 2099);

			const years = [];
			for (var i = startYear; i <= endYear; i++) {
				if (i === dy) {
					// 默认选中年
					md.value[0] = i - startYear;
				}
				// 是闰年
				var isLeap = (i % 4) === 0;
				const months = [];
				for (var j = 1; j <= 12; j++) {
					if (i === dy &&
						j === dm) {
						// 默认选中月
						md.value[1] = j - 1;
					}
					var k = 1;
					var dayCount = j == 2 && isLeap ? 29 : calendar.solarDays(i, j);
					if (i === startYear) {
						// 不能早于最小日期
						if (j < sm) {
							continue;
						} else if (j === sm) {
							k = sd;
						}
					}
					if (i === endYear) {
						// 不能晚于最大日期
						var endMonth = em;
						if (j > endMonth) {
							break;
						} else if (j === endMonth) {
							dayCount = ed;
						}
					}
					const days = [];
					for (k; k <= dayCount; k++) {
						if (i === dy &&
							j === dm &&
							k === dd) {
							// 默认选中日
							md.value[2] = k - 1;
						}
						days.push(k);
					}
					months.push({
						num: j,
						days: days,
					});
				}
				years.push({
					num: i,
					months: months,
				});
			}
			md.cYears = years;
		}
	}

	// 显示选择器
	function show() {
		init();
		md.open = true;
	}

	// 隐藏选择器
	function hide() {
		md.open = false;
	}

	// 切换公历/农历
	function _onLunar(e) {
		if (md.isLunar !== e) {
			md.isLunar = e;
			if (md.selYear) {
				// 将已选中的日期，转换成切换后的日历格式
				var res;
				if (md.isLunar) {
					// 切换成农历，那就将选中的公历转农历
					res = calendar.solar2lunar(md.selYear.num, md.selMonth.num, md.selDay);
				} else {
					// 切换成公历，那就将选中的农历转公历
					res = calendar.lunar2solar(md.selYear.num, md.selMonth.num, md.selDay, md.selMonth.isLeap);
				}
				md.defaultDate = new Date(res.cYear, res.cMonth - 1, res.cDay);
			}
			init();
		}
	}

	// 滚轮变化
	function _onChange(e) {
		md.value = e.detail.value;
		setSelectValue();
	}

	// 修改选中值
	function setSelectValue() {
		const v = md.value;

		md.selYear = md.isLunar ? md.lYears[Math.min(v[0], md.lYears.length - 1)] : md.cYears[Math.min(v[0], md.cYears
			.length - 1)]
		md.selMonth = md.selYear.months[Math.min(v[1], md.selYear.months.length - 1)];
		md.selDay = md.selMonth.days[Math.min(v[2], md.selMonth.days.length - 1)];
	}

	// 点击确定
	function _onSubmit() {
		var res;
		if (md.isLunar) {
			res = calendar.lunar2solar(md.selYear.num, md.selMonth.num, md.selDay, md.selMonth.isLeap);
		} else {
			res = calendar.solar2lunar(md.selYear.num, md.selMonth.num, md.selDay);
		}
		res.isLunar = md.isLunar;

		emit("submit", res);
		hide();
	}

	// 公历显示补0
	function zeroPadding(num) {
		if (md.isLunar) {
			return num;
		}
		return num.toString().padStart(2, "0");
	}
</script>

<style scoped lang="scss">
	@import "./nj-birthday-picker.scss";
</style>