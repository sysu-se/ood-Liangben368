<!-- src/components/Controls/ActionBar/Actions.svelte -->
<script>
	import { candidates } from '@sudoku/stores/candidates';
	import { userGrid, needsExplore } from '@sudoku/stores/grid';
	import { cursor } from '@sudoku/stores/cursor';
	import { hints, currentHint } from '@sudoku/stores/hints';
	import { notes } from '@sudoku/stores/notes';
	import { settings } from '@sudoku/stores/settings';
	import { keyboardDisabled } from '@sudoku/stores/keyboard';
	import { gamePaused } from '@sudoku/stores/game';
	import { explore } from '@sudoku/stores/explore';
	import { get } from 'svelte/store';

	$: hintsAvailable = $hints > 0;
	$: isExploring = $explore;

	function handleHint() {
		if (!hintsAvailable) return;
		
		const nextStep = hints.getNextStepHint();
		if (nextStep) {
			currentHint.set(nextStep);
			hints.useHint();
			cursor.set(nextStep.x, nextStep.y);
		} else {
			alert('当前局面需要使用探索模式进行尝试！');
		}
	}

	function applyHintValue() {
		const hint = get(currentHint);
		if (hint) {
			userGrid.set({ x: hint.x, y: hint.y }, hint.value);
			currentHint.set(null);
		}
	}

	function handleEnterExplore() {
		const $grid = get(userGrid);
		explore.enterExplore($grid);
	}

	function handleCommitExplore() {
		explore.commitExplore();
	}

	function handleAbandonExplore() {
		const restoredGrid = explore.abandonExplore();
		if (restoredGrid) {
			userGrid.set(restoredGrid);
		}
	}
</script>

<div class="action-buttons space-x-3">

	<!-- Undo -->
	<button class="btn btn-round" disabled={$gamePaused} on:click={() => userGrid.undo()} title="Undo">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6" />
		</svg>
	</button>

	<!-- Redo -->
	<button class="btn btn-round" disabled={$gamePaused} on:click={() => userGrid.redo()} title="Redo">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 10h-10a8 8 90 00-8 8v2M21 10l-6 6m6-6l-6-6" />
		</svg>
	</button>

	<!-- Hint -->
	<button class="btn btn-round btn-badge" disabled={$keyboardDisabled || !hintsAvailable} on:click={handleHint} title="Hints ({$hints})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
		</svg>

		{#if $settings.hintsLimited}
			<span class="badge" class:badge-primary={hintsAvailable}>{$hints}</span>
		{/if}
	</button>

	<!-- 探索模式按钮 -->
	{#if !isExploring}
		<button class="btn btn-round" disabled={$gamePaused || !$needsExplore} on:click={handleEnterExplore} title="进入探索模式">
			<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
			</svg>
		</button>
	{:else}
		<!-- 探索中：显示提交和放弃按钮 -->
		<button class="btn btn-round bg-green-500 text-white" on:click={handleCommitExplore} title="提交探索结果">
			<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
			</svg>
		</button>
		<button class="btn btn-round bg-red-500 text-white" on:click={handleAbandonExplore} title="放弃探索">
			<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
			</svg>
		</button>
	{/if}

	<!-- Notes -->
	<button class="btn btn-round btn-badge" on:click={notes.toggle} title="Notes ({$notes ? 'ON' : 'OFF'})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
		</svg>

		<span class="badge tracking-tighter" class:badge-primary={$notes}>{$notes ? 'ON' : 'OFF'}</span>
	</button>

</div>

<!-- 提示弹窗 -->
{#if $currentHint}
	<div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" on:click={() => currentHint.set(null)}>
		<div class="bg-white p-6 rounded-lg shadow-xl" on:click|stopPropagation>
			<h3 class="text-xl font-bold mb-2">找到下一步！</h3>
			<p class="mb-4">位置: ({$currentHint.x + 1}, {$currentHint.y + 1})</p>
			<p class="mb-4">应填数字: <span class="text-2xl font-bold text-blue-600">{$currentHint.value}</span></p>
			<p class="mb-4 text-sm text-gray-600">原因: {$currentHint.reason}</p>
			<div class="flex justify-end space-x-3">
				<button class="btn btn-gray" on:click={() => currentHint.set(null)}>稍后</button>
				<button class="btn btn-primary" on:click={applyHintValue}>填入</button>
			</div>
		</div>
	</div>
{/if}

<style>
	.action-buttons {
		@apply flex flex-wrap justify-evenly self-end;
	}

	.btn-badge {
		@apply relative;
	}

	.badge {
		min-height: 20px;
		min-width:  20px;
		@apply p-1 rounded-full leading-none text-center text-xs text-white bg-gray-600 inline-block absolute top-0 left-0;
	}

	.badge-primary {
		@apply bg-primary;
	}
</style>
