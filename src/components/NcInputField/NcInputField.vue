<!--
  - @copyright Copyright (c) 2022 Marco Ambrosini <marcoambrosini@pm.me>
  -
  - @author Marco Ambrosini <marcoambrosini@pm.me>
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
### Description

This component is used by the other Fields components.
It extends and styles an HTMLInputElement.

You cannot use it as is. This is here for documentation purposes.
See the other field components.

For a list of all available props and attributes, please check the [HTMLInputElement documentation](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#attributes)

</docs>

<template>
	<div class="input-field" :class="{ 'input-field--disabled': disabled }">
		<div class="input-field__main-wrapper">
			<input v-bind="$attrs"
				:id="computedId"
				ref="input"
				class="input-field__input"
				:type="type"
				:disabled="disabled"
				:placeholder="computedPlaceholder"
				:aria-describedby="ariaDescribedby"
				aria-live="polite"
				:class="[inputClass,
					{
						'input-field__input--trailing-icon': showTrailingButton || hasTrailingIcon,
						'input-field__input--leading-icon': hasLeadingIcon,
						'input-field__input--label-outside': labelOutside,
						'input-field__input--success': success,
						'input-field__input--error': error,
					}]"
				:value="value"
				v-on="$listeners"
				@input="handleInput">
			<!-- Label -->
			<label v-if="!labelOutside && isValidLabel"
				class="input-field__label"
				:class="[{
					'input-field__label--trailing-icon': showTrailingButton || hasTrailingIcon,
					'input-field__label--leading-icon': hasLeadingIcon,
				}]"
				:for="computedId">
				{{ label }}
			</label>

			<!-- Leading icon -->
			<div v-show="hasLeadingIcon" class="input-field__icon input-field__icon--leading">
				<!-- Leading material design icon in the text field, set the size to 18 -->
				<slot />
			</div>

			<!-- trailing button -->
			<NcButton v-if="showTrailingButton"
				type="tertiary-no-background"
				class="input-field__clear-button"
				:aria-label="trailingButtonLabel"
				:disabled="disabled"
				@click="handleTrailingButtonClick">
				<!-- Populating this slot creates a trailing button within the
				input boundaries that emits a `trailing-button-click` event -->
				<template #icon>
					<slot name="trailing-button-icon" />
				</template>
			</NcButton>

			<!-- Success and error icons -->
			<div v-else-if="success || error"
				class="input-field__icon input-field__icon--trailing">
				<Check v-if="success" :size="20" style="color: var(--color-success-text);" />
				<AlertCircle v-else-if="error" :size="20" style="color: var(--color-error-text);" />
			</div>
		</div>
		<p v-if="helperText.length > 0"
			:id="`${inputName}-helper-text`"
			class="input-field__helper-text-message"
			:class="{
				'input-field__helper-text-message--error': error,
				'input-field__helper-text-message--success': success,
			}">
			<Check v-if="success" class="input-field__helper-text-message__icon" :size="18" />
			<AlertCircle v-else-if="error" class="input-field__helper-text-message__icon" :size="18" />
			{{ helperText }}
		</p>
	</div>
</template>

<script>
import NcButton from '../NcButton/index.js'
import GenRandomId from '../../utils/GenRandomId.js'

import AlertCircle from 'vue-material-design-icons/AlertCircleOutline.vue'
import Check from 'vue-material-design-icons/Check.vue'

