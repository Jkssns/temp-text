<template>
	<view class="content">
		<section>
			<div>
				<input style="border: 1px solid; height: 40px;" :style="{background: disabled ? '#eee' : ''}" type="text" maxlength="10" v-model="password" placeholder="请输入4-10位密码" :disabled="disabled" />
				<button @click="setPw">设置/输入密码</button>
			</div>
		</section>
		
		<main>
			<input style="height: 300px;border: 1px solid;" type="textarea" v-model="tempText" maxlength="-1" placeholder="请输入需要传输的文本"/>
			<button @click="handleConfirm">完成/更新</button>
			<button @click="handleDelete">删除本条记录（无副本，彻底删除）</button>
		</main>
		
		<footer class="footer_tip">Tip：任何人可以输入任何密码，请不要设置得过于简单。</footer>
	</view>
</template>

<script setup>
	import { ref } from 'vue'
	const tempText = ref('')
	const password = ref('')
	const tempItem = ref()
	const db = uniCloud.database();
	const collection = db.collection('temp-text');
	const disabled = ref(false)
	
	const setPw = () => {
		const pw = password.value
		if (pw.length >= 4 && pw.length <= 10) {
			collection.get().then(res => {
				const data = res.result.data;
				const item = data.find(item => item.password === pw)
				if (item) {
					uni.showToast({
						title: '获取成功',
						icon: 'none',
					});

					tempText.value = item.text
					tempItem.value = item
				} else {
					collection.add({
						password: password.value,
					}).then(res => {
						uni.showToast({
							title: '添加成功',
							icon: 'none',
						});
						disabled.value = true
						tempItem.value = {
							_id: res.result.id
						}
					}).catch(() => {
						handleErr()
					})
				}
			}).catch((err) => {
				handleErr()
			})
		} else {
			uni.showToast({
				title: '请输入4-10位密码',
				icon: 'none',
			});
		}
	}
	
	const handleConfirm = () => {
		if (!tempText.value) {
			uni.showToast({
				title: '请输入',
				icon: 'none',
			});
			return;
		}
		collection.doc(tempItem.value._id).update({
			text: tempText.value,
		}).then(() => {
			uni.showToast({
				title: '添加/更新成功',
				icon: 'none',
			});
		}).catch(() => {
			handleErr()
		})
	}
	
	const handleDelete = () => {
		collection.doc(tempItem.value._id).remove().then(res => {
			tempItem.value = {}
			password.value = ''
			tempText.value = ''
			disabled.value = false
			uni.showToast({
				title: '删除成功',
				icon: 'none',
			});
		}).catch(() => {
			handleErr()
		})
	}
	
	const handleErr = () => {
		alert('服务器错误，请联系维护人，微信号：Ted-Mosdy')
	}
	
</script>

<style>
	section {
		height: 40%;
	}
	main {
		margin-top: 30px;
	}
	.footer_tip {
		width: 100%;
		position: absolute;
		bottom: 10px;
		font-size: 16px;
		text-align: center;
	}
</style>
