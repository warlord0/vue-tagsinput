# Vue Tags Input

A simple tags input with typeahead built with Vue.js 2.

Forked from @voerror/vue-tagsinput

I needed a tags input that would use data from an XHTTPRequest/Ajax as the tagged items would be in their thousands. So a lookup to an API was needed.

I reworked the source and added in some code to allow the use of a custom fetch callback. This populates the parent's data source based on the typeahead.

![](docs/screenshot.png)

## Installation via NPM

```
npm i git+https://git@github.com/warlord0/vue-tagsinput --save-dev
```
or
```
npm i git+https://git@github.com/warlord0/vue-tagsinput --save
```

Then register the component with Vue:

```javascript
import BTagsInput from '@warlord0/vue-tagsinput';

Vue.component('tags-input', BTagsInput);
```

Include the `dist/style.css` file on your page to apply the styling. You can use CDN, `require()` it inside your JS code, or `@include` it inside your (S)CSS assets. Read the `Styling` section to learn how to modify the appearance.

Include the CSS file on your page to apply the styling. Read the `Styling` section to learn how to modify the appearance.

## Usage

```html
<b-tags-input element-id="tags"
    v-model="selectedTags"
    :existing-tags="{
        'web-development': 'Web Development',
        'php': 'PHP',
        'javascript': 'JavaScript',
    }"
    :typeahead="true"></tags-input>
```

```html
<b-tags-input element-id="tags"
    v-model="selectedTags"
    :existing-tags="{
        1: 'Web Development',
        2: 'PHP',
        3: 'JavaScript',
    }"
    :typeahead="true"></tags-input>
```

with callback to API:

```html
<b-tags-input
  :typeahead="true"
  :typeahead-max-results="20"
  :typeahead-activation-threshold="3"
  :existing-tags="mytags"
  v-model="model.field"
  :only-existing-tags="true"
  typeahead-fetch="fetch"
  placeholder="Choose Tag"
  element-id="mytag"
/>
```

`element-id` will be applied to `id` and `name` attributes of the hidden input that contains the list of the selected tags as its value. Optionally you can also use the `v-model` directive to bind a variable to the array of selected tags.

`existing-tags` is the list of the existing on your website tags. Include it even if you're not using typeahead.

Remove the `typeahead` property to disable this functionality.

#### Setting Selected Tags Programmatically

If you need to programmatically (manually) set or change the list of selected tags from "outside" - just set the required value to the variable bound with the component via `v-model`.

Acceptable values:
- an array of tag slugs or tag strings
- a string with tags separated via comma)

For example, the variable name is `selectedTags`:
```html
<tags-input element-id="tags"
    v-model="selectedTags"></tags-input>
```

You can pre-set the value of this variable:
```javascript
new Vue({
    el: '#app',

    components: { BTagsInput },

    data: {
        selectedTags: [
            'tags',
            'selected',
            'by',
            'default',
        ],
        // ALTERNATIVELY
        selectedTags: 'tags,selected,by,default',
    }
});
```

or change it whenever you need to:
```javascript
new Vue({
    el: '#app',

    components: { BTagsInput },

    data: {
        selectedTags: [],
    },

    methods: {
        setSelectedTags() {
            this.selectedTags = ['programmatically', 'selected', 'tags'];
            // ALTERNATIVELY
            this.selectedTags = 'programmatically,selected,tags';
        }
    }
});
```
### Example API data retrieval

In the vue's `method: {}``

```JavaScript    
  fetch: function (search, callback) {
    this.axios.get('/api/data', {
      params: {
        query: search.search,
        limit: (search.limit > 10) ? search.limit : 10
      }
    }).then((response) => {
      this.mytags = response
      callback()
    })
  }
```    

#### All Available Props

Prop | Type | Default | Required | Description
--- | --- | --- | --- | ---
elementId | String | - | no | id & name for the hidden input
existingTags | Object | {} | no | An object with existing tags where keys are tag slugs or ids and values are strings to be displayed
typeahead | Boolean | false | no | Whether the typeahead (autocomplete) functionality should be enabled.
typeahead-max-results | Number | 0 | no | Maximum number of typeahead results to be shown. 0 - unlimited.
typeahead-activation-threshold | Number | 1 | no | Show typeahead results only after at least this many characters were entered.
typeahead-fetch | Function | {} | false | Callback to fetch data from API
placeholder | String | 'Add a tag' | no | The placeholder of the tag input.
limit | Number | 0 | no | Limit the number of tags that can be chosen. 0 = no limit.
only-existing-tags | Boolean | false | no | Only existing tags can be added/chosen. New tags won't be created.
input-class | String | 'tags-input-default-class' | no | Apply a class to make the wrapping div look like an input. For example, you can use 'form-control' for Bootstrap or 'input' for Bulma.
delete-on-backspace | Boolean | true | no | Whether deleting tags by pressing Backspace is allowed.
allow-duplicates | Boolean | false | no | Allow users to add the same tags multiple times.
validate | Function | text => true | false | Callback to validate tags' text with

## Data

The list of selected tags is stored as a string (tags separated with a comma) inside a hidden input with id and name set to the value from the `element-id` props (but only if you've provided this prop).

You can also bind the array of selected tags to a variable via `v-model`.

If a tag is listed in `existing-tags`, the tag's slug will be used, otherwise the text entered by user is added.

Example value of the hidden input:
```
web-development,javascript,This is a new tag,php
```

## Styling

Edit the `dist/style.css` file if you want to modify the package's appearance.

You can apply a wrapper class to make the input look different via the `input-class` prop. `input-class="form-control"` if you're working with bootstrap, `input-class="input"` if you're working with Bulma, or `input-class="your-custom-class"` if you have something else.

## Using Typeahead (Autocomplete)

When search results are displayed underneath the input, use the `arrow down` and `arrow up` keys on the keyboard to move the selection. Press `Enter` to select a tag. Press `Esc` to discard the search results and then `Enter` to add a new tag the way you've typed it.

## Updating From Older Versions

#### 1.8.0 -> 1.8.1

`VoerroTagInput` renamed to more generic `BTagsInput` as in Boostrap4-TagsInput

#### Older versions up to v1.4.0 -> v1.5.0

The `oldTags` property was removed. See the `Setting Selected Tags Programmatically` section on how to (pre)set the list of existing tags.

#### v1.5.0 -> v1.5.1

`TagsInput` was renamed to `VoerroTagsInput` to eliminate possible name conflicts with other packages.