export default {
	name: 'NcInputField',

	components: {
		NcButton,
		AlertCircle,
		Check,
	},

	inheritAttrs: false,

	props: {
		/**
		 * The value of the input field
		 */
		value: {
			type: String,
			required: true,
		},

		/**
		 * The type of the input element
		 */
		type: {
			type: String,
			default: 'text',
			validator: (value) => [
				'text',
				'password',
				'email',
				'tel',
				'url',
				'search',
				'number',
			].includes(value),
		},

		/**
		 * The input label, always provide one for accessibility purposes.
		 * This will also be used as a placeholder unless the placeholder
		 * prop is populated with a different string.
		 */
		label: {
			type: String,
			default: undefined,
		},

		/**
		 * Pass in true if you want to use an external label. This is useful
		 * if you need a label that looks different from the one provided by
		 * this component
		 */
		labelOutside: {
			type: Boolean,
			default: false,
		},

		/**
		 * The placeholder of the input. This defaults as the string that's
		 * passed into the label prop. In order to remove the placeholder,
		 * pass in an empty string.
		 */
		placeholder: {
			type: String,
			default: undefined,
		},

		/**
		 * Controls whether to display the trailing button.
		 */
		showTrailingButton: {
			type: Boolean,
			default: false,
		},

		/**
		 * Label of the trailing button
		 *
		 * Required when showTrailingButton is set
		 */
		trailingButtonLabel: {
			type: String,
			default: '',
		},

		/**
		 * Toggles the success state of the component. Adds a checkmark icon.
		 * this cannot be used together with canClear.
		 */
		success: {
			type: Boolean,
			default: false,
		},

		/**
		 * Toggles the error state of the component. Adds an error icon.
		 * this cannot be used together with canClear.
		 */
		error: {
			type: Boolean,
			default: false,
		},

		/**
		 * Additional helper text message
		 *
		 * This will be displayed beneath the input field. In case the field is
		 * also marked as having an error, the text will be displayed in red.
		 */
		helperText: {
			type: String,
			default: '',
		},

		/**
		 * Disable the input field
		 */
		disabled: {
			type: Boolean,
			default: false,
		},
		/**
		 * Class to add to the input field.
		 * Necessary to use NcInputField in the NcActionInput component.
		 */
		inputClass: {
			type: [Object, String],
			default: '',
		},
	},

	emits: [
		'update:value',
		'trailing-button-click',
	],

	computed: {
		computedId() {
			return this.$attrs.id && this.$attrs.id !== '' ? this.$attrs.id : this.inputName
		},

		inputName() {
			return 'input' + GenRandomId()
		},

		hasLeadingIcon() {
			return this.$slots.default
		},

		hasTrailingIcon() {
			return this.success
		},

		hasPlaceholder() {
			return this.placeholder !== '' && this.placeholder !== undefined
		},

		computedPlaceholder() {
			return this.hasPlaceholder ? this.placeholder : this.label
		},

		isValidLabel() {
			const isValidLabel = this.label || this.labelOutside
			if (!isValidLabel) {
				console.warn('You need to add a label to the NcInputField component. Either use the prop label or use an external one, as per the example in the documentation.')
			}
			return isValidLabel
		},

		ariaDescribedby() {
			const ariaDescribedby = []
			if (this.helperText.length > 0) {
				ariaDescribedby.push(`${this.inputName}-helper-text`)
			}
			if (this.$attrs['aria-describedby']) {
				ariaDescribedby.push(this.$attrs['aria-describedby'])
			}
			return ariaDescribedby.join(' ') || null
		},
	},

	methods: {
		/**
		 * Focus the input element
		 *
		 * @public
		 */
		focus() {
			this.$refs.input.focus()
		},

		/**
		 * Select all the text in the input
		 *
		 * @public
		 */
		select() {
			this.$refs.input.select()
		},

		handleInput(event) {
			this.$emit('update:value', event.target.value)
		},

		handleTrailingButtonClick(event) {
			this.$emit('trailing-button-click', event)
		},
	},
}
</script>

<style lang="scss" scoped>

