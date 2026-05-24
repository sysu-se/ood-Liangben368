<script>
	export let data;
	export let hideModal;
</script>

<div class="text-center">
	<h2 class="text-2xl font-bold mb-4">下一步提示</h2>
	
	<div class="mb-4">
		<p class="text-lg mb-2">
			位置：第 {data.hint.y + 1} 行，第 {data.hint.x + 1} 列
		</p>
		<p class="text-lg mb-2">
			应填数字：<span class="text-primary font-bold text-3xl">{data.hint.value}</span>
		</p>
		<p class="text-gray-600">
			原因：{data.hint.reason}
		</p>
	</div>
	
	<div class="flex justify-center space-x-4">
		<button class="btn" on:click={hideModal}>
			知道了
		</button>
		<button class="btn btn-primary" on:click={() => { data.onApply(); hideModal(); }}>
			自动填写
		</button>
	</div>
</div>
