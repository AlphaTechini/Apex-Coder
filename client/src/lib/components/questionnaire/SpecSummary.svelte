<script>
	let {
		spec = {},
		canStartBuild = false,
		isStartingBuild = false,
		onStartBuild = () => {},
		onEditSpec = () => {}
	} = $props();

	function formatArray(arr) {
		if (!arr || !Array.isArray(arr) || arr.length === 0) return 'None';
		return arr.join(', ');
	}
</script>

<div class="spec-summary bg-panel border border-white/10 rounded-xl p-8 shadow-neonSoft">
	<div class="mb-8 border-b border-white/10 pb-6">
		<h3 class="text-2xl font-bold text-text-primary mb-2">Project Specification Review</h3>
		<p class="text-text-secondary">
			Review your project details before generating the implementation plan.
		</p>
	</div>

	<div class="space-y-8">
		<!-- Project Overview -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>📱</span> Project Basics
			</h4>
			<div
				class="grid grid-cols-1 md:grid-cols-2 gap-6 bg-bg-secondary/50 p-6 rounded-lg border border-white/5"
			>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">App Name</span>
					<p class="text-white font-medium">{spec.project_overview?.app_name || 'Not specified'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Niche</span>
					<p class="text-white capitalize">{spec.project_overview?.niche || 'Not specified'}</p>
				</div>
				<div class="md:col-span-2">
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Summary</span>
					<p class="text-white/80">{spec.project_overview?.app_summary || 'Not specified'}</p>
				</div>
				<div class="md:col-span-2">
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Details</span>
					<p class="text-white/80 whitespace-pre-wrap">
						{spec.project_overview?.app_details || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Target Audience</span
					>
					<p class="text-white/80">{spec.project_overview?.potential_users || 'Not specified'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Scale</span>
					<p class="text-white">
						{spec.project_overview?.estimated_user_count || 'Not specified'} users
					</p>
				</div>
			</div>
		</section>

		<!-- User Flow -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>🗺️</span> User Journey
			</h4>
			<div class="bg-bg-secondary/50 p-6 rounded-lg border border-white/5 space-y-4">
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Overview Flow</span
					>
					<p class="text-white/80 whitespace-pre-wrap">
						{spec.user_flow?.overview_flow || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Key Actions</span>
					<p class="text-white/80">{spec.user_flow?.key_user_actions || 'Not specified'}</p>
				</div>
			</div>
		</section>

		<!-- App Structure -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>🏗️</span> Structure & Tech
			</h4>
			<div
				class="grid grid-cols-1 md:grid-cols-2 gap-6 bg-bg-secondary/50 p-6 rounded-lg border border-white/5"
			>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Type</span>
					<p class="text-white capitalize">{spec.app_structure?.app_type || 'Not specified'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Authentication</span
					>
					<p class="text-white">
						{spec.app_structure?.authentication_needed ? 'Required' : 'Not required'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Deployment</span>
					<p class="text-white uppercase">
						{spec.app_structure?.deployment_preference || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Roles</span>
					<p class="text-white/80">{spec.app_structure?.roles_or_permissions || 'None'}</p>
				</div>
			</div>
		</section>

		<!-- Data & Privacy -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>🛡️</span> Data & Privacy
			</h4>
			<div
				class="grid grid-cols-1 md:grid-cols-2 gap-6 bg-bg-secondary/50 p-6 rounded-lg border border-white/5"
			>
				<div class="md:col-span-2">
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Data Sources</span
					>
					<p class="text-white/80">{spec.data_flow?.data_sources || 'None'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Privacy Level</span
					>
					<p class="text-white capitalize">{spec.data_flow?.data_privacy || 'Not specified'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Storage</span>
					<p class="text-white capitalize">
						{spec.data_flow?.user_data_storage || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Public Sharing</span
					>
					<p class="text-white">{spec.data_flow?.data_shared_publicly ? 'Yes' : 'No'}</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Analytics</span>
					<p class="text-white">{spec.data_flow?.analytics_or_tracking ? 'Enabled' : 'Disabled'}</p>
				</div>
			</div>
		</section>

		<!-- Design -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>🎨</span> Design & Vibe
			</h4>
			<div
				class="grid grid-cols-1 md:grid-cols-2 gap-6 bg-bg-secondary/50 p-6 rounded-lg border border-white/5"
			>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Theme</span>
					<p class="text-white capitalize">
						{spec.design_preferences?.theme_style || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1">Vibe</span>
					<p class="text-white capitalize">
						{spec.design_preferences?.general_vibe || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Primary Color</span
					>
					<div class="flex items-center gap-2">
						<div
							class="w-4 h-4 rounded-full"
							style="background-color: {spec.design_preferences?.accent_color || '#3B82F6'}"
						></div>
						<p class="text-white uppercase">
							{spec.design_preferences?.accent_color || 'Not specified'}
						</p>
					</div>
				</div>
			</div>
		</section>

		<!-- Project Clarification -->
		<section>
			<h4 class="text-lg font-semibold text-accent-primary mb-4 flex items-center gap-2">
				<span>🎯</span> Goals & Vision
			</h4>
			<div class="bg-bg-secondary/50 p-6 rounded-lg border border-white/5 space-y-4">
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Primary Goals</span
					>
					<p class="text-white/80 whitespace-pre-wrap">
						{spec.project_clarification?.project_goals || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Success Metrics</span
					>
					<p class="text-white/80">
						{spec.project_clarification?.success_metrics || 'Not specified'}
					</p>
				</div>
				<div>
					<span class="block text-xs text-white/50 uppercase tracking-wider mb-1"
						>Unique Features</span
					>
					<p class="text-white/80">
						{spec.project_clarification?.unique_features || 'Not specified'}
					</p>
				</div>
			</div>
		</section>
	</div>

	<!-- Action Buttons -->
	<div class="mt-8 pt-6 border-t border-white/10 flex items-center justify-end gap-4">
		<button
			type="button"
			onclick={onEditSpec}
			class="px-6 py-3 text-sm font-medium text-white/70 hover:text-white transition-colors"
		>
			Edit Specification
		</button>

		<button
			type="button"
			onclick={onStartBuild}
			disabled={!canStartBuild || isStartingBuild}
			class="flex items-center gap-2 bg-accent-primary hover:shadow-neon px-8 py-3 rounded-lg text-black font-semibold transition-all disabled:opacity-50 disabled:cursor-not-allowed"
		>
			{#if isStartingBuild}
				<svg
					class="animate-spin h-4 w-4 text-black"
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
				>
					<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"
					></circle>
					<path
						class="opacity-75"
						fill="currentColor"
						d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
					></path>
				</svg>
				starting...
			{:else}
				<span>🚀</span> Generate Plan
			{/if}
		</button>
	</div>
</div>
