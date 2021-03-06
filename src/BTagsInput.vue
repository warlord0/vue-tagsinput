<template>
  <div>
    <div
      :class="inputClass + ' tags-input'"
    >
      <span
        v-for="(badge, index) in tagBadges"
        :key="index"
        class="badge badge-pill badge-light"
      >
        <span v-html="badge"/>
        <i
          href="#"
          class="tags-input-remove"
          @click.prevent="removeTag(index)"
        />
      </span>

      <input
        v-model="input"
        :placeholder="placeholder"
        type="text"
        @keydown.enter.prevent="tagFromInput"
        @keydown.8="removeLastTag"
        @keydown.down="nextSearchResult"
        @keydown.up="prevSearchResult"
        @keyup.esc="ignoreSearchResults"
        @keyup="searchTag"
        @value="tags"
      >

      <input
        v-if="elementId"
        :id="elementId"
        :name="elementId"
        v-model="hiddenInput"
        type="hidden"
      >
    </div>

    <p
      v-show="searchResults.length"
      class="typeahead"
    >
      <span
        v-for="(tag, index) in searchResults"
        :key="index"
        :class="{
          'badge-primary': index == searchSelection,
          'badge-dark': index != searchSelection
        }"
        class="badge"
        @click="tagFromSearch(tag)"
        v-text="tag.text"
      />
    </p>
  </div>
</template>

