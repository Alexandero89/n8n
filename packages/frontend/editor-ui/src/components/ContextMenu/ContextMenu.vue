<script lang="ts" setup>
import { type ContextMenuAction, useContextMenu } from '@/composables/useContextMenu';
import { useStyles } from '@/composables/useStyles';
import { N8nActionDropdown } from '@n8n/design-system';
import { watch, ref } from 'vue';

const contextMenu = useContextMenu();
const { position, isOpen, actions, target } = contextMenu;
const dropdown = ref<InstanceType<typeof N8nActionDropdown>>();
const emit = defineEmits<{ action: [action: ContextMenuAction, nodeIds: string[]] }>();
const { APP_Z_INDEXES } = useStyles();

watch(
	isOpen,
	() => {
		if (isOpen) {
			dropdown.value?.open();
		} else {
			dropdown.value?.close();
		}
	},
	{ flush: 'post' },
);

function onActionSelect(item: string) {
	const action = item as ContextMenuAction;
	// Add handling logic for your new action
	if (action === 'custom_filter_by') {
		// Example: Emit an event, call a function, or trigger a state change
		console.log('Custom Filter By action triggered');
	}
	contextMenu._dispatchAction(action);
	emit('action', action, contextMenu.targetNodeIds.value);
}

function onVisibleChange(open: boolean) {
	if (!open) {
		contextMenu.close();
	}
}
</script>

<template>
	<Teleport v-if="isOpen" to="body">
		<div
			:class="$style.contextMenu"
			:style="{
				left: `${position[0]}px`,
				top: `${position[1]}px`,
				zIndex: APP_Z_INDEXES.CONTEXT_MENU,
			}"
		>
			<N8nActionDropdown
				ref="dropdown"
				:items="actions"
				placement="bottom-start"
				data-test-id="context-menu"
				:hide-arrow="target?.source !== 'node-button'"
				:teleported="false"
				@select="onActionSelect"
				@visible-change="onVisibleChange"
			>
				<template #activator>
					<div :class="$style.activator"></div>
				</template>

				<template #item="{ item }">
					<div :class="[$style.menuItem, { [$style.hasChildren]: item.children }]">
						{{ item.label }}
						<div v-if="item.children" :class="$style.submenu">
							<N8nActionDropdown
								:items="item.children"
								placement="right-start"
								@select="onActionSelect"
							/>
						</div>
					</div>
				</template>
			</N8nActionDropdown>
		</div>
	</Teleport>
</template>

<style module lang="scss">
.contextMenu {
	position: fixed;
}
.activator {
	pointer-events: none;
	opacity: 0;
}
.menuItem {
	position: relative;

	// Only show submenu when hovering the menuItem
	&:hover .submenu {
		display: block;
	}
}

.hasChildren {
	cursor: pointer;
}

.submenu {
	display: none; /* hidden by default */
	position: absolute;
	left: 100%;
	top: 0;
	z-index: 1000;
	background: white;
	border: 1px solid #ddd;
	min-width: 150px;
}
</style>
