<!--
 - @copyright Copyright (c) 2019 Christoph Wurst <christoph@winzerhof-wurst.at>
 -
 - @author Christoph Wurst <christoph@winzerhof-wurst.at>
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
<template>
	<NcButton v-tooltip.auto="label"
		class="app-navigation-toggle"
		type="tertiary"
		:aria-expanded="open ? 'true' : 'false'"
		:aria-label="label"
		aria-controls="app-navigation-vue"
		@click="toggleNavigation">
		<template #icon>
			<MenuOpenIcon v-if="open" :size="20" />
			<MenuIcon v-else :size="20" />
		</template>
	</NcButton>
</template>

<script>
import NcButton from '../NcButton/index.js'
import Tooltip from '../../directives/Tooltip/index.js'
import { t } from '../../l10n.js'

import MenuIcon from 'vue-material-design-icons/Menu.vue'
import MenuOpenIcon from 'vue-material-design-icons/MenuOpen.vue'

export default {
	name: 'NcAppNavigationToggle',

	directives: {
		tooltip: Tooltip,
	},

	components: {
		NcButton,
		MenuIcon,
		MenuOpenIcon,
	},

	props: {
		open: {
			type: Boolean,
			required: true,
		},
	},

	emits: ['update:open'],

	computed: {
		label() {
			return this.open ? t('Close navigation') : t('Open navigation')
		},
	},
	methods: {
		toggleNavigation() {
			this.$emit('update:open', !this.open)
		},
	},
}
</script>

<style scoped lang="scss">

button.app-navigation-toggle {
	position: absolute;
	top: $topbar-margin;
	right: - $topbar-margin;
	margin-right: - $clickable-area;
}

</style>
