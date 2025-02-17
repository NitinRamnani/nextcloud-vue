 <!--
  - @copyright Copyright (c) 2020 John Molakvoæ <skjnldsv@protonmail.com>
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
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
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<docs>
This component is made to be used in the Nextcloud top header.

```
<template>
	<div id="nextcloud-header">
		<NcHeaderMenu id="search"
			aria-label="Search">
			<template #trigger>
				<Magnify />
			</template>
			<div>
				<input placeholder="Search for files, comments, contacts..." type="search" style="width: 99%;" />
				<NcEmptyContent
					name="Search"
					description="Start typing to search">
					<template #icon>
						<Magnify />
					</template>
				</NcEmptyContent>
			</div>
		</NcHeaderMenu>
	</div>
</template>
<script>
import Magnify from 'vue-material-design-icons/Magnify'

export default {
	components: {
		Magnify,
	},
}
</script>
<style>
#nextcloud-header {
	display: flex;
	justify-content: right;
	background-color: var(--color-primary);
}
</style>
```
</docs>

<template>
	<component :is="wrapperTag"
		:id="id"
		v-click-outside="clickOutsideConfig"
		:aria-labelledby="isNav ? triggerId : null"
		:class="{ 'header-menu--opened': opened }"
		class="header-menu">
		<!-- Trigger -->
		<button :id="triggerId"
			ref="trigger"
			class="header-menu__trigger button-vue"
			:aria-label="ariaLabel"
			:aria-describedby="description ? descriptionId : null"
			:aria-controls="`header-menu-${id}`"
			:aria-expanded="opened.toString()"
			@click.prevent="toggleMenu">
			<!-- @slot Icon trigger slot. Make sure the svg path
				is at least 16px. Usually mdi icon works at 20px -->
			<slot name="trigger" />
		</button>

		<span v-if="description"
			:id="descriptionId"
			class="header-menu__description hidden-visually">
			{{ description }}
		</span>

		<!-- Visual triangle -->
		<div v-show="opened" class="header-menu__carret" />

		<!-- Menu opened content -->
		<div v-show="opened"
			:id="`header-menu-${id}`"
			class="header-menu__wrapper">
			<div ref="content" class="header-menu__content">
				<!-- @slot Main content -->
				<slot />
			</div>
		</div>
	</component>
</template>

<script>
import { vOnClickOutside as ClickOutside } from '@vueuse/components'
import { createFocusTrap } from 'focus-trap'

import GenRandomId from '../../utils/GenRandomId.js'
import { clickOutsideOptions } from '../../mixins/index.js'
import { getTrapStack } from '../../utils/focusTrap.js'

