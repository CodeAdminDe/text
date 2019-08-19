<!--
  - @copyright Copyright (c) 2019 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
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

<template>
	<div id="rich-workspace" :class="{'icon-loading': editing && !ready }">
		<transition name="component-fade" mode="out-in">
			<read-only-editor v-if="content && !ready" :key="path" :content="content"
				:is-rich-editor="true"
				@click.native="edit" />
			<div v-else-if="!content && !ready" class="empty-workspace" @click="createNew">
				<i>Click to enter notes, lists or links</i>
			</div>
		</transition>
		<transition name="component-fade" mode="out-in">
			<editor-wrapper v-if="editing && file" v-show="ready" :file-id="file.id"
				:active="true"
				:mime="file.mimetype" @ready="ready=true" />
		</transition>
	</div>
</template>

<script>
import axios from 'nextcloud-axios'
import ReadOnlyEditor from '../components/ReadOnlyEditor'
import EditorWrapper from '../components/EditorWrapper'
export default {
	name: 'RichWorkspace',
	components: {
		EditorWrapper,
		ReadOnlyEditor
	},
	props: {
		path: {
			type: String,
			required: true
		}
	},
	data() {
		return {
			content: null,
			file: null,
			editing: false,
			ready: false
		}
	},
	watch: {
		path: function() {
			this.fetch().then(() => {})
		}
	},
	async mounted() {
		await this.fetch()
	},
	methods: {
		edit() {
			this.file = FileList.findFile('README.md')
			this.editing = true
		},
		async fetch() {
			const davPath = `dav/files/${OC.getCurrentUser().uid}/${this.path}/README.md`
			this.content = localStorage.getItem('richworkspace/' + davPath)
			let data = null
			try {
				const response = await axios.get(OC.linkToRemote(davPath))
				data = response.data ? response.data : ''
			} catch (e) {
				console.debug('No README found for ' + this.path)
			}
			localStorage.setItem('richworkspace/' + davPath, data)
			this.$nextTick(() => {
				this.content = data
				this.editing = false
				this.ready = false
			})
		},
		createNew() {
			window.FileList.createFile('README.md').then((status, data) => {
				this.fetch().then(() => {})
			})
		}
	}
}
</script>

<style scoped>
	#rich-workspace {
		padding: 20px;
	}
	.empty-workspace {
		opacity: 0.7;
	}
	#rich-workspace::v-deep div[contenteditable=false] {
		width: 100%;
		padding: 0px;
		background-color: var(--color-main-background);
		opacity: 1;
		border: none;
	}

	#rich-workspace::v-deep #editor-container {
		height: 100%;
		position: unset !important;
	}

	#rich-workspace::v-deep #editor-wrapper {
		position: unset !important;
	}

	#rich-workspace::v-deep #editor-wrapper .ProseMirror {
		padding: 0px;
		margin: 0;
	}

	#rich-workspace::v-deep .menubar .menubar-icons {
		margin-left: 0;
	}

	#rich-workspace::v-deep .editor__content {
		margin: 0;
		max-width: 100%;
	}
	.component-fade-enter-active, .component-fade-leave-active {
		transition: opacity .3s ease;
	}
	.component-fade-enter, .component-fade-leave-to
		/* .component-fade-leave-active below version 2.1.8 */ {
		opacity: 0;
	}
</style>
