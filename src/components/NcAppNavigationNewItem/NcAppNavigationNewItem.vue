<!--
 - @copyright Copyright (c) 2020 Jonathan Treffler <mail@jonathan-treffler.de>
 -
 - @author Jonathan Treffler <mail@jonathan-treffler.de>
 -
 - @license GNU AGPL version 3 or any later version
 -
 - This program is free software: you can redistribute it and/or modify
 - it under the terms of the GNU Affero General Public License as
 - published by the Free Software Foundation, either version 3 of the
 - License, or (at your option) any later version.
 -
 - This program is distributed in the hope that it will be useful,
 - but WITHOUT ANY WARRANTY; without even the implied warranty of
 - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 - GNU Affero General Public License for more details.
 -
 - You should have received a copy of the GNU Affero General Public License
 - along with this program. If not, see <http://www.gnu.org/licenses/>.
 -
 -->

<docs>
# Usage

### New Item element
```vue
	<template>
		<NcAppNavigationNewItem name="New Item" @new-item="function(value){alert(value)}">
			<template #icon>
				<Plus :size="20" />
			</template>
		</NcAppNavigationNewItem>
	</template>
	<script>
	import Plus from 'vue-material-design-icons/Plus'

	export default {
		components: {
			Plus,
		},
	}
	</script>
```

### New Item element with a loading animation instead of the icon
```vue
	<template>
		<NcAppNavigationNewItem name="New Item" :loading="true">
			<template #icon>
				<Plus :size="20" />
			</template>
		</NcAppNavigationNewItem>
	</template>
	<script>
	import Plus from 'vue-material-design-icons/Plus'

	export default {
		components: {
			Plus,
		},
	}
	</script>
```
</docs>
<template>
	<li :class="{
			'app-navigation-entry--newItemActive': newItemActive,
		}"
		class="app-navigation-entry">
		<!-- New Item -->
		<button class="app-navigation-entry-button" @click="handleNewItem">
			<span :class="{ [icon]: !loading }"
				class="app-navigation-entry-icon">
				<NcLoadingIcon v-if="loading" />
				<slot v-else name="icon" />
			</span>

			<span v-if="!newItemActive" class="app-navigation-new-item__name" :title="name">
				{{ name }}
			</span>

			<!-- new Item input -->
			<span v-if="newItemActive" class="newItemContainer">
				<NcInputConfirmCancel ref="newItemInput"
					v-model="newItemValue"
					:placeholder="editPlaceholder !== '' ? editPlaceholder : name"
					@cancel="cancelNewItem"
					@confirm="handleNewItemDone" />
			</span>
		</button>
	</li>
</template>

<script>
import NcInputConfirmCancel from '../NcAppNavigationItem/NcInputConfirmCancel.vue'
import NcLoadingIcon from '../NcLoadingIcon/index.js'

export default {
	name: 'NcAppNavigationNewItem',

	components: {
		NcInputConfirmCancel,
		NcLoadingIcon,
	},

	props: {
		/**
		 * The name of the element.
		 */
		name: {
			type: String,
			required: true,
		},
		/**
		 * Refers to the icon on the left, this prop accepts a class
		 * like 'icon-category-enabled'.
		 */
		icon: {
			type: String,
			default: '',
		},

		/**
		 * Displays a loading animated icon on the left of the element
		 * instead of the icon.
		 */
		loading: {
			type: Boolean,
			default: false,
		},
		/**
		 * Only for 'editable' items, sets label for the edit action button.
		 */
		editLabel: {
			type: String,
			default: '',
		},
		/**
		 * Sets the placeholder text for the editing form.
		 */
		editPlaceholder: {
			type: String,
			default: '',
		},
	},

	emits: ['new-item'],

	data() {
		return {
			newItemValue: '',
			newItemActive: false,
		}
	},

	methods: {
		handleNewItem() {
			if (!this.loading) {
				this.newItemActive = true
				this.$nextTick(() => {
					this.$refs.newItemInput.focusInput()
				})
			}
		},
		cancelNewItem() {
			this.newItemActive = false
		},
		handleNewItemDone() {
			this.$emit('new-item', this.newItemValue)
			this.newItemValue = ''
			this.newItemActive = false
		},
	},
}
</script>

<style lang="scss">
.app-navigation-new-item__name {
	overflow: hidden;
	max-width: 100%;
	white-space: nowrap;
	text-overflow: ellipsis;
	padding-left: 7px;
	font-size: 14px;
}

.newItemContainer {
	width: calc(100% - #{$clickable-area});
	margin: auto;
}
</style>