export default {
	name: 'NcHeaderMenu',

	directives: {
		ClickOutside,
	},

	mixins: [
		clickOutsideOptions,
	],

	props: {
		/**
		 * Unique id for this menu
		 */
		id: {
			type: String,
			required: true,
		},

		/**
		 * aria-label attribute of the menu open button
		 */
		ariaLabel: {
			type: String,
			default: '',
		},

		/**
		 * Current menu open state
		 */
		open: {
			type: Boolean,
			default: false,
		},

		/**
		 * Pass `true` if the header menu is used for website navigation
		 *
		 * The wrapper tag will be set to `nav` and its `aria-labelledby`
		 * will be associated with the menu open button
		 */
		isNav: {
			type: Boolean,
			default: false,
		},

		/**
		 * Additional visually hidden description text for the menu
		 * open button
		 */
		description: {
			type: String,
			default: null,
		},
	},

	emits: [
		'close',
		'closed',
		'open',
		'opened',
		'update:open',
		'cancel',
	],

	data() {
		return {
			focusTrap: null,
			opened: this.open,
			shortcutsDisabled: window.OCP?.Accessibility?.disableKeyboardShortcuts?.(),
			triggerId: GenRandomId(),
			descriptionId: GenRandomId(),
		}
	},

	computed: {
		wrapperTag() {
			return this.isNav ? 'nav' : 'div'
		},

		clickOutsideConfig() {
			return [
				this.closeMenu,
				this.clickOutsideOptions,
			]
		},
	},

	watch: {
		open(open) {
			if (open) {
				this.openMenu()
			} else {
				this.closeMenu()
			}
		},
	},

	mounted() {
		document.addEventListener('keydown', this.onKeyDown)
	},
	beforeDestroy() {
		document.removeEventListener('keydown', this.onKeyDown)
	},

	methods: {
		/**
		 * Toggle the current menu open state
		 */
		toggleMenu() {
			// Toggling current state
			if (!this.opened) {
				this.openMenu()
			} else {
				this.closeMenu()
			}
		},

		/**
		 * Close the current menu
		 *
		 * @param {boolean} cancelled emit a cancel event instead of close
		 */
		closeMenu(cancelled = false) {
			// Close the menu
			this.opened = false
			this.$emit(cancelled ? 'cancel' : 'close')
			this.$emit('update:open', false)

			// Kill focus trap
			this.clearFocusTrap()

			// Wait for component to finish rendering
			this.$nextTick(() => {
				this.$emit('closed')
			})
		},

		/**
		 * Open the current menu
		 */
		openMenu() {
			// Open the menu
			this.opened = true
			this.$emit('open')
			this.$emit('update:open', true)

			// Wait for component to finish rendering
			this.$nextTick(() => {
				this.useFocusTrap()
				this.$emit('opened')
			})
		},

		onKeyDown(event) {
			if (this.shortcutsDisabled || !this.opened) {
				return
			}

			// If escape have been pressed, we close
			if (event.key === 'Escape') {
				event.preventDefault()

				/** User cancelled the menu by pressing escape */
				this.closeMenu(true)
			}
		},

		/**
		 * Add focus trap for accessibility.
		 * Shall only be used when all children are mounted
		 * and available in the DOM. We use $nextTick for that.
		 */
		async useFocusTrap() {
			if (this.focusTrap) {
				return
			}
			// Init focus trap
			const contentContainer = this.$refs.content
			this.focusTrap = createFocusTrap(contentContainer, {
				allowOutsideClick: true,
				trapStack: getTrapStack(),
				fallbackFocus: this.$refs.trigger,
			})
			this.focusTrap.activate()
		},
		clearFocusTrap() {
			this.focusTrap?.deactivate()
			this.focusTrap = null
		},
	},
}
</script>

<style lang="scss" scoped>
// content inner and outer margin
// Also used for menu top-right positioning
$externalMargin: 8px;

.header-menu {
	position: relative;
	width: var(--header-height);
	height: var(--header-height);

	&__trigger {
		display: flex;
		align-items: center;
		justify-content: center;
		width: var(--header-height);
		height: var(--header-height);
		margin: 0;
		padding: 0;
		cursor: pointer;
		opacity: .85;
		background-color: transparent;
		border: none;

		// header is filled with primary or image background
		filter: none !important;
		color: var(--color-primary-text) !important;
	}

	&--opened &__trigger,
	&__trigger:hover,
	&__trigger:focus,
	&__trigger:active {
		opacity: 1;
	}

	&__trigger:focus-visible {
		outline: none;
	}

	&__wrapper {
		position: fixed;
		z-index: 2000;
		top: 50px;
		right: 0;
		box-sizing: border-box;
		margin: 0 $externalMargin;
		padding: 8px;
		border-radius: 0 0 var(--border-radius) var(--border-radius);
		border-radius: var(--border-radius-large);
		background-color: var(--color-main-background);

		filter: drop-shadow(0 1px 5px var(--color-box-shadow));
	}

	&__carret {
		position: absolute;
		z-index: 2001; // Because __wrapper is 2000.
		bottom: 0;
		left: calc(50% - 10px);
		width: 0;
		height: 0;
		content: ' ';
		pointer-events: none;
		border: 10px solid transparent;
		border-bottom-color: var(--color-main-background);
	}

	&__content {
		overflow: auto;
		width: 350px;
		max-width: calc(100vw - 2 * $externalMargin);
		min-height: calc(44px * 1.5);
		max-height: calc(100vh - 50px * 2);
		:deep(.empty-content) {
			margin: 12vh 10px;
		}
	}
}

@media only screen and (max-width: math.div($breakpoint-mobile, 2)) {
	.header-menu {
		width: $clickable-area;

		&__trigger {
			width: $clickable-area;
		}
	}
}
</style>
