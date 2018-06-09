# GitHub Content Script Navigation
Content scripts on GitHub don't work as expected: content scripts ordinarily
reload for each new page visited but, on GitHub, they don't. This is because
links on GitHub mutate the DOM and use the [history.pushState API][], instead of
loading pages the standard way, which creates an entirely new DOM. We fix this
by adding a MutationObserver that will check when the DOM has been updated,
thus indicating that the new page has been loaded from a user's perspective,
and notify the WebExtension about this event.

## Usage
To use this fix, copy `github_navigation.js` into your project. In your
`manifest.json`, add it as a content script after your other content script files:
```json
"content_scripts": [{
   "matches": ["*://*.github.com/*"],
   "js": ["content_script.js", "github_navigation.js"]
 }],
```

In another content script file, define an `onPageLoad` function:
```js
function onPageLoad() {
    // Code to run when the page loads
}
```

`onPageLoad` will be called when a new page loads, i.e. when the
content script is first loaded and when the history.pushState APIs
are used.

See [the sample][] for more details.

### Caveats
Some pages will have additional async events (e.g. the Milestone page loads issues
asynchronously) so it may be necessary to add additional logic to check for page
load completion.

**Important:** this implementation is fragile. It relies on the structure of the
GitHub website, which is subject to change.

## Alternatives
The seemingly "correct" approach would be to create a `history.pushState`
observer using [the `webNavigation.onHistoryStateUpdated` listener][hist listener].
However, this listener does not work as expected: it's called twice -- once 
before the DOM has been mutated and once after -- and I haven't found a good
way to distinguish them other than to look at changes in the DOM, which is
already the approach this solution takes.

## License
The license for this repository is the X11 license, which is similar to the MIT license.

[the sample]: https://github.com/mcomella/github-content-script-navigation/sample
[history.pushState API]: https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_pushState()_method 
[hist listener]: https://developer.mozilla.org/en-US/Add-ons/WebExtensions/API/webNavigation/onHistoryStateUpdated
