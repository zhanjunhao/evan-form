<template>
	<view class="evan-form-show">
		<evan-form :hide-required-asterisk="hideRequiredAsterisk" ref="form" :model="info" :rules="rules">
			<evan-form-item label="邮箱：" prop="email" v-if="showFlag">
				<input class="form-input" placeholder-class="form-input-placeholder" v-model="info.email" placeholder="请输入邮箱" />
			</evan-form-item>
			<evan-form-item label="姓名：" prop="name">
				<input class="form-input" placeholder-class="form-input-placeholder" v-model="info.name" placeholder="请输入姓名" />
			</evan-form-item>
			<evan-form-item label="简介：" prop="desc">
				<textarea class="form-input textarea" placeholder-class="form-input-placeholder" v-model="info.desc" placeholder="请输入简介(10-30个字)" />
			</evan-form-item>
			<evan-form-item label="自定义验证(手机号)：" prop="phone">
				<input class="form-input" placeholder-class="form-input-placeholder" v-model="info.phone" placeholder="请输入手机号" />
			</evan-form-item>
			<evan-form-item label="自定义label颜色：" :label-style="{color:'blue'}"></evan-form-item>
			<evan-form-item label="自定义宽度默认为auto：" :label-style="{width:'190rpx','line-height':'50rpx'}"></evan-form-item>
			<evan-form-item label="不显示底部border：" :border="false"></evan-form-item>
			<evan-form-item prop="sex">
				<template slot="formItem">
					<view class="customize-form-item">
						<view class="customize-form-item__label">完全自定义内容（性别）：</view>
						<radio-group @change="sexChange">
							<label class="customize-form-item__radio" v-for="item in sexes" :key="item.value">
								<view>
									<radio :value="item.value" :checked="item.value === info.sex" />
								</view>
								<view class="customize-form-item__radio__text">{{item.name}}</view>
							</label>
						</radio-group>
					</view>
				</template>
			</evan-form-item>
		</evan-form>
		<button @click="save" class="evan-form-show__button">保存</button>
		<button @click="utilsSave" class="evan-form-show__button">直接调用utils验证</button>
		<button @click="validateSingle" class="evan-form-show__button">只验证邮箱</button>
		<button @click="validateMultiple" class="evan-form-show__button">只验证邮箱和手机号</button>
		<button @click="hideReqired" class="evan-form-show__button">{{hideRequiredAsterisk?'显示':'隐藏'}}*号</button>
	</view>
</template>

<script>
	import EvanForm from '@/components/evan-form/evan-form.vue'
	import EvanFormItem from '@/components/evan-form/evan-form-item.vue'
	import utils from '@/components/evan-form/utils.js'
	export default {
		components: {
			EvanForm,
			EvanFormItem
		},
		data() {
			return {
				hideRequiredAsterisk: false,
				sexes: [{
						name: '男',
						value: 'man'
					},
					{
						name: '女',
						value: 'woman'
					}
				],
				// 表单的内容必须初始化
				info: {
					name: '',
					email: '',
					desc: '',
					phone: '',
					sex: '',
				},
				rules: {
					name: {
						required: true,
						message: '请输入姓名'
					},
					email: [{
						required: true,
						message: '请输入邮箱'
					}, {
						type: 'email',
						message: '邮箱格式不正确'
					}],
					desc: [{
							required: true,
							message: '请输入简介'
						},
						{
							min: 10,
							max: 30,
							message: '简介必须在10到30个字之间'
						}
					],
					phone: [{
							required: true,
							message: '请输入手机号'
						},
						{
							validator: (rule, value, callback) => {
								// 注意这里如果用的是methods里的isMobilePhone将不生效
								if (this.$utils.isMobilePhone(value)) {
									callback()
								} else {
									callback(new Error('手机号格式不正确'))
								}
							}
						},
						// 或者这样也是可以的
						// {
						// 	validator: this.isMobile
						// }
					],
					sex: {
						required: true,
						message: '请选择性别'
					}
				},
				showFlag: true,
			}
		},
		mounted() {
			// 测试修改校验
			setTimeout(() => {
				this.showFlag = false;
				this.rules.name.required = false;
			},1000);
		},
		methods: {
			save() {
				this.$refs.form.validate((res) => {
					if (res) {
						uni.showToast({
							title: '验证通过'
						})
					}
				})
			},
			utilsSave() {
				utils.validate(this.info, this.$refs.form.getRules(this.rules), (res, errors) => {
					if (res) {
						uni.showToast({
							title: '验证通过'
						})
					}
				})
			},
			validateSingle() {
				this.$refs.form.validateField('email', (res) => {
					if (res) {
						uni.showToast({
							title: '验证通过'
						})
					}
				})
			},
			validateMultiple() {
				this.$refs.form.validateField(['email', 'phone'], (res) => {
					if (res) {
						uni.showToast({
							title: '验证通过'
						})
					}
				})
			},
			hideReqired() {
				this.hideRequiredAsterisk = !this.hideRequiredAsterisk
			},
			isMobilePhone() {
				const reg = /^1\d{10}$/
				if (reg.test(value)) {
					return true
				}
				return false
			},
			isMobile(rule, value, callback) {
				if (this.$utils.isMobilePhone(value)) {
					callback()
				} else {
					callback(new Error('手机号格式不正确'))
				}
			},
			sexChange(e) {
				this.info.sex = e.detail.value
			}
		}
	}
</script>

<style lang="scss">
	.evan-form-show {
		padding: 0 30rpx;
		background-color: #fff;

		.form-input {
			font-size: 28rpx;
			color: #333;
			text-align: right;
			width: 100%;
			box-sizing: border-box;
			height: 60rpx;
			&.textarea{
				height: 240rpx;
				padding: 24rpx 0;
			}
		}

		.form-input-placeholder {
			font-size: 28rpx;
			color: #999
		}

		&__button {
			width: 100%;
			height: 88rpx;
			border-radius: 8rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			padding: 0;
			font-size: 36rpx;
			color: #fff;
			margin-top: 20rpx;
			background-color: #2D87D5;

			&::before,
			&::after {
				border: none;
			}
		}

		.customize-form-item {
			&__label {
				font-size: 28rpx;
				color: #333;
				margin-bottom: 16rpx;
			}

			&__radio {
				display: flex;
				align-items: center;
				margin-bottom: 16rpx;

				&__text {
					font-size: 28rpx;
					color: #333;
				}
			}
		}
	}
</style>