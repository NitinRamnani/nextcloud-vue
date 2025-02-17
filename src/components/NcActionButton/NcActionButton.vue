<!--
  - @copyright Copyright (c) 2019 John Molakvoæ <skjnldsv@protonmail.com>
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
  - @author Marco Ambrosini <marcoambrosini@icloud.com>
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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<docs>
This component is made to be used inside of the [NcActions](#NcActions) component slots.

```vue
	<template>
		<div style="display: flex; align-items: center;">
			<NcActions>
				<NcActionButton @click="showMessage('Delete')">
					<template #icon>
						<Delete :size="20" />
					</template>
					Delete
				</NcActionButton>
				<NcActionButton :close-after-click="true" @click="showMessage('Delete and close menu')">
					<template #icon>
						<Delete :size="20" />
					</template>
					Delete and close
				</NcActionButton>
				<NcActionButton :close-after-click="true" @click="focusInput">
					<template #icon>
						<Plus :size="20" />
					</template>
					Create
				</NcActionButton>
				<NcActionButton :disabled="true" @click="showMessage('Disabled')">
					<template #icon>
						<Delete :size="20" />
					</template>
					Disabled button
				</NcActionButton>
			</NcActions>
			<input ref="input" />
		</div>
	</template>
	<script>
	import Delete from 'vue-material-design-icons/Delete'
	import Plus from 'vue-material-design-icons/Plus'

	export default {
		components: {
			Delete,
			Plus,
		},
		methods: {
			showMessage(msg) {
				alert(msg)
			},
			focusInput() {
				this.$nextTick(() => this.$refs.input.focus())
			},
		},
	}
	</script>
```

If you're using a long text you can specify a name

```vue
	<template>
		<NcActions>
			<NcActionButton @click="showMessage('Add')">
				<template #icon>
					<Plus :size="20" />
				</template>
				Add new
			</NcActionButton>
			<NcActionButton name="Long button" @click="showMessage('Delete')">
				<template #icon>
					<Delete :size="20" />
				</template>
				This button is associated with a very long text.\nAnd with new lines too.
			</NcActionButton>
		</NcActions>
	</template>
	<script>
	import Delete from 'vue-material-design-icons/Delete'
	import Plus from 'vue-material-design-icons/Plus'

	export default {
		components: {
			Delete,
			Plus,
		},
		methods: {
			showMessage(msg) {
				alert(msg)
			},
		},
	}
	</script>

```

Action icon attribute with a single action

```vue
	<template>
		<NcActions>
			<NcActionButton @click="showMessage('Add')">
				<template #icon>
					<Plus :size="20" />
				</template>
				Add new
			</NcActionButton>
		</NcActions>
	</template>
	<script>
	import Plus from 'vue-material-design-icons/Plus'

	export default {
		components: {
			Plus,
		},
		methods: {
			showMessage(msg) {
				alert(msg)
			},
		},
	}
	</script>

```

You can also use a custom icon, for example from the vue-material-design-icons library:

```vue
<template>
	<NcActions>
		<NcActionButton>
			<template #icon>
				<HandBackLeft :size="20" />
			</template>
			Raise left hand
		</NcActionButton>
		<NcActionButton>
			<template #icon>
				<HandBackRight :size="20" />
			</template>
			Raise right hand
		</NcActionButton>
	</NcActions>
</template>
<script>
import HandBackLeft from 'vue-material-design-icons/HandBackLeft'
import HandBackRight from 'vue-material-design-icons/HandBackRight'

export default {
	components: {
		HandBackLeft,
		HandBackRight,
	},
}
</script>
```
</docs>

<template>
	<li class="action" role="presentation" :class="{ 'action--disabled': disabled }">
		<button class="action-button"
			:class="{ focusable: isFocusable }"
			:aria-label="ariaLabel"
			:title="title"
			role="menuitem"
			type="button"
			@click="onClick">
			<!-- @slot Manually provide icon -->
			<slot name="icon">
				<span :class="[isIconUrl ? 'action-button__icon--url' : icon]"
					:style="{ backgroundImage: isIconUrl ? `url(${icon})` : null }"
					:aria-hidden="ariaHidden"
					class="action-button__icon" />
			</slot>

			<!-- long text with name -->
			<p v-if="name">
				<strong class="action-button__name">
					{{ name }}
				</strong>
				<br>
				<!-- white space is shown on longtext, so we can't
					put {{ text }} on a new line for code readability -->
				<span class="action-button__longtext" v-text="text" />
			</p>

			<!-- long text only -->
			<!-- white space is shown on longtext, so we can't
				put {{ text }} on a new line for code readability -->
			<p v-else-if="isLongText"
				class="action-button__longtext"
				v-text="text" />

			<!-- default text display -->
			<span v-else class="action-button__text">{{ text }}</span>

			<!-- fake slot to gather inner text -->
			<slot v-if="false" />
		</button>
	</li>
</template>

<script>
import ActionTextMixin from '../../mixins/actionText.js'

/**
 * Button component to be used in Actions
 */
export default {
	name: 'NcActionButton',

	mixins: [ActionTextMixin],

	props: {
		/**
		 * disabled state of the action button
		 */
		disabled: {
			type: Boolean,
			default: false,
		},
		/**
		 * aria-hidden attribute for the icon slot
		 */
		ariaHidden: {
			type: Boolean,
			default: null,
		},
	},
	computed: {
		/**
		 * determines if the action is focusable
		 *
		 * @return {boolean} is the action focusable ?
		 */
		isFocusable() {
			return !this.disabled
		},
	},
}
</script>

<style lang="scss" scoped>
@import '../../assets/action';
@include action-active;
@include action--disabled;
@include action-item('button');
</style>
