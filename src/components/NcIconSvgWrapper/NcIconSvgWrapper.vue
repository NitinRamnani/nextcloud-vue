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
### Description

Render raw SVG string icons.

### Example

```vue
<template>
	<div class="grid">
		<NcButton aria-label="Close">
			<template #icon>
				<NcIconSvgWrapper :svg="closeSvg" name="Close" />
			</template>
		</NcButton>
		<NcButton aria-label="Settings">
			<template #icon>
				<NcIconSvgWrapper :svg="cogSvg" name="Cog" />
			</template>
		</NcButton>
		<NcButton aria-label="Add">
			<template #icon>
				<NcIconSvgWrapper :svg="plusSvg" name="Plus" />
			</template>
		</NcButton>
		<NcButton aria-label="Send">
			<template #icon>
				<NcIconSvgWrapper :svg="sendSvg" name="Send" />
			</template>
		</NcButton>
		<NcButton aria-label="Star">
			<template #icon>
				<NcIconSvgWrapper :svg="starSvg" name="Star" />
			</template>
		</NcButton>
	</div>
</template>

<script>
import closeSvg from '@mdi/svg/svg/close.svg?raw'
import cogSvg from '@mdi/svg/svg/cog.svg?raw'
import plusSvg from '@mdi/svg/svg/plus.svg?raw'
import sendSvg from '@mdi/svg/svg/send.svg?raw'
import starSvg from '@mdi/svg/svg/star.svg?raw'

export default {
	data() {
		return {
			closeSvg,
			cogSvg,
			plusSvg,
			sendSvg,
			starSvg,
		}
	},
}
</script>

<style>
.grid {
	display: grid;
	grid-template-columns: repeat(5, max-content);
	gap: 10px;
}
</style>
```
</docs>

<template>
	<span class="icon-vue"
		role="img"
		:aria-hidden="!name"
		:aria-label="name"
		v-html="cleanSvg" /> <!-- eslint-disable-line vue/no-v-html -->
</template>

<script>
import DOMPurify from 'dompurify'

export default {
	name: 'NcIconSvgWrapper',

	props: {
		svg: {
			type: String,
			default: '',
		},
		name: {
			type: String,
			default: '',
		},
	},

	computed: {
		cleanSvg() {
			if (!this.svg) {
				return
			}
			return DOMPurify.sanitize(this.svg)
		},
	},
}
</script>

<style lang="scss" scoped>
.icon-vue {
	display: flex;
	justify-content: center;
	align-items: center;
	min-width: 44px;
	min-height: 44px;
	opacity: 1;

	&:deep(svg) {
		fill: currentColor;
		width: 20px;
		height: 20px;
		max-width: 20px;
		max-height: 20px;
	}
}
</style>
