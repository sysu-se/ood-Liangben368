<script>
	import { userGrid, needsExplore, canUndo, canRedo } from '@sudoku/stores/grid';
	import { hints, currentHint } from '@sudoku/stores/hints';
	import { explore } from '@sudoku/stores/explore';
	import { modal } from '@sudoku/stores/modal';

	function handleUndo() {
		userGrid.undoGrid();
	}

	function handleRedo() {
		userGrid.redoGrid();
	}

	function handleHint() {
		const hint = hints.getNextStepHint();
		if (hint) {
			currentHint.set(hint);
			modal.show('hint', {
				hint,
				onApply: () => {
					hints.useHint();
					userGrid.applyHint({ x: hint.x, y: hint.y });
					modal.hide();
				}
			});
		} else {
			modal.show('info', {
				title: '没有可用提示',
				text: '当前棋盘没有可以逻辑推断的下一步，请进入探索模式尝试。'
			});
		}
	}

	function handleEnterExplore() {
		explore.enterExplore();
	}

	function handleCommitExplore() {
		try {
			explore.commitExplore();
		} catch (e) {
			modal.show('error', {
				title: '提交失败',
				text: e.message
			});
		}
	}

	function handleAbandonExplore() {
		modal.show('confirm', {
			title: '放弃探索',
			text: '确定要放弃本次探索吗？所有探索中的修改将被撤销。',
			button: '放弃',
			callback: () => {
				explore.abandonExplore();
			}
		});
	}
</script>

<div class="action-bar flex justify-between items-center mb-4">
	<div class="flex space-x-2">
		<button class="btn btn-sm" on:click={handleUndo} disabled={!$canUndo}>
			Undo
		</button>
		<button class="btn btn-sm" on:click={handleRedo} disabled={!$canRedo}>
			Redo
		</button>
	</div>

	<div class="flex space-x-2">
		{#if !$explore.isExploring}
			<button class="btn btn-sm btn-primary" on:click={handleHint} disabled={$hints <= 0}>
				提示 ({ $hints === Infinity ? '∞' : $hints })
			</button>
			
			{#if $needsExplore}
				<button class="btn btn-sm btn-warning" on:click={handleEnterExplore}>
					进入探索模式
				</button>
			{/if}
		{:else}
			<span class="font-bold"
			      class:text-yellow-500={!$explore.hasConflict}
			      class:text-red-500={$explore.hasConflict}>
				{#if $explore.hasConflict}
					⚠️ 探索冲突
				{:else}
					🔍 探索模式
				{/if}
			</span>
			<button class="btn btn-sm btn-success" 
			        on:click={handleCommitExplore}
			        disabled={$explore.hasConflict}>
				提交
			</button>
			<button class="btn btn-sm btn-danger" on:click={handleAbandonExplore}>
				放弃
			</button>
		{/if}
	</div>
</div>

<style>
	.btn-sm {
		@apply py-1 px-3 text-sm;
	}
	
	.btn-warning {
		@apply bg-yellow-500 hover:bg-yellow-600 text-white;
	}
	
	.btn-success {
		@apply bg-green-500 hover:bg-green-600 text-white;
	}
	
	.btn-success:disabled {
		@apply bg-gray-400 cursor-not-allowed;
	}
	
	.btn-danger {
		@apply bg-red-500 hover:bg-red-600 text-white;
	}
</style>
