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