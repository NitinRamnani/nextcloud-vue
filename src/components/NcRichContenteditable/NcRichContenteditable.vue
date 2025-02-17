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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
-->

<docs>

### General description

This component displays contenteditable div with automated `@` [at] autocompletion and `:` [colon] emoji autocompletion.

### Examples

Try mentioning user @Test01 or inserting emoji :smile

```vue
<template>
	<div>
		<NcRichContenteditable
			:value.sync="message"
			:auto-complete="autoComplete"
			:maxlength="100"
			:user-data="userData"
			@submit="onSubmit" />
		<br>

		<NcRichContenteditable
			:value.sync="message"
			:auto-complete="autoComplete"
			:maxlength="400"
			:multiline="true"
			:user-data="userData"
			@submit="onSubmit" />

		<h5>Output - raw</h5>
		{{ JSON.stringify(message) }}

		<h5>Output - preformatted</h5>
		<p class="pre-line">{{ message }}</p>

		<h5>Output - in NcRichText with markdown support</h5>
		<NcRichText :text="text" :arguments="userMentions" autolink use-markdown />
	</div>
</template>
<script>
export default {
	data() {
		return {
			message: '**Lorem ipsum** dolor sit amet.',
			// You need to provide this for the inline mention to understand what to display or not.
			// Key should be a string with leading '@', like @Test02 or @"Test Offline"
			userData: {
				Test01: {
					icon: 'icon-user',
					id: 'Test01',
					title: 'Test01',
					source: 'users',
					primary: true,
				},
				Test02: {
					icon: 'icon-user',
					id: 'Test02',
					title: 'Test02',
					source: 'users',
					status: {
						clearAt: null,
						icon: '🎡',
						message: 'Visiting London',
						status: 'away',
					},
					subline: 'Visiting London',
				},
				'Test@User': {
					icon: 'icon-user',
					id: 'Test@User',
					title: 'Test 03',
					source: 'users',
					status: {
						clearAt: null,
						icon: '🎡',
						message: 'Having space in my name',
						status: 'online',
					},
					subline: 'Visiting London',
				},
				'Test Offline': {
					icon: 'icon-user',
					id: 'Test Offline',
					title: 'Test Offline',
					source: 'users',
					status: {
						clearAt: null,
						icon: null,
						message: null,
						status: 'offline',
					},
					subline: null,
				},
				'Test DND': {
					icon: 'icon-user',
					id: 'Test DND',
					title: 'Test DND',
					source: 'users',
					status: {
						clearAt: null,
						icon: null,
						message: 'Out sick',
						status: 'dnd',
					},
					subline: 'Out sick',
				},
			},
			// To display user bubbles in NcRichText, special format of data should be provided:
			// Key should be in curly brackets without '@' and ' ' symbols, like {user-2}
			userMentions: {
				'user-1': {
					component: 'NcUserBubble',
					props: {
						displayName: 'Test01',
						user: 'Test01',
						primary: true,
					},
				},
				'user-2': {
					component: 'NcUserBubble',
					props: {
						displayName: 'Test02',
						user: 'Test02',
					},
				},
				'user-3': {
					component: 'NcUserBubble',
					props: {
						displayName: 'Test 03',
						user: 'Test@User',
					},
				},
				'user-4': {
					component: 'NcUserBubble',
					props: {
						displayName: 'Test Offline',
						user: 'Test Offline',
					},
				},
				'user-5': {
					component: 'NcUserBubble',
					props: {
						displayName: 'Test DND',
						user: 'Test DND',
					},
				},
			}
		}
	},
	computed: {
		text() {
			return this.message
					.replace('@Test01', '{user-1}')
					.replace('@Test02', '{user-2}')
					.replace('@Test@User', '{user-3}')
					.replace('@"Test Offline"', '{user-4}')
					.replace('@"Test DND"', '{user-5}')
		},
	},
	methods: {
		/**
		* Do your own query to the autocompletion api.
		* The returned data bellow is a fake data example.
		* The callback expects the same format returned by the core/autocomplete/get ocs api endpoint!
		* @see userData example above
		*
		* @param {string} search the query
		* @param {Function} callback the callback to process the results with
		*/
		autoComplete(search, callback) {
			callback(Object.values(this.userData))
		},
		onSubmit() {
			alert(this.message)
		}
	}
}
</script>
<style lang="scss" scoped>
	h5 {
		font-weight: bold;
		margin: 40px 0 20px 0;
	}

	.pre-line {
		white-space: pre-line;
	}
</style>
```

