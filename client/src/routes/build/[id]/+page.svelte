<script>
	import { onMount, onDestroy } from 'svelte';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import PipelineStageCard from '$lib/components/PipelineStageCard.svelte';
	import StatusBadge from '$lib/components/StatusBadge.svelte';
	import TerminalLog from '$lib/components/TerminalLog.svelte';
	import ErrorDisplay from '$lib/components/ErrorDisplay.svelte';
	import RetryButton from '$lib/components/RetryButton.svelte';
	import CancelButton from '$lib/components/CancelButton.svelte';

	let buildId = $page.params.id;
	let buildData = $state(null);
	let pipelineStatus = $state(null);
	let stages = $state([]);
	let logs = $state([]);
	let error = $state(null);
	let loading = $state(true);
	let websocket = $state(null);
	let retrying = $state(false);
	let cancelling = $state(false);

	// Pipeline stage definitions
	const STAGE_DEFINITIONS = {
		'questionnaire': { label: 'Questionnaire Complete', description: 'User input processed' },
		'clarifier': { label: 'AI Clarification', description: 'Refining requirements with AI' },
		'normalizer': { label: 'Spec Normalization', description: 'Cleaning and structuring specs' },
		'docs-creator': { label: 'Documentation', description: 'Generating project documentation' },
		'schema-generator': { label: 'Database Schema', description: 'Creating data models' },
		'structural-validator': { label: 'Structure Validation', description: 'Validating architecture' },
		'file-structure-generator': { label: 'File Structure', description: 'Planning project files' },
		'validator': { label: 'Final Validation', description: 'Cross-checking all components' },
		'empty-file-creation': { label: 'File Creation', description: 'Creating empty project files' },
		'code-generation': { label: 'Code Generation', description: 'AI writing your code' },
		'repo-creation': { label: 'GitHub Repository', description: 'Creating and pushing to GitHub' },
		'aws-deployment': { label: 'AWS Deployment', description: 'Deploying to the cloud' }
	};

	onMount(async () => {
		await loadBuildData();
		setupWebSocket();
	});

	onDestroy(() => {
		if (websocket) {
			websocket.close();
		}
	});

	async function loadBuildData() {
		try {
			loading = true;
			error = null;

			// Load build data
			const buildResponse = await fetch(`/api/builds/${buildId}`, {
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token') || ''}`
				}
			});

			if (!buildResponse.ok) {
				if (buildResponse.status === 404) {
					error = 'Build not found';
					return;
				}
				throw new Error(`Failed to load build: ${buildResponse.status}`);
			}

			const buildResult = await buildResponse.json();
			buildData = buildResult.data;

			// Load pipeline status
			const statusResponse = await fetch(`/api/builds/${buildId}/pipeline-status`, {
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token') || ''}`
				}
			});

			if (statusResponse.ok) {
				const statusResult = await statusResponse.json();
				pipelineStatus = statusResult.data;
			}

			// Load stages
			const stagesResponse = await fetch(`/api/builds/${buildId}/stages`, {
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token') || ''}`
				}
			});

			if (stagesResponse.ok) {
				const stagesResult = await stagesResponse.json();
				stages = transformStages(stagesResult.data.stageStatuses || {});
			}

		} catch (err) {
			console.error('Error loading build data:', err);
			error = err.message;
		} finally {
			loading = false;
		}
	}

	function transformStages(stageStatuses) {
		const stageList = [];
		
		// Create stages from status data
		Object.entries(stageStatuses).forEach(([stageName, status]) => {
			const definition = STAGE_DEFINITIONS[stageName] || {
				label: stageName,
				description: `Stage: ${stageName}`
			};

			stageList.push({
				id: stageName,
				label: definition.label,
				description: definition.description,
				status: status.status || 'pending',
				startedAt: status.startedAt,
				completedAt: status.completedAt,
				error: status.error,
				events: status.events || []
			});
		});

		// If no stages yet, create default pipeline stages
		if (stageList.length === 0) {
			Object.entries(STAGE_DEFINITIONS).forEach(([stageName, definition]) => {
				stageList.push({
					id: stageName,
					label: definition.label,
					description: definition.description,
					status: 'pending',
					events: []
				});
			});
		}

		return stageList;
	}

	function setupWebSocket() {
		try {
			const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
			const wsUrl = `${protocol}//${window.location.host}/api/pipelines/${buildId}/stream`;
			
			websocket = new WebSocket(wsUrl);

			websocket.onopen = () => {
				console.log('WebSocket connected for build', buildId);
			};

			websocket.onmessage = (event) => {
				try {
					const data = JSON.parse(event.data);
					handleWebSocketMessage(data);
				} catch (err) {
					console.error('Error parsing WebSocket message:', err);
				}
			};

			websocket.onclose = () => {
				console.log('WebSocket disconnected');
				// Attempt to reconnect after 5 seconds if build is still active
				if (buildData?.status === 'running' || buildData?.status === 'queued') {
					setTimeout(() => {
						if (!websocket || websocket.readyState === WebSocket.CLOSED) {
							setupWebSocket();
						}
					}, 5000);
				}
			};

			websocket.onerror = (err) => {
				console.error('WebSocket error:', err);
			};
		} catch (err) {
			console.error('Failed to setup WebSocket:', err);
		}
	}

	function handleWebSocketMessage(data) {
		console.log('WebSocket message:', data);

		if (data.type === 'build_status') {
			if (buildData) {
				buildData = { ...buildData, status: data.status, progress: data.progress };
			}
		} else if (data.type === 'stage_update') {
			updateStageStatus(data.stage, data.status, data.details);
		} else if (data.type === 'log') {
			logs = [...logs, {
				timestamp: new Date(),
				level: data.level || 'info',
				message: data.message,
				stage: data.stage
			}];
		} else if (data.type === 'error') {
			error = data.message;
		}
	}

	function updateStageStatus(stageName, status, details = {}) {
		stages = stages.map(stage => {
			if (stage.id === stageName) {
				return {
					...stage,
					status,
					...details,
					completedAt: status === 'completed' ? new Date() : stage.completedAt
				};
			}
			return stage;
		});
	}

	async function handleRetry() {
		try {
			retrying = true;
			error = null;

			const response = await fetch(`/api/pipelines/${buildId}/retry`, {
				method: 'POST',
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token') || ''}`,
					'Content-Type': 'application/json'
				}
			});

			if (response.ok) {
				const result = await response.json();
				// Redirect to new build if a new one was created
				if (result.data?.buildId && result.data.buildId !== buildId) {
					goto(`/build/${result.data.buildId}`);
				} else {
					// Reload current build data
					await loadBuildData();
				}
			} else {
				const errorData = await response.json();
				error = errorData.message || 'Failed to retry build';
			}
		} catch (err) {
			console.error('Error retrying build:', err);
			error = err.message;
		} finally {
			retrying = false;
		}
	}

	async function handleCancel() {
		try {
			cancelling = true;

			const response = await fetch(`/api/pipelines/${buildId}/cancel`, {
				method: 'POST',
				headers: {
					'Authorization': `Bearer ${localStorage.getItem('token') || ''}`,
					'Content-Type': 'application/json'
				}
			});

			if (response.ok) {
				await loadBuildData();
			} else {
				const errorData = await response.json();
				error = errorData.message || 'Failed to cancel build';
			}
		} catch (err) {
			console.error('Error cancelling build:', err);
			error = err.message;
		} finally {
			cancelling = false;
		}
	}

	function getOverallProgress() {
		if (!stages.length) return 0;
		
		const completedStages = stages.filter(s => s.status === 'completed').length;
		return Math.round((completedStages / stages.length) * 100);
	}

	function getCurrentStage() {
		const runningStage = stages.find(s => s.status === 'running');
		if (runningStage) return runningStage;
		
		const lastCompleted = stages.filter(s => s.status === 'completed').pop();
		if (lastCompleted) {
			const nextIndex = stages.findIndex(s => s.id === lastCompleted.id) + 1;
			return stages[nextIndex] || lastCompleted;
		}
		
		return stages[0];
	}

	// Reactive values
	let overallProgress = $derived(getOverallProgress());
	let currentStage = $derived(getCurrentStage());
	let isActive = $derived(buildData?.status === 'running' || buildData?.status === 'queued');
	let isCompleted = $derived(buildData?.status === 'completed');
	let isFailed = $derived(buildData?.status === 'failed');
	let canRetry = $derived(isFailed && !retrying);
	let canCancel = $derived(isActive && !cancelling);