.input-field {
	position: relative;
	width: 100%;
	border-radius: var(--border-radius-large);
	margin-block-start: 6px; // for the label in active state

	&__main-wrapper {
		height: 38px; // 44px - 6px margin
		position: relative;
	}

	&--disabled {
		opacity: 0.7;
		filter: saturate(0.7);
	}

	&__input {
		margin: 0;
		padding-inline: 10px 6px; // align with label 8px margin label + 4px padding label - 2px border input
		height: 38px !important;
		width: 100%;

		font-size: var(--default-font-size);
		text-overflow: ellipsis;

		background-color: var(--color-main-background);
		color: var(--color-main-text);
		border: 2px solid var(--color-border-maxcontrast);
		border-radius: var(--border-radius-large);

		cursor: pointer;
		-webkit-appearance: textfield !important;
		-moz-appearance: textfield !important;

		// Center text if external label is used
		&--label-outside {
			padding-block: 0;
		}

		&:active:not([disabled]),
		&:hover:not([disabled]),
		&:focus:not([disabled]) {
			border-color: var(--color-primary-element);
		}

		// Hide placeholder while not focussed -> show label instead (only if internal label is used)
		&:not(:focus,&--label-outside)::placeholder {
			opacity: 0;
		}

		&:focus {
			cursor: text;
		}

		&:disabled {
			cursor: default;
		}

		&:focus-visible {
			box-shadow: unset !important; // Override server rules
		}

		&--leading-icon {
			padding-inline-start: 32px;
		}

		&--trailing-icon {
			padding-inline-end: 32px;
		}

		&--success {
			border-color: var(--color-success) !important; //Override hover border color
			&:focus-visible {
				box-shadow: rgb(248, 250, 252) 0px 0px 0px 2px, var(--color-primary-element) 0px 0px 0px 4px, rgba(0, 0, 0, 0.05) 0px 1px 2px 0px
			}

			// Align label text color with border color (on hover / focus)
			&:focus + .input-field__label,
			&:hover:not(:placeholder-shown) + .input-field__label {
				color: var(--color-success-text);
			}
		}

		&--error {
			border-color: var(--color-error) !important; //Override hover border color
			&:focus-visible {
				box-shadow: rgb(248, 250, 252) 0px 0px 0px 2px, var(--color-primary-element) 0px 0px 0px 4px, rgba(0, 0, 0, 0.05) 0px 1px 2px 0px
			}

			// Align label text color with border color (on hover / focus)
			&:focus + .input-field__label,
			&:hover:not(:placeholder-shown) + .input-field__label {
				color: var(--color-error-text);
			}
		}

		// Align label text color with border color (on hover / focus)
		&:not(&--success, &--error) {
			&:focus + .input-field__label,
			&:hover:not(:placeholder-shown) + .input-field__label {
				color: var(--color-primary-element);
			}
		}
	}

	&__label {
		position: absolute;
		margin-inline: 12px 0;
		// fix height and line height to center label
		height: 17px;
		max-width: fit-content;
		line-height: 1;
		inset-block-start: 12px;
		inset-inline: 0;
		// Fix color so that users do not think the input already has content
		color: var(--color-text-maxcontrast);
		// only one line labels are allowed
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		// forward events to input
		pointer-events: none;
		// Position transition
		transition: height var(--animation-quick), inset-block-start var(--animation-quick), font-size var(--animation-quick), color var(--animation-quick), background-color var(--animation-quick) var(--animation-slow);

		&--leading-icon {
			// 32px icon + 2px border
			margin-inline-start: 34px;
		}

		&--trailing-icon {
			// 32px icon + 2px border
			margin-inline-end: 34px;
		}
	}

	&__input:focus + &__label,
	&__input:not(:placeholder-shown) + &__label {
		inset-block-start: -6px;
		font-size: 13px; // minimum allowed font size for accessibility
		background-color: var(--color-main-background);
		height: 14px;
		padding-inline: 4px;
		margin-inline-start: 8px;

		transition: height var(--animation-quick), inset-block-start var(--animation-quick), font-size var(--animation-quick), color var(--animation-quick);
		&--leading-icon {
			margin-inline-start: 30px;
		}
	}

	&__icon {
		position: absolute;
		height: 32px;
		width: 32px;
		display: flex;
		align-items: center;
		justify-content: center;
		opacity: 0.7;

		&--leading {
			inset-block-end: 3px;
			inset-inline-start: 2px;
		}

		&--trailing {
			inset-block-end: 3px;
			inset-inline-end: 2px;
		}
	}

	&__clear-button.button-vue {
		position: absolute;
		inset-block-end: 3px;
		inset-inline-end: 2px;
		min-width: unset;
		min-height: unset;
		height: 32px;
		width: 32px !important;
		border-radius: var(--border-radius-large);
	}

	&__helper-text-message {
		padding-block: 4px;
		display: flex;
		align-items: center;

		&__icon {
			margin-inline-end: 8px;
		}

		&--error {
			color: var(--color-error-text);
		}

		&--success {
			color: var(--color-success-text);
		}
	}
}
</style>
