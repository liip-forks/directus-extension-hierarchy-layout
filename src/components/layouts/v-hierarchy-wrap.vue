<template>
	<div class="forest-view-wrap">
		<v-list-item
			block
			dense
			clickable
			class="group"
			:class="selectedClass"
			@click="edit({item, event: $event})"
		>
			<v-list-item-icon>
				<v-icon v-if="selectMode" :name="checkedIcon"></v-icon>
				<v-icon v-else class="drag-handle" name="drag_handle"/>
			</v-list-item-icon>

			<div class="" style="flex-grow: 1">
				<render-template
					:collection="collection"
					:item="item"
					:template="title"/>
			</div>

			<div>
				<v-icon
					v-if="hasChildren"
					v-tooltip="expand ? t('collapse') : t('expand')"
					:name="expand ? 'unfold_less' : 'unfold_more'"
					:class="expand ? 'unfold_less' : 'unfold_more'"
					clickable
					class="collapse-toggle"
					@click.stop.prevent="expand= !expand"
				/>
			</div>
		</v-list-item>

		<template v-if="expand">
			<slot></slot>
		</template>

	</div>
</template>

<script setup lang="ts">
import {computed, ref} from "vue";
import {useI18n} from 'vue-i18n';

const {t} = useI18n();
const expand = ref(true);

const props = defineProps<{
	item: any;
	optTitle: string;
	optParentField: string;
	collection: string;
	hasChildren: boolean;
	selectMode: boolean;
	selection: Array<number | string>;
	edit: (item) => void;
}>();

/**
 * A row at the top level renders the title without its parent placeholders, and
 * without the brackets around them: `{{name}} ({{parent.name}})` reads
 * "Home (--)" otherwise, because `render-template` prints a null value as `--`
 * and a Directus template has no conditionals.
 *
 * Dropping the brackets is the part a display option cannot do. The field store
 * resolves `parent.name` to the very same field as `name`, so a prefix/suffix
 * set there to bracket the parent would bracket the row's own name as well.
 */
const title = computed(() => {
	if (props.item[props.optParentField]) return props.optTitle;

	const placeholder = `{{\\s*${props.optParentField}(\\.[^}]*)?\\s*}}`;

	return props.optTitle
		.replace(new RegExp(`\\s*[([{]?\\s*${placeholder}\\s*[)\\]}]?`, 'g'), '')
		.trim();
});

const checkedIcon = computed(() => {
	return props.selection?.indexOf(props.item.id) >= 0 ? 'check_circle' : 'radio_button_unchecked';
});

const selectedClass = computed(() => {
	return props.selection?.indexOf(props.item.id) >= 0 ? 'selected' : '';
});

</script>

<style scoped>
.collection-item {
	margin-top: 8px;
}

.unfold_less {
	opacity: 0.4;
}

.forest-view-wrap .group.selected {
	background-color: var(--theme--background-normal) !important;
	border-color: var(--theme--primary-subdued) !important;
}
</style>
