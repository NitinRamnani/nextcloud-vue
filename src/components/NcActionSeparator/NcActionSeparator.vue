<!--
  - @copyright Copyright (c) 2019 Nextcloud et al.
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
  - @author Marco Ambrosini <marcoambrosini@icloud.com>
  - @author Kristof Hamann
  - @author Joas Schilling <coding@schilljs.com>
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
	<li class="action action-separator action--disabled" />
</template>

<script>
export default {
	name: 'NcActionSeparator',
}
</script>

<style lang="scss" scoped>
.action-separator {
	height: 0;
	margin: 5px 10px 5px 15px;
	border-bottom: 1px solid var(--color-border-dark);
	cursor: default;
}
</style>
