<script>
	import Vue from 'vue'
	// 引入query-string
	import qs from 'query-string'
	export default {
		methods: {
			// 更新提示
			update() {
				const updateManager = wx.getUpdateManager()
				updateManager.onCheckForUpdate(function(res) {
					// 请求完新版本信息的回调
					console.log("是否有新版本？", res.hasUpdate)
				})
				updateManager.onUpdateReady(function() {
					wx.showModal({
						title: '发现新版本',
						content: '新版本已经准备好，是否重启应用？',
						success(res) {
							if (res.confirm) {
								// 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
								updateManager.applyUpdate()
							}
						}
					})
				})
			},
			// 初始化
			init() {
				uni.getSystemInfo({
					success: function(e) {
						// #ifndef MP
						Vue.prototype.StatusBar = e.statusBarHeight
						if (e.platform == 'android') {
							Vue.prototype.CustomBar = e.statusBarHeight + 50
						} else {
							Vue.prototype.CustomBar = e.statusBarHeight + 45
						}
						// #endif
						// #ifdef MP
						Vue.prototype.StatusBar = e.statusBarHeight
						let custom = wx.getMenuButtonBoundingClientRect()
						Vue.prototype.Custom = custom
						Vue.prototype.CustomBar = custom.bottom + custom.top - e.statusBarHeight
						// #endif
					}
				})
			},
			login() {
				wx.login({
					success: async res => {
						console.log("尝试登录", res)
						if (res.code) {
							//发起网络请求
							wx.getSetting({
								success: async setting => {
									if (setting.authSetting['scope.userInfo']) {
										// 已经授权，可以直接调用 getUserInfo 获取头像昵称、
										console.log("用户已授权", setting)
										wx.getUserInfo({
											success: async Info => {
												console.log("尝试获取用户信息", Info)
												let data = {
													code: res.code
												}
												Object.assign(data, Info.userInfo);
												let openid = await this.$api.login(data)
												uni.setStorageSync('openid', openid)
											}
										})
									} else {
										let openid = await this.$api.login({
											code: res.code
										})
										uni.setStorageSync('openid', openid)
									}
								}
							})
						} else {
							console.log('登录失败！' + res.errMsg)
						}
					}
				})
			}
		},
		onLaunch: function() {
			console.log('APP onLaunch')
			this.update()
			this.init()
			this.login()
		},
		onHide: function() {
			console.log('App Hide')
		},
	}
</script>

<style>
	/* 引入color UI wxss 库 */
	@import './static/css/main.wxss';
	@import './static/css/icon.wxss';
	@import './static/css/animation.wxss';
	/* ------------------ */
	/* 引入自定义 wxss 库 */
	@import './static/css/tmui.wxss';
</style>
