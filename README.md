Vue-GDocs
=========
Vue component to embed Google Docs content.

Features:

* Works with any published Google Docs URL
* Fixes up padding / width of the original automatically
* Removes tracking beacons from URLs
* Makes all links open in a new tab by default


```vue
<script>
import VueGdocs from 'vue-gdocs';

export {
	components: {VueGdocs}
};
</script>

<template>
	<div>
		<vue-gdocs :url="https://docs.google.com/document/d/e/SOME-LONG-HASH/pub"/>
	</div>
</template>
```


Installation
------------
Install via NPM:

```
npm install --save vue-gdocs
```


Import into your app:

```vue
<script>
import VueGdocs from 'vue-gdocs';

export {
	components: {VueGdocs}
};
</script>
```

Use in your template:

```vue
<template>
	<div>
		<vue-gdocs :url="https://docs.google.com/document/d/e/SOME-LONG-HASH/pub"/>
	</div>
</template>
```


API
===

Properties
----------
The `vue-gdocs` component accepts the following properties:

| Property      | Type      | Default   | Description                                                                                                                  |
|---------------|-----------|-----------|------------------------------------------------------------------------------------------------------------------------------|
| `url`         | `String`  |           | The URL of the *published* Google Doc to display                                                                             |
| `urlOptions`  | `Object`  | `{}`      | Additional options to pass to `fetch` when retrieving the URL                                                                |
| `follow`      | `Boolean` | `false`   | Handle all document-to-document links by replacing the current document contents, if false link will act like a regular link |
| `fixes`       | `Array`   | See below | An array of fixes to apply, in order                                                                                         |
| `customFixes` | `Object`  | `{}`      | Additional user defined fixes, see notes below                                                                               |


**Notes:**
* Fixes defaults to all available fixes being enabled in a suitable execution order
* `customFixes` is a simple lookup object of functions to run. Each function is called as `($doc)` (a jQuery wrapped version of the full document. Each function can return `undefined` (to carry on with processing on the top level document) or a jQuery object as a replacement body
* Following links only works if the link is a published document link - i.e. should end in `/pub`


Events
------
The following events are emitted from the component:

| Event        | Payload          | Description                                                                                                  |
|--------------|------------------|--------------------------------------------------------------------------------------------------------------|
| `click-link` | `({url, event})` | Emitted when a link is clicked within the document, use `event.preventDefault()` to prevent normal behaviour |



Fixes
-----
The following fixes ship with the component and are all enabled by default:

| Fix             | Description                                                        |
|-----------------|--------------------------------------------------------------------|
| `bodyRoot`      | Add the `gdoc-root` class to the actual body root `<div/>` wrapper |
| `bodyPadding`   | Remove main document padding                                       |
| `bodyWidth`     | Remove main document maximum-width restrictions                    |
| `linkTargets`   | Make all `<a/>` links open in a new window                         |
| `linkUnshorten` | Remove Google shortner prefixes for all links                      |
