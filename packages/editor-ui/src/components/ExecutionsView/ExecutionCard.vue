<template>
	<div
		:class="{
			['execution-card']: true,
			[$style.executionCard]: true,
			[$style.active]: isActive,
			[$style[executionUIDetails.name]]: true,
			[$style.highlight]: highlight,
		}"
	>
		<router-link
			:class="$style.executionLink"
			:to="{
				name: VIEWS.EXECUTION_PREVIEW,
				params: { workflowId: currentWorkflow, executionId: execution.id },
			}"
			:data-test-execution-status="executionUIDetails.name"
		>
			<div :class="$style.description">
				<n8n-text color="text-dark" :bold="true" size="medium" data-test-id="execution-time">
					{{ executionUIDetails.startTime }}
				</n8n-text>
				<div :class="$style.executionStatus">
					<n8n-spinner
						v-if="executionUIDetails.name === 'running'"
						size="small"
						:class="[$style.spinner, 'mr-4xs']"
					/>
					<n8n-text :class="$style.statusLabel" size="small">{{
						executionUIDetails.label
					}}</n8n-text>
					<n8n-text
						v-if="executionUIDetails.name === 'running'"
						:color="isActive ? 'text-dark' : 'text-base'"
						size="small"
					>
						{{ $locale.baseText('executionDetails.runningTimeRunning') }}
						<execution-time :start-time="execution.startedAt" />
					</n8n-text>
					<n8n-text
						v-else-if="
							executionUIDetails.name !== 'waiting' && executionUIDetails.name !== 'unknown'
						"
						:color="isActive ? 'text-dark' : 'text-base'"
						size="small"
					>
						{{
							$locale.baseText('executionDetails.runningTimeFinished', {
								interpolate: { time: executionUIDetails.runningTime },
							})
						}}
					</n8n-text>
				</div>
				<div v-if="execution.mode === 'retry'">
					<n8n-text :color="isActive ? 'text-dark' : 'text-base'" size="small">
						{{ $locale.baseText('executionDetails.retry') }} #{{ execution.retryOf }}
					</n8n-text>
				</div>
			</div>
			<div :class="$style.icons">
				<n8n-action-dropdown
					v-if="executionUIDetails.name === 'error'"
					:class="[$style.icon, $style.retry]"
					:items="retryExecutionActions"
					activatorIcon="redo"
					data-test-id="retry-execution-button"
					@select="onRetryMenuItemSelect"
				/>
				<n8n-tooltip v-if="execution.mode === 'manual'" placement="top">
					<template #content>
						<span>{{ $locale.baseText('executionsList.test') }}</span>
					</template>
					<font-awesome-icon
						v-if="execution.mode === 'manual'"
						:class="[$style.icon, $style.manual]"
						icon="flask"
					/>
				</n8n-tooltip>
			</div>
		</router-link>
	</div>
</template>

<script lang="ts">
import { IExecutionsSummary } from '@/Interface';
import mixins from 'vue-typed-mixins';
import { executionHelpers, IExecutionUIData } from '@/mixins/executionsHelpers';
import { VIEWS } from '@/constants';
import { showMessage } from '@/mixins/showMessage';
import { restApi } from '@/mixins/restApi';
import ExecutionTime from '@/components/ExecutionTime.vue';

export default mixins(executionHelpers, showMessage, restApi).extend({
	name: 'execution-card',
	components: {
		ExecutionTime,
	},
	data() {
		return {
			VIEWS,
		};
	},
	props: {
		execution: {
			type: Object as () => IExecutionsSummary,
			required: true,
		},
		highlight: {
			type: Boolean,
			default: false,
		},
	},
	computed: {
		retryExecutionActions(): object[] {
			return [
				{
					id: 'current-workflow',
					label: this.$locale.baseText('executionsList.retryWithCurrentlySavedWorkflow'),
				},
				{
					id: 'original-workflow',
					label: this.$locale.baseText('executionsList.retryWithOriginalWorkflow'),
				},
			];
		},
		executionUIDetails(): IExecutionUIData {
			return this.getExecutionUIDetails(this.execution);
		},
		isActive(): boolean {
			return this.execution.id === this.$route.params.executionId;
		},
	},
	methods: {
		onRetryMenuItemSelect(action: string): void {
			this.$emit('retryExecution', { execution: this.execution, command: action });
		},
	},
});
</script>

<style module lang="scss">
.executionCard {
	display: flex;
	padding-right: var(--spacing-m);

	&.active {
		border-left: var(--spacing-4xs) var(--border-style-base) transparent !important;

		.executionStatus {
			color: var(--color-text-dark) !important;
		}
	}

	&:hover,
	&.active {
		.executionLink {
			background-color: var(--color-foreground-light);
		}
	}

	&.running {
		.spinner {
			position: relative;
			top: 1px;
		}
		&,
		& .executionLink {
			border-left: var(--spacing-4xs) var(--border-style-base) hsl(var(--color-warning-h), 94%, 80%);
		}
		.statusLabel,
		.spinner {
			color: var(--color-warning);
		}
	}

	&.success {
		&,
		& .executionLink {
			border-left: var(--spacing-4xs) var(--border-style-base) hsl(var(--color-success-h), 60%, 70%);
		}
	}

	&.waiting {
		&,
		& .executionLink {
			border-left: var(--spacing-4xs) var(--border-style-base)
				hsl(var(--color-secondary-h), 94%, 80%);
		}
		.statusLabel {
			color: var(--color-secondary);
		}
	}

	&.error {
		&,
		& .executionLink {
			border-left: var(--spacing-4xs) var(--border-style-base) hsl(var(--color-danger-h), 94%, 80%);
		}
		.statusLabel {
			color: var(--color-danger);
		}
	}

	&.unknown {
		&,
		& .executionLink {
			border-left: var(--spacing-4xs) var(--border-style-base) var(--color-text-light);
		}
	}
}

.executionLink {
	display: flex;
	width: 100%;
	align-items: center;
	justify-content: space-between;
	color: var(--color-text-base);
	font-size: var(--font-size-xs);
	padding: var(--spacing-xs);
	padding-right: var(--spacing-s);
	position: relative;
	left: calc(
		-1 * var(--spacing-4xs)
	); // Hide link border under card border so it's not visible when not hovered

	&:active {
		.icon,
		.statusLabel {
			color: var(--color-text-base);
		}
	}
}

.icons {
	display: flex;
	align-items: baseline;
}

.icon {
	font-size: var(--font-size-s);

	&.retry {
		svg {
			color: var(--color-primary);
		}
	}

	&.manual {
		position: relative;
		top: 1px;
	}

	& + & {
		margin-left: var(--spacing-2xs);
	}
}
</style>
