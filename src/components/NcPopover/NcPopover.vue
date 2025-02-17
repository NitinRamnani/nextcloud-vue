<!--
  - @copyright Copyright (c) 2019 Marco Ambrosini <marcoambrosini@icloud.com>
  -
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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
-->

<docs>

### General description

This component is just a wrapper for the floating-vue plugin by Akryum,
please refer to this documentation for customization:
https://github.com/Akryum/floating-vue

This components has two slots:
* 'trigger' which can be any html element and it will trigger the popover
this slot is optional since you can toggle the popover also by updating the
open prop on this component;

* a default slot that is for the content of the popover.

### Examples

#### With a `<button>` as a trigger:

```vue
<template>
	<div style="display: flex">
		<NcPopover>
			<template #trigger>
				<NcButton>I am the trigger</NcButton>
			</template>
			<template>
				<form tabindex="0" @submit.prevent>
					<h2>this is some content</h2>
					<p>
						Lorem ipsum dolor sit amet, consectetur adipiscing elit. <br/>
						Vestibulum eget placerat velit.
					</p>
					<label>
						Label element
						<input type="text" placeholder="input element"/>
					</label>
				</form>
			</template>
		</NcPopover>
	</div>
</template>
```

#### Without focus trap:

The [`focus-trap`](https://github.com/focus-trap/focus-trap) emits an error when used in a non-focusable element tree.

The prop `:focus-trap="false"` help to prevent it when the default behavior is not relevant.

```vue
<template>
	<div style="display: flex">
		<NcPopover :focus-trap="false">
			<template #trigger>
				<NcButton>Click me!</NcButton>
			</template>
			<template>
				Hi! 🚀
			</template>
		</NcPopover>
	</div>
</template>
```

#### With passing props to `floating-vue`'s `Dropdown`:

```vue
<template>
	<div style="display: flex">
		<NcPopover container="body" :popper-hide-triggers="(triggers) => [...triggers, 'click']">
			<template #trigger>
				<NcButton>I am the trigger</NcButton>
			</template>
			<template #default>
				<NcButton>Click on the button will close NcPopover</NcButton>
			</template>
		</NcPopover>
	</div>
</template>
```
</docs>

<template>
	<Dropdown ref="popover"
		:distance="10"
		:arrow-padding="10"
		v-bind="$attrs"
		:no-auto-focus="true /* Handled by the focus trap */"
		:popper-class="popoverBaseClass"
		v-on="$listeners"
		@apply-show="afterShow"
		@apply-hide="afterHide">
		<!-- This will be the popover target (for the events and position) -->
		<slot name="trigger" />
		<!-- This will be the content of the popover -->
		<template #popper>
			<slot />
		</template>
	</Dropdown>
</template>

<script>
import { Dropdown } from 'floating-vue'
import { createFocusTrap } from 'focus-trap'
import { getTrapStack } from '../../utils/focusTrap.js'

export default {
	name: 'NcPopover',

	components: {
		Dropdown,
	},

	inheritAttrs: false,

	props: {
		popoverBaseClass: {
			type: String,
			default: '',
		},
		/**
		 * Enable popover focus trap
		 */
		focusTrap: {
			type: Boolean,
			default: true,
		},
		/**
		 * Set element to return focus to after focus trap deactivation
		 *
		 * @type {import('focus-trap').FocusTargetValueOrFalse}
		 */
		setReturnFocus: {
			default: undefined,
			type: [HTMLElement, SVGElement, String, Boolean],
		},
	},

	emits: [
		'after-show',
		'after-hide',
	],

	beforeDestroy() {
		this.clearFocusTrap()
		this.clearEscapeStopPropagation()
	},

	methods: {
		/**
		 * @return {HTMLElement|undefined}
		 */
		getPopoverContentElement() {
			return this.$refs.popover?.$refs.popperContent?.$el
		},

		/**
		 * Add focus trap for accessibility.
		 */
		async useFocusTrap() {
			await this.$nextTick()

			if (!this.focusTrap) {
				return
			}

			const el = this.getPopoverContentElement()

			if (!el) {
				return
			}

			// Init focus trap
			this.$focusTrap = createFocusTrap(el, {
				// Prevents to lose focus using esc key
				// Focus will be release when popover be hide
				escapeDeactivates: false,
				allowOutsideClick: true,
				setReturnFocus: this.setReturnFocus,
				trapStack: getTrapStack(),
			})
			this.$focusTrap.activate()
		},

		/**
		 * Remove focus trap
		 *
		 * @param {object} options The configuration options for focusTrap
		 */
		clearFocusTrap(options = {}) {
			try {
				this.$focusTrap?.deactivate(options)
				this.$focusTrap = null
			} catch (err) {
				console.warn(err)
			}
		},

		/**
		 * Add stopPropagation for Escape.
		 * It prevents global Escape handling after closing popover.
		 *
		 * Manual event handling is used here instead of v-on because there is no direct access to the node.
		 * Alternative - wrap <template #popover> in a div wrapper.
		 */
		addEscapeStopPropagation() {
			const el = this.getPopoverContentElement()
			el?.addEventListener('keydown', this.stopKeydownEscapeHandler)
		},

		/**
		 * Remove stop Escape handler
		 */
		clearEscapeStopPropagation() {
			const el = this.getPopoverContentElement()
			el?.removeEventListener('keydown', this.stopKeydownEscapeHandler)
		},

		/**
		 * @param {KeyboardEvent} event - native keydown event
		 */
		stopKeydownEscapeHandler(event) {
			if (event.type === 'keydown' && event.key === 'Escape') {
				event.stopPropagation()
			}
		},

		afterShow() {
			/**
			 * Triggered after the tooltip was visually displayed.
			 *
			 * This is different from the 'show' and 'apply-show' which
			 * run earlier than this where there is no guarantee that the
			 * tooltip is already visible and in the DOM.
			 */
			this.$nextTick(() => {
				this.$emit('after-show')
				this.useFocusTrap()
				this.addEscapeStopPropagation()
			})
		},
		afterHide() {
			/**
			 * Triggered after the tooltip was visually hidden.
			 */
			this.$emit('after-hide')
			this.clearFocusTrap()
			this.clearEscapeStopPropagation()
		},
	},
}
</script>

<style lang="scss">

.resize-observer {
	position:absolute;
	top:0;
	left:0;
	z-index:-1;
	width:100%;
	height:100%;
	border:none;
	background-color:transparent;
	pointer-events:none;
	display:block;
	overflow:hidden;
	opacity:0
}

.resize-observer object {
	display:block;
	position:absolute;
	top:0;
	left:0;
	height:100%;
	width:100%;
	overflow:hidden;
	pointer-events:none;
	z-index:-1
}

$arrow-width: 10px;

.v-popper--theme-dropdown {
	&.v-popper__popper {
		z-index: 100000;
		top: 0;
		left: 0;
		display: block !important;

		filter: drop-shadow(0 1px 10px var(--color-box-shadow));

		.v-popper__inner {
			padding: 0;
			color: var(--color-main-text);
			border-radius: var(--border-radius-large);
			overflow: hidden;
			background: var(--color-main-background);
		}

		.v-popper__arrow-container {
			position: absolute;
			z-index: 1;
			width: 0;
			height: 0;
			border-style: solid;
			border-color: transparent;
			border-width: $arrow-width;
		}

		&[data-popper-placement^='top'] .v-popper__arrow-container {
			bottom: -$arrow-width;
			border-bottom-width: 0;
			border-top-color: var(--color-main-background);
		}

		&[data-popper-placement^='bottom'] .v-popper__arrow-container {
			top: -$arrow-width;
			border-top-width: 0;
			border-bottom-color: var(--color-main-background);
		}

		&[data-popper-placement^='right'] .v-popper__arrow-container {
			left: -$arrow-width;
			border-left-width: 0;
			border-right-color: var(--color-main-background);
		}

		&[data-popper-placement^='left'] .v-popper__arrow-container {
			right: -$arrow-width;
			border-right-width: 0;
			border-left-color: var(--color-main-background);
		}

		&[aria-hidden='true'] {
			visibility: hidden;
			transition: opacity var(--animation-quick), visibility var(--animation-quick);
			opacity: 0;
		}

		&[aria-hidden='false'] {
			visibility: visible;
			transition: opacity var(--animation-quick);
			opacity: 1;
		}
	}
}

</style>