<script>
export default {
  props: {
    elementId: {
      type: String,
      default: 'VoerroTagsInputId'},

    inputClass: {
      type: String,
      default: 'tags-input-default-class'
    },

    existingTags: {
      type: Object,
      default: () => {
        return {}
      }
    },

    value: {
      type: [Array, String],
      default: () => {
        return []
      }
    },

    typeahead: {
      type: Boolean,
      default: false
    },

    typeaheadActivationThreshold: {
      type: Number,
      default: 1
    },

    typeaheadMaxResults: {
      type: Number,
      default: 0
    },

    placeholder: {
      type: String,
      default: 'Add a tag'
    },

    limit: {
      type: Number,
      default: 0
    },

    onlyExistingTags: {
      type: Boolean,
      default: false
    },

    deleteOnBackspace: {
      type: Boolean,
      default: true
    },

    allowDuplicates: {
      type: Boolean,
      default: false
    },

    validate: {
      type: Function,
      default: () => true
    },

    typeaheadFetch: {
      type: Function,
      default: () => {}
    }
  },

  data() {
    return {
      badgeId: 0,
      tagBadges: [],
      tags: [],

      input: '',
      oldInput: '',
      hiddenInput: '',

      searchResults: [],
      searchSelection: 0,
    }
  },

  watch: {
    tags() {
      // Updating the hidden input
      this.hiddenInput = this.tags.join(',')

      // Update the bound v-model value
      this.$emit('input', this.tags)
    },

    value() {
      this.tagsFromValue()
    }
  },

  created() {
    this.tagsFromValue()
  },

  methods: {
    tagFromInput() {
      // If we're choosing a tag from the search results
      if (this.searchResults.length && this.searchSelection >= 0) {
        this.tagFromSearch(this.searchResults[this.searchSelection])

        this.input = ''
      } else {
        // If we're adding an unexisting tag
        let text = this.input.trim()

        // If the new tag is not an empty string and passes validation
        if (!this.onlyExistingTags && text.length && this.validate(text)) {
          this.input = ''

          // Determine the tag's slug and text depending on if the tag exists
          // on the site already or not
          let slug = this.makeSlug(text)
          let existingTag = this.existingTags[slug]

          slug = existingTag ? slug : text
          text = existingTag ? existingTag : text

          this.addTag(slug, text)
        }
      }
    },

    tagFromSearch(tag) {
      this.searchResults = []
      this.input = ''
      this.oldInput = ''

      this.addTag(tag.slug, tag.text)
    },

    makeSlug(value) {
      return value.toLowerCase().replace(/\s/g, '-')
    },

    addTag(slug, text) {
      // Check if the limit has been reached
      if (this.limit > 0 && this.tags.length >= this.limit) {
        return false
      }

      // Attach the tag if it hasn't been attached yet
      if (!this.tagSelected(slug)) {
        this.tagBadges.push(text.replace(/\s/g, '&nbsp;'))
        this.tags.push(slug)
      }
    },

    removeLastTag(e) {
      if (!e.target.value.length && this.deleteOnBackspace) {
        this.removeTag(this.tags.length - 1)
      }
    },

    removeTag(index) {
      this.tags.splice(index, 1)
      this.tagBadges.splice(index, 1)
    },

    searchTag: _.debounce(function () { // eslint-disable-line no-undef
      if (this.typeahead === true) {

        if (this.oldInput != this.input) {
          this.searchResults = []
          this.searchSelection = 0
          let input = this.input.trim()

          if (this.typeaheadFetch) {
            this.typeaheadFetch({ search: input, limit: this.typeaheadMaxResults }, () => {
              process.nextTick(() => { // We need to process the props update from the parent
                // No need to call searchLoadedTag as we already have a filtered and sorted list
                for (let slug in this.existingTags) {
                  let text = this.existingTags[slug].toLowerCase()
                  this.searchResults.push({
                    slug,
                    text: this.existingTags[slug]
                  })
                }
              })
            })
          } else {
            this.searchLoadedTag(input)
          }

        }
      }

    }, 1000),

    searchLoadedTag(input) {
      if (input.length && input.length >= this.typeaheadActivationThreshold) {
        for (let slug in this.existingTags) {
          let text = this.existingTags[slug].toLowerCase()

          if (text.search(input.toLowerCase()) > -1 && !this.tagSelected(slug)) {
            this.searchResults.push({
              slug,
              text: this.existingTags[slug]
            })
          }
        }

        // Sort the search results alphabetically
        this.searchResults.sort((a, b) => {
          if (a.text < b.text) return -1
          if (a.text > b.text) return 1

          return 0
        })

        // Shorten Searchresults to desired length
        if (this.typeaheadMaxResults > 0) {
          this.searchResults = this.searchResults.slice(
            0,
            this.typeaheadMaxResults
          )
        }
      }

      this.oldInput = this.input
    },

    nextSearchResult() {
      if (this.searchSelection + 1 <= this.searchResults.length - 1) {
        this.searchSelection++
      }
    },

    prevSearchResult() {
      if (this.searchSelection > 0) {
        this.searchSelection--
      }
    },

    ignoreSearchResults() {
      this.searchResults = []
      this.searchSelection = 0
    },

    /**
     * Clear the list of selected tags
     */
    clearTags() {
      this.tags.splice(0, this.tags.length)
      this.tagBadges.splice(0, this.tagBadges.length)
    },

    /**
     * Replace the currently selected tags with the tags from the value
     */
    tagsFromValue() {
      if (this.value && this.value.length) {
        let tags = Array.isArray(this.value) ?
          this.value :
          this.value.split(',')

        if (this.tags == tags) {
          return
        }

        this.clearTags()

        for (let slug of tags) {
          let existingTag = this.existingTags[slug]
          let text = existingTag ? existingTag : slug

          this.addTag(slug, text)
        }
      } else {
        if (this.tags.length == 0) {
          return
        }

        this.clearTags()
      }
    },

    /**
     * Check if the tag with the provided slug is already selected
     */
    tagSelected(slug) {
      if (this.allowDuplicates) {
        return false
      }

      if (!slug) {
        return false
      }

      let searchSlug = this.makeSlug(slug)
      let found = this.tags.find((value) => {
        return searchSlug == this.makeSlug(value)
      })

      return !!found
    }
  }
}
</script>

<style>
/* tags-input */

.tags-input {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
}

.tags-input input {
  flex: 1;
  background: transparent;
  border: none;
}

.tags-input span {
  margin-right: 0.3rem;
  margin-bottom: 0.2rem;
}

.typeahead>span {
  cursor: pointer;
  margin-right: 0.3rem;
}
</style>