</docs>

<template>
	<div ref="contenteditable"
		v-tooltip="tooltipString"
		:class="{
			'rich-contenteditable__input--empty': isEmptyValue,
			'rich-contenteditable__input--multiline': multiline,
			'rich-contenteditable__input--overflow': isOverMaxlength,
			'rich-contenteditable__input--disabled': disabled,
		}"
		:contenteditable="canEdit"
		:placeholder="placeholder"
		:aria-placeholder="placeholder"
		aria-multiline="true"
		class="rich-contenteditable__input"
		role="textbox"
		v-on="listeners"
		@input="onInput"
		@compositionstart="isComposing = true"
		@compositionend="isComposing = false"
		@keydown.delete="onDelete"
		@keydown.enter.exact="onEnter"
		@keydown.ctrl.enter.exact.stop.prevent="onCtrlEnter"
		@paste="onPaste"
		@keyup.stop.prevent.capture="onKeyUp" />
</template>

<script>
import { t } from '../../l10n.js'
import NcAutoCompleteResult from './NcAutoCompleteResult.vue'
import richEditor from '../../mixins/richEditor/index.js'
import Tooltip from '../../directives/Tooltip/index.js'
import { emojiSearch, emojiAddRecent } from '../../functions/emoji/index.js'
import { searchProvider, getLinkWithPicker } from '../NcRichText/index.js'

import Tribute from 'tributejs/dist/tribute.esm.js'
import debounce from 'debounce'
import stringLength from 'string-length'

export default {
	name: 'NcRichContenteditable',

	directives: {
		tooltip: Tooltip,
	},

	mixins: [richEditor],

	props: {
		value: {
			type: String,
			default: '',
			required: true,
		},

		placeholder: {
			type: String,
			default: t('Write a message …'),
		},

		autoComplete: {
			type: Function,
			default: () => [],
		},

		menuContainer: {
			type: Element,
			default: () => document.body,
		},

		/**
		 * Make the contenteditable looks like a textarea or not.
		 * Default looks like a single-line input.
		 * This also handle the default enter/shift+enter behaviour.
		 * if multiline, enter = newline; otherwise enter = submit
		 * shift+enter always add a new line. ctrl+enter always submits
		 */
		multiline: {
			type: Boolean,
			default: false,
		},

		/**
		 * Is the content editable ?
		 */
		contenteditable: {
			type: Boolean,
			default: true,
		},

		/**
		 * Disable the editing and show specific disabled design
		 */
		disabled: {
			type: Boolean,
			default: false,
		},

		/**
		 * Max allowed length
		 */
		maxlength: {
			type: Number,
			default: null,
		},

		/**
		 * Enable or disable emoji autocompletion
		 */
		emojiAutocomplete: {
			type: Boolean,
			default: true,
		},

		/**
		 * Enable or disable link autocompletion
		 */
		linkAutocomplete: {
			type: Boolean,
			default: true,
		},
	},

	emits: [
		'submit',
		'paste',
		'update:value',
	],

	data() {
		return {
			textSmiles: [],
			tribute: null,
			autocompleteOptions: {
				// Allow spaces in the middle of mentions
				allowSpaces: true,
				fillAttr: 'id',
				// Search against id and title (display name)
				lookup: result => `${result.id} ${result.title}`,
				// Where to inject the menu popup
				menuContainer: this.menuContainer,
				// Popup mention autocompletion templates
				menuItemTemplate: item => this.renderComponentHtml(item.original, NcAutoCompleteResult),
				// Hide if no results
				noMatchTemplate: () => '<span class="hidden"></span>',
				// Inner display of mentions
				selectTemplate: item => this.genSelectTemplate(item?.original?.id),
				// Autocompletion results
				values: this.debouncedAutoComplete,
			},
			emojiOptions: {
				trigger: ':',
				// Don't use the tribute search function at all
				// We pass search results as values (see below)
				lookup: (result, query) => query,
				// Where to inject the menu popup
				menuContainer: this.menuContainer,
				// Popup mention autocompletion templates
				menuItemTemplate: item => {
					if (this.textSmiles.includes(item.original)) {
						// Display the raw text string for :), :-D, … for non emoji results,
						// instead of trying to show an image and their name.
						return item.original
					}

					return `<span class="tribute-container-emoji__item__emoji">${item.original.native}</span> :${item.original.short_name}`
				},
				// Hide if no results
				noMatchTemplate: () => t('No emoji found'),
				// Display raw emoji along with its name
				selectTemplate: (item) => {
					if (this.textSmiles.includes(item.original)) {
						// Replace the selection with the raw text string for :), :-D, … for non emoji results
						return item.original
					}

					emojiAddRecent(item.original)
					return item.original.native
				},
				// Pass the search results as values
				values: (text, cb) => {
					const emojiResults = emojiSearch(text)
					if (this.textSmiles.includes(':' + text)) {
						/**
						 * Prepend text smiles to the search results so that Tribute
						 * is not interfering with normal writing, aka. "Cocos Island Meme".
						 * E.g. `:)` and `:-)` got replaced by the flag of Cocos Island,
						 * when submitting the input with Enter after writing them
						 */
						emojiResults.unshift(':' + text)
					}
					cb(emojiResults)
				},
				// Class added to the menu container
				containerClass: 'tribute-container-emoji',
				// Class added to each list item
				itemClass: 'tribute-container-emoji__item',
			},
			linkOptions: {
				trigger: '/',
				// Don't use the tribute search function at all
				// We pass search results as values (see below)
				lookup: (result, query) => query,
				// Where to inject the menu popup
				menuContainer: this.menuContainer,
				// Popup mention autocompletion templates
				menuItemTemplate: item => `<img class="tribute-container-link__item__icon" src="${item.original.icon_url}"> <span class="tribute-container-link__item__title">${item.original.title}</span>`,
				// Hide if no results
				noMatchTemplate: () => t('No link provider found'),
				selectTemplate: this.getLink,
				// Pass the search results as values
				values: (text, cb) => cb(searchProvider(text)),
				// Class added to the menu container
				containerClass: 'tribute-container-link',
				// Class added to each list item
				itemClass: 'tribute-container-link__item',
			},

			// Represent the raw untrimmed text of the contenteditable
			// serves no other purpose than to check whether the
			// content is empty or not
			localValue: this.value,

			// Is in text composition session in IME
			isComposing: false,
		}
	},

	computed: {
		/**
		 * Is the current trimmed value empty?
		 *
		 * @return {boolean}
		 */
		isEmptyValue() {
			return !this.localValue
				|| (this.localValue && this.localValue.trim() === '')
		},

		/**
		 * Is this Firefox? 🙄
		 *
		 * @return {boolean}
		 */
		isFF() {
			return !!navigator.userAgent.match(/firefox/i)
		},

		/**
		 * Is the current value over maxlength?
		 *
		 * @return {boolean}
		 */
		isOverMaxlength() {
			if (this.isEmptyValue || !this.maxlength) {
				return false
			}
			return stringLength(this.localValue) > this.maxlength
		},

		/**
		 * Tooltip to show if characters count is over limit
		 *
		 * @return {string}
		 */
		tooltipString() {
			if (!this.isOverMaxlength) {
				return null
			}
			return {
				content: t('Message limit of {count} characters reached', { count: this.maxlength }),
				shown: true,
				trigger: 'manual',
			}
		},

		/**
		 * Edit is only allowed when contenteditableis true and disabled is false
		 *
		 * @return {boolean}
		 */
		canEdit() {
			return this.contenteditable && !this.disabled
		},

		/**
		 * Proxied native event handlers without custom event handlers
		 *
		 * @return {Record<string, Function>}
		 */
		listeners() {
			/**
			 * All component's event handlers are set as native event handlers with by v-on directive.
			 * The component also raised custom events manually by $emit for corresponding events.
			 * As a result, it triggers handlers twice.
			 * The v-on="listeners" directive should only set proxied native events handler without custom events
			 */
			const listeners = { ...this.$listeners }
			delete listeners.paste
			return listeners
		},
	},

	watch: {
		/**
		 * If the parent value change, we compare the plain text rendering
		 * If it's different, we render everything and update the main content
		 */
		value() {
			const html = this.$refs.contenteditable.innerHTML
			// Compare trimmed versions to be safe
			if (this.value.trim() !== this.parseContent(html).trim()) {
				this.updateContent(this.value)
			}
		},
	},

	mounted() {
		/**
		 * Populate the list of text smiles we want to offer via Tribute.
		 * We add the colon `:)` and colon-dash `:-)` version for each of them.
		 */
		const smilesCharacters = ['d', 'D', 'p', 'P', 's', 'S', 'x', 'X', ')', '(', '|', '/']
		this.textSmiles = []
		smilesCharacters.forEach((char) => {
			this.textSmiles.push(':' + char)
			this.textSmiles.push(':-' + char)
		})

		this.autocompleteTribute = new Tribute(this.autocompleteOptions)
		this.autocompleteTribute.attach(this.$el)

		if (this.emojiAutocomplete) {
			this.emojiTribute = new Tribute(this.emojiOptions)
			this.emojiTribute.attach(this.$el)
		}

		if (this.linkAutocomplete) {
			this.linkTribute = new Tribute(this.linkOptions)
			this.linkTribute.attach(this.$el)
		}

		// Update default value
		this.updateContent(this.value)

		// Removes the contenteditable attribute at first load if the prop is
		// set to false.
		this.$refs.contenteditable.contentEditable = this.canEdit
	},
	beforeDestroy() {
		if (this.autocompleteTribute) {
			this.autocompleteTribute.detach(this.$el)
		}
		if (this.emojiTribute) {
			this.emojiTribute.detach(this.$el)
		}
		if (this.linkTribute) {
			this.linkTribute.detach(this.$el)
		}
	},

	methods: {
		/**
		 * Focus the richContenteditable
		 *
		 * @public
		 */
		focus() {
			this.$refs.contenteditable.focus()
		},

		getLink(item) {
			// there is no way to get a tribute result asynchronously
			// so we immediately insert a node and replace it when the result comes
			getLinkWithPicker(item.original.id)
				.then(link => {
					// replace dummy temp element by a text node which contains the link
					const tmpElem = document.getElementById('tmp-link-result-node')
					const newElem = document.createTextNode(link)
					tmpElem.replaceWith(newElem)
					this.setCursorAfter(newElem)
					this.updateValue(this.$refs.contenteditable.innerHTML)
				})
				.catch((error) => {
					console.debug('Smart picker promise rejected:', error)
					const tmpElem = document.getElementById('tmp-link-result-node')
					this.setCursorAfter(tmpElem)
					tmpElem.remove()
				})
			return '<span id="tmp-link-result-node"></span>'
		},
		setCursorAfter(element) {
			const range = document.createRange()
			range.setEndAfter(element)
			range.collapse()
			const selection = window.getSelection()
			selection.removeAllRanges()
			selection.addRange(range)
		},
		/**
		 * Re-emit the input event to the parent
		 *
		 * @param {Event} event the input event
		 */
		onInput(event) {
			this.updateValue(event.target.innerHTML)
		},

		/**
		 * When pasting, sanitize the content, extract text
		 * and render it again
		 *
		 * @param {Event} event the paste event
		 * @fires Event paste the original paste event
		 */
		onPaste(event) {
			// Either disabled or edit deactivated
			if (!this.canEdit) {
				return
			}

			event.preventDefault()
			const clipboardData = event.clipboardData

			/** The original paste event */
			this.$emit('paste', event)

			// If we have a file or if we don't have any text, ignore
			if (clipboardData.files.length !== 0
				|| !Object.values(clipboardData.items).find(item => item?.type.startsWith('text'))) {
				return
			}

			const text = clipboardData.getData('text')
			const selection = window.getSelection()

			// If no selection, replace the whole data
			if (!selection.rangeCount) {
				this.updateValue(text)
				return
			}

			// Generate text and insert
			const range = selection.getRangeAt(0)
			selection.deleteFromDocument()
			range.insertNode(document.createTextNode(text))

			// Put cursor at the end of the selection
			const newRange = document.createRange()
			newRange.setStart(event.target, range.endOffset)
			newRange.collapse(true)
			selection.removeAllRanges()
			selection.addRange(newRange)

			// Propagate data
			this.updateValue(this.$refs.contenteditable.innerHTML)
		},

		/**
		 * Update the value text from the provided html
		 *
		 * @param {string} htmlOrText the html content (or raw text with @mentions)
		 */
		updateValue(htmlOrText) {
			const text = this.parseContent(htmlOrText)
			this.localValue = text
			this.$emit('update:value', text)
		},

		/**
		 * Update content and local value
		 *
		 * @param {string} value the message value
		 */
		updateContent(value) {
			const renderedContent = this.renderContent(value)
			this.$refs.contenteditable.innerHTML = renderedContent
			this.localValue = value
		},

		/**
		 * Because FF have a decade old bug preventing contenteditable=false
		 * to properly be deleted on backspace, we have to hack 👀
		 * https://stackoverflow.com/a/59383394/3885878
		 * https://stackoverflow.com/a/30574622
		 *
		 * @param {Event} event the delete keydown event
		 */
		onDelete(event) {
			if (!this.isFF || !window.getSelection) {
				return
			}

			// Either disabled or edit deactivated
			if (!this.canEdit) {
				return
			}

			// fix backspace bug in FF
			// https://bugzilla.mozilla.org/show_bug.cgi?id=685445
			// https://bugzilla.mozilla.org/show_bug.cgi?id=1665167
			const selection = window.getSelection()
			const node = event.target
			if (!selection.isCollapsed || !selection.rangeCount) {
				return
			}

			const curRange = selection.getRangeAt(selection.rangeCount - 1)
			if (curRange.commonAncestorContainer.nodeType === 3 && curRange.startOffset > 0) {
				// we are in child selection. The characters of the text node is being deleted
				return
			}

			const range = document.createRange()
			if (selection.anchorNode !== node) {
				// selection is in character mode. expand it to the whole editable field
				range.selectNodeContents(node)
				range.setEndBefore(selection.anchorNode)
			} else if (selection.anchorOffset > 0) {
				range.setEnd(node, selection.anchorOffset)
			} else {
				// reached the beginning of editable field
				return
			}
			range.setStart(node, range.endOffset - 1)

			const previousNode = range.cloneContents().lastChild
			if (previousNode && previousNode.contentEditable === 'false') {
				// this is some rich content, e.g. smile. We should help the user to delete it
				range.deleteContents()
				event.preventDefault()
			}
		},
		/**
		 * Enter key pressed. Submits if not multiline
		 *
		 * @param {Event} event the keydown event
		 */
		onEnter(event) {
			// Prevent submitting if multiline
			// or length is over maxlength
			// or autocompletion menu is opened
			// or in a text composition session with IME
			if (this.multiline
				|| this.isOverMaxlength
				|| this.autocompleteTribute.isActive || this.emojiTribute.isActive || this.linkTribute.isActive
				|| this.isComposing) {
				return
			}

			event.preventDefault()
			event.stopPropagation()
			this.$emit('submit', event)
		},

		/**
		 * Ctrl + Enter key pressed is used to submit
		 *
		 * @param {Event} event the keydown event
		 */
		onCtrlEnter(event) {
			if (this.isOverMaxlength) {
				return
			}
			this.$emit('submit', event)
		},

		/**
		 * Debounce the autocomplete function
		 */
		debouncedAutoComplete: debounce(async function(search, callback) {
			this.autoComplete(search, callback)
		}, 100),

		onKeyUp(event) {
			// prevent tribute from opening on keyup
			event.stopImmediatePropagation()
		},
	},
}
</script>

<style lang="scss" scoped>
// Standalone styling, independent from server
.rich-contenteditable__input {
	overflow-y: auto;
	width: auto;
	margin: 0;
	padding: 8px;
	cursor: text;
	white-space: pre-wrap;
	word-break: break-word;
	color: var(--color-main-text);
	border: 2px solid var(--color-border-dark);
	border-radius: var(--border-radius-large);
	outline: none;
	background-color: var(--color-main-background);
	font-family: var(--font-face);
	font-size: inherit;
	min-height: $clickable-area;
	max-height: $clickable-area * 5.5;

	// Cannot use :empty because of firefox bug https://bugzilla.mozilla.org/show_bug.cgi?id=1513303
	&--empty:before {
		content: attr(placeholder);
		color: var(--color-text-maxcontrast);
		position: absolute;
	}

	&[contenteditable='false']:not(&--disabled) {
		cursor: default;
		background-color: transparent;
		color: var(--color-main-text);
		border-color: transparent;
		opacity: 1;
		border-radius: 0;
	}

	&--multiline {
		min-height: $clickable-area * 3;
		// No max for mutiline
		max-height: none;
	}

	&--disabled {
		opacity: $opacity_disabled;
		color: var(--color-text-maxcontrast);
		border: 2px solid var(--color-background-darker);
		border-radius: var(--border-radius);
		background-color: var(--color-background-dark);
	}
}

</style>

<style lang="scss">
.tribute-container, .tribute-container-emoji, .tribute-container-link {
	z-index: 9000;
	overflow: auto;
	min-width: 250px;
	max-width: 300px;
	// Show maximum 4 entries and a half to show scroll
	// 44px + 10px padding
	max-height: ($clickable-area + 20px) * 4.5;
	// Space it out a bit from the text
	margin: 5px 0;
	color: var(--color-main-text);
	border-radius: var(--border-radius);
	background: var(--color-main-background);
	box-shadow: 0 1px 5px var(--color-box-shadow);
}

.tribute-container-emoji, .tribute-container-link {
	min-width: 200px;
	max-width: 200px;
	padding: 4px;
	// Show maximum 5 entries and a half to show scroll
	max-height: 35px * 5 + math.div(35px, 2) !important;

	&__item {
		border-radius: 8px;
		padding: 4px 8px;
		margin-bottom: 4px;
		opacity: 0.8;
		cursor: pointer;

		// Take care of long names
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;

		&:last-child {
			margin-bottom: 0;
		}

		&__emoji {
			padding-right: 8px;
		}
	}

	.highlight {
		opacity: 1;
		color: var(--color-primary-element-light-text);
		background: var(--color-primary-element-light);
		&, * {
			cursor: pointer;
		}
	}
}

.tribute-container-link {
	min-width: 200px;
	max-width: 300px;
	&__item {
		display: flex;
		align-items: center;
		&__title {
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}
		&__icon {
			margin: auto 0;
			width: 20px;
			height: 20px;
			object-fit: contain;
			padding-right: 8px;
			filter: var(--background-invert-if-dark);
		}
	}
}

</style>