</script>

<svelte:head>
	<title>Build {buildId} - AI App Builder</title>
</svelte:head>

<div class="min-h-screen bg-bg-primary py-8">
	<div class="mx-auto max-w-6xl px-4 sm:px-6 lg:px-8">
		<!-- Header -->
		<div class="mb-8">
			<div class="flex items-center justify-between">
				<div>
					<h1 class="text-3xl font-bold text-white">Build Progress</h1>
					<p class="mt-2 text-text-secondary">
						Build ID: <span class="font-mono text-accent-primary">{buildId}</span>
					</p>
				</div>
				
				<div class="flex space-x-3">
					{#if canCancel}
						<CancelButton onclick={handleCancel} loading={cancelling} />
					{/if}
					{#if canRetry}
						<RetryButton onclick={handleRetry} loading={retrying} />
					{/if}
				</div>
			</div>

			<!-- Overall Status -->
			<div class="mt-6 rounded-xl border border-white/10 bg-bg-panel p-6">
				<div class="flex items-center justify-between">
					<div class="flex items-center space-x-4">
						<StatusBadge status={buildData?.status || 'unknown'} />
						<div>
							<h2 class="text-lg font-semibold text-white">
								{buildData?.projectName || 'Project Build'}
							</h2>
							{#if currentStage}
								<p class="text-sm text-text-secondary">
									Current: {currentStage.label}
								</p>
							{/if}
						</div>
					</div>
					
					<div class="text-right">
						<div class="text-2xl font-bold text-accent-primary">
							{overallProgress}%
						</div>
						<div class="text-sm text-text-secondary">Complete</div>
					</div>
				</div>

				<!-- Progress Bar -->
				<div class="mt-4">
					<div class="h-2 rounded-full bg-white/10">
						<div 
							class="h-2 rounded-full bg-accent-primary transition-all duration-500"
							style="width: {overallProgress}%"
						></div>
					</div>
				</div>
			</div>
		</div>

		{#if loading}
			<!-- Loading State -->
			<div class="text-center">
				<div class="inline-block h-8 w-8 animate-spin rounded-full border-4 border-accent-primary border-t-transparent"></div>
				<p class="mt-4 text-text-secondary">Loading build data...</p>
			</div>
		{:else if error}
			<!-- Error State -->
			<ErrorDisplay {error} onRetry={loadBuildData} />
		{:else}
			<!-- Main Content -->
			<div class="grid grid-cols-1 gap-8 lg:grid-cols-3">
				<!-- Pipeline Stages -->
				<div class="lg:col-span-2">
					<h3 class="mb-4 text-xl font-semibold text-white">Pipeline Stages</h3>
					<div class="space-y-4">
						{#each stages as stage}
							<PipelineStageCard
								title={stage.label}
								description={stage.description}
								status={stage.status}
								startedAt={stage.startedAt}
								completedAt={stage.completedAt}
								error={stage.error}
								events={stage.events}
							/>
						{/each}
					</div>
				</div>

				<!-- Sidebar -->
				<div class="space-y-6">
					<!-- Build Info -->
					<div class="rounded-xl border border-white/10 bg-bg-panel p-4">
						<h4 class="mb-3 font-semibold text-white">Build Information</h4>
						<div class="space-y-2 text-sm">
							<div class="flex justify-between">
								<span class="text-text-secondary">Status:</span>
								<StatusBadge status={buildData?.status || 'unknown'} />
							</div>
							{#if buildData?.startedAt}
								<div class="flex justify-between">
									<span class="text-text-secondary">Started:</span>
									<span class="text-white">
										{new Date(buildData.startedAt).toLocaleString()}
									</span>
								</div>
							{/if}
							{#if buildData?.completedAt}
								<div class="flex justify-between">
									<span class="text-text-secondary">Completed:</span>
									<span class="text-white">
										{new Date(buildData.completedAt).toLocaleString()}
									</span>
								</div>
							{/if}
						</div>
					</div>

					<!-- Live Logs -->
					{#if logs.length > 0}
						<div class="rounded-xl border border-white/10 bg-bg-panel p-4">
							<h4 class="mb-3 font-semibold text-white">Live Logs</h4>
							<TerminalLog {logs} maxHeight="300px" />
						</div>
					{/if}

					<!-- Results -->
					{#if isCompleted && buildData?.resources}
						<div class="rounded-xl border border-white/10 bg-bg-panel p-4">
							<h4 class="mb-3 font-semibold text-white">Deployment Results</h4>
							<div class="space-y-2">
								{#if buildData.resources.appUrl}
									<a 
										href={buildData.resources.appUrl}
										target="_blank"
										rel="noopener noreferrer"
										class="block rounded-lg bg-accent-primary px-4 py-2 text-center font-semibold text-black hover:shadow-neon transition-all"
									>
										View Live App →
									</a>
								{/if}
								{#if buildData.resources.repoUrl}
									<a 
										href={buildData.resources.repoUrl}
										target="_blank"
										rel="noopener noreferrer"
										class="block rounded-lg border border-white/20 bg-white/10 px-4 py-2 text-center font-semibold text-white hover:bg-white/20 transition-all"
									>
										View GitHub Repo →
									</a>
								{/if}
							</div>
						</div>
					{/if}
				</div>
			</div>
		{/if}
	</div>
</div>