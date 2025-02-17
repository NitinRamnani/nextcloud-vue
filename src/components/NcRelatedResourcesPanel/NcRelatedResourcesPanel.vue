<!--
  - @copyright 2022 Christopher Ng <chrng8@gmail.com>
  -
  - @author Christopher Ng <chrng8@gmail.com>
  -
  - @license AGPL-3.0-or-later
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
### Usage

Use this component to display the related resources of a given item.

```
<template>
	<NcRelatedResourcesPanel provider-id="talk"
		:item-id="conversationId" />
</template>

<script>
export default {
	data() {
		return {
			conversationId: 1,
		}
	},
}
</script>
```
</docs>

<template>
	<div v-if="appEnabled && isVisible" class="related-resources">
		<div class="related-resources__header">
			<h5>{{ headerTranslated }}</h5>
			<p>{{ description }}</p>
		</div>

		<NcResource v-for="resource in resources"
			:key="resource.itemId"
			class="related-resources__entry"
			:icon="resource.icon"
			:name="resource.title"
			:url="resource.url" />
	</div>
</template>

<script>
import axios from '@nextcloud/axios'
import { generateOcsUrl } from '@nextcloud/router'

import NcResource from './NcResource.vue'

import { t } from '../../l10n.js'

export default {
	name: 'NcRelatedResourcesPanel',

	components: {
		NcResource,
	},

	props: {
		/**
		 * The provider id implemented with `\OCA\RelatedResources\IRelatedResourceProvider::getProviderId()`
		 */
		providerId: {
			type: String,
			default: null,
		},
		/**
		 * The item id which uniquely identities the e.g. Calendar event, Deck board, file, Talk room, etc.
		 */
		itemId: {
			type: [String, Number],
			default: null,
		},
		/**
		 * Only used by the files sidebar
		 *
		 * File info is passed when registered with `OCA.Sharing.ShareTabSections.registerSection()`
		 */
		fileInfo: {
			type: Object,
			default: null,
		},
	},

	emits: [
		'has-error',
		'has-resources',
	],

	data() {
		return {
			appEnabled: OC?.appswebroots?.related_resources !== undefined,
			headerTranslated: t('Related resources'),
			loading: false,
			error: null,
			resources: [],
		}
	},

	computed: {
		isVisible() {
			if (this.loading) {
				return false
			}
			return this.error ?? this.resources.length > 0
		},

		description() {
			if (this.error) {
				return t('Error getting related resources. Please contact your system administrator if you have any questions.')
			}
			return t('Anything shared with the same group of people will show up here')
		},

		hasResourceInfo() {
			if (this.providerId !== null && this.itemId !== null) {
				return true
			}
			if (this.fileInfo !== null) {
				return true
			}
			return false
		},

		isFiles() {
			return this.fileInfo?.id !== undefined
		},

		url() {
			let providerId = null
			let itemId = null

			if (this.isFiles) {
				providerId = 'files'
				itemId = this.fileInfo.id
			} else {
				providerId = this.providerId
				itemId = this.itemId
			}

			return generateOcsUrl('/apps/related_resources/related/{providerId}?itemId={itemId}&format=json', {
				providerId,
				itemId,
			})
		},
	},

	watch: {
		providerId() {
			this.fetchRelatedResources()
		},
		itemId() {
			this.fetchRelatedResources()
		},
		fileInfo() {
			this.fetchRelatedResources()
		},
		error(error) {
			/**
			 * Emitted when the error value changes
			 *
			 * @type {boolean}
			 */
			this.$emit('has-error', Boolean(error))
		},
		resources(resources) {
			/**
			 * Emitted when the resources value changes
			 *
			 * @type {boolean}
			 */
			this.$emit('has-resources', resources.length > 0)
		},
	},

	created() {
		this.fetchRelatedResources()
	},

	methods: {
		t,
		async fetchRelatedResources() {
			if (!this.appEnabled || !this.hasResourceInfo) {
				return
			}

			this.loading = true
			this.error = null
			this.resources = []
			try {
				const response = await axios.get(this.url)
				this.resources = response.data.ocs?.data
			} catch (e) {
				this.error = e
				console.error(e)
			} finally {
				this.loading = false
			}
		},
	},
}
</script>

<style lang="scss" scoped>
.related-resources {
	&__header {
		margin: 0 0 10px 46px;

		h5 {
			font-weight: bold;
		}

		p {
			color: var(--color-text-maxcontrast);
		}
	}
}
</style>
