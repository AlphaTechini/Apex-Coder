<script>
	import { slide } from 'svelte/transition';

	export let errors = {};
	export let onDismiss = () => {};

	// Compute total error count
	$: errorCount = Object.keys(errors).reduce((count, field) => count + errors[field].length, 0);

	// Get first error message for display priority
	$: firstErrorField = Object.keys(errors)[0];
	$: firstErrorMessage = firstErrorField ? errors[firstErrorField][0] : '';
</script>

{#if errorCount > 0}
	<div
		transition:slide={{ duration: 300, axis: 'y' }}
		class="fixed top-24 left-1/2 transform -translate-x-1/2 z-50 w-full max-w-lg px-4"
	>
		<div
			class="bg-accent-error/10 border border-accent-error/40 text-white rounded-xl shadow-2xl backdrop-blur-md p-4 flex items-start gap-4"
		>
			<div class="flex-shrink-0 mt-1">
				<svg
					class="w-6 h-6 text-accent-error"
					fill="none"
					stroke="currentColor"
					viewBox="0 0 24 24"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
					/>
				</svg>
			</div>

			<div class="flex-1">
				<h3 class="font-bold text-accent-error text-lg">
					Please check your inputs ({errorCount} error{errorCount > 1 ? 's' : ''})
				</h3>
				<p class="text-white/80 text-sm mt-1">
					{firstErrorMessage}
					{#if errorCount > 1}
						<span class="opacity-60 block mt-1">+ {errorCount - 1} more issues</span>
					{/if}
				</p>
			</div>

			<button
				on:click={onDismiss}
				class="flex-shrink-0 text-white/40 hover:text-white transition-colors"
			>
				<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M6 18L18 6M6 6l12 12"
					/>
				</svg>
			</button>
		</div>
	</div>
{/if}
