<script>
import $ from 'jquery';

/**
* Embed a Google Docs publishable URL within a page as if it were an inline DOM component
*
* @param {String} url The Published URL of the document
* @param {Object} [urlOptions] Additional options to provide to `fetch()` when retrieving the document contents
* @param {Boolean} [follow=false] Handle all document-to-document links by replacing the current document contents, if false link will act like a regular link
* @param {Array} [fixes] Fixes to apply to the retrieved document in order, see code for default list of fixes
* @param {Object} [customFixes] An object lookup of custom fixes to apply, each fix should either return `undefined` (to carry on with processing on the top level document) or a jQuery object as a replacement body
*
* @emits click-link Emitted as `({event, url})` when a link is clicked within the document, use `event.preventDefault()` to prevent auto-opening
*
* @slot loading Loading area - defaults to a Font-Awesome spinner + "Loading..." text
*/
export default {
	data() { return {
		isLoading: true,
		docUrl: this.url, // Copy from prop initally, but can be overriden if following other doc links
		html: undefined,
	}},
	props: {
		url: {type: String, required: true},
		urlOptions: {type: Object},
		follow: {type: Boolean, default: false},
		fixes: {type: Array, default() { return [
			'bodyRoot',
			'bodyPadding',
			'bodyWidth',
			'linkTargets',
			'linkUnshorten',
		] } },
		customFixes: {type: Object, default() { return {} }},
	},
	methods: {
		/**
		* Fetch the remote URL and format it for output
		* @returns {Promise} A promise which resolves when the operation has completed
		*/
		refresh() {
			return Promise.resolve()
				.then(()=> this.isLoading = true)
				.then(()=> this.docUrl || Promise.reject('No Google Doc URL specified'))
				.then(()=> fetch(this.docUrl, this.urlOptions))
				.then(res => res.text())
				.then(html => $.parseHTML(html)) // Parse into "safe" HTML
				.then(html => $(html).toArray().find(el => el.id == 'contents')) // extract contents div only
				.then(html => $(html)) // Wrap with jQuery
				.then(doc => {
					return this.fixes.reduce((doc, fix) => {
						var fixFuncName = 'fix' + fix.substring(0, 1).toUpperCase() + fix.substring(1); // via `fix${fix |> _.upperFirst}`
						var fixFunc = this.customFixes[fixFuncName] || this[fixFuncName];

						if (!fixFunc) throw new Error(`Cannot locate fix function "${fixFuncName}" as a built-in fix or in customFixes`);
						var resultDoc = fixFunc(doc);
						return resultDoc || doc; // Use resultDoc if we have one, otherwise default back to the main document
					}, doc);
				})
				.then(doc => this.html = doc.html())
				.finally(()=> this.isLoading = false)
		},


		// Document fixes {{{
		/**
		* Remove main document maximum-width restrictions
		*/
		fixBodyRoot(doc) {
			doc
				.children('div')
				.first()
				.addClass('gdoc-root')
		},


		/**
		* Remove main document padding
		*/
		fixBodyPadding(doc) {
			doc
				.children('div')
				.first()
				.css('padding', '0 20px')
		},


		/**
		* Remove main document maximum-width restrictions
		*/
		fixBodyWidth(doc) {
			doc
				.children('div')
				.first()
				.css('max-width', 'none')
		},


		/**
		* Make all <a/> links open in a new window
		*/
		fixLinkTargets(doc) {
			doc
				.find('a[href^="http://"], a[href^="https://"]')
				.attr("target", "_blank")
		},


		/**
		* Remove Google shortner prefixes for all links
		*/
		fixLinkUnshorten(doc) {
			doc
				.find('a[href^="https://www.google.com/url"]')
				.each((i, el) => {
					var $el = $(el);
					$(el).attr('href', $el.attr('href')
						.replace(/^https:\/\/www\.google\.com\/url\?q=(.*?)&.*$/, '$1') // Remove `&...` slush from GitHub URLs
						.replace(/^https:\/\/www\.google\.com\/url\?q=/, '') // Rewrite all other URLs
					)
				})
		},
		// }}}
	},
	created() {
		this.refresh();
	},
	mounted() {
		var vm = this;
		$(this.$el).on('click', 'a', e => {
			var url = $(e.target).attr('href');
			if (!url) return; // Not a Href - ignore

			vm.$emit('click-link', {url, event: e});
			if (
				vm.follow // We can follow the link AND
				&& !event.defaultPrevented // Above emitter didn't stop us AND
				&& /^https:\/\/docs\.google\.com\/document\/.+\/pub$/.test(url) // Cross-link to another GDoc file
			) { // Replace this GDoc URL with new URL
				event.preventDefault();
				vm.docUrl = url;
				vm.refresh();
			}
		});
	},
}
</script>

<template>
	<div>
		<div v-if="isLoading">
			<slot name="loading">
				<h3>
					<i class="far fa-spinner fa-spin"/>
					Loading...
				</h3>
			</slot>
		</div>
		<div v-else v-html="html"/>
	</div>
</template>
