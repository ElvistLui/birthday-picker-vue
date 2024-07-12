# birthday-picker-vue

本间的公历/农历转换，依赖 js-calendar-converter.js

> 只支持vue3，使用最新的顶层语法编写
>
> ## uniapp 日期选择器
>
> - 可选择年、月、日
> - 可选择公历（阳历）农历（阴历）
> - 支持自定义弹窗标题、主题色等

## 示例图

![公历](https://gitee.com/elvist/image/raw/master/images/birthdaypicker1.png)

![农历](https://gitee.com/elvist/image/raw/master/images/birthdaypicker2.png)

![结果](https://gitee.com/elvist/image/raw/master/images/birthdaypicker3.png)

## 属性说明

|    属性     | 是否必填 |  值类型  |        默认值         |                      说明                      |
| :---------: | :------: | :------: | :-------------------: | :--------------------------------------------: |
|    title    |    否    |  String  |      生日选择器       |                   选择器标题                   |
| themeColor  |    否    |  String  |        #ff0000        |          主题色，支持rgab和16进制颜色          |
|  maskColor  |    否    |  String  | rgba(49, 47, 47, 0.4) |       遮罩层背景色，支持rgab和16进制颜色       |
|  startDate  |    否    |   Date   |       1901/1/1        |          选择开始年月日，不早于1901年          |
|   endDate   |    否    |   Date   |      new Date()       |          选择结束年月日，不晚于2100年          |
| defaultDate |    否    |   Date   |      new Date()       |                默认初始选中时间                |
|   isLunar   |    否    | Boolean  |         false         |               默认选中公历/农历                |
|   @submit   |    否    | Function |        Object         | 监听选择事件，回调函数会返回选中日期的完整信息 |

## 完整示例

```javascript
<template>
	<view class="page">
		<view class="form-item" @click="selectDate">
			<view class="form-title">生日</view>
			<input class="form-value" placeholder="请点击选择生日" v-model="date" :disabled="true" />
		</view>
		<view class="json">返回结果：</view>
		<view class="json">{{ JSON.stringify(dateObj) }}</view>
		<!-- 生日选择器 -->
		<datePicker ref="pickerRef" @submit="handleBirthdayDate" title="生日选择器" maskColor="rgba(0,0,0,0.3)"
			:defaultDate="defaultDate" :isLunar="defaultLunar" />
	</view>
</template>

<script>
	import datePicker from '/components/nj-birthday-picker/nj-birthday-picker.vue'

	export default {
		components: {
			datePicker,
		},
		data() {
			return {
				title: 'Hello',
				date: "",
				dateObj: {},
			}
		},
		onLoad() {

		},
		methods: {
			// 去选择生日
			selectDate() {
				this.$refs.pickerRef.show();
			},
			// 选择生日回调
			handleBirthdayDate(res) {
				this.dateObj = res;
				if (res.isLunar) {
					this.date = res.lYear + res.gzYear + "年" + res.IMonthCn + res.IDayCn;
				} else {
					this.date = res.date;
				}
			},
		}
	}
</script>

<style>
	.page {
		flex: 1;
		width: 100vw;
		padding: 20px;
	}

	.form-item {
		display: flex;
		align-items: center;
		height: 60px;
		flex-direction: row;
		border-bottom: 1px solid #eee;
	}

	.form-title {
		width: 60px;
		font-size: 18px;
		font-weight: normal;
		color: #333;
	}

	.form-value {
		flex: 1;
		font-size: 18px;
		font-weight: normal;
		color: #333;
	}
	
	.json {
		margin: 10px 10px;
	}
</style>
```



本插件基于 https://ext.dcloud.net.cn/plugin?id=16152 修改。