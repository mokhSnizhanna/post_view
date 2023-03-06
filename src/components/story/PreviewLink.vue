<template>
  <a :href="linkInfo.link" v-if="linkInfo.isValid" class="preview-link" target="_blank">
    <div v-if="hasImage" class="preview-link__image">
      <img :src="image" @error="handleErrorImage">
    </div>
    <div class="preview-link__info">
      <p v-if="hasSiteName" class="preview-link__name">{{ siteName }}</p>
      <p v-else class="preview-link__name">{{ siteLink }}</p>
      <p v-if="hasTitle" class="preview-link__title">{{ title }}</p>
      <p v-if="hasDescription" class="preview-link__description">{{ description }}</p>
    </div>
  </a>

</template>

<script>


export default {
  name: 'PreviewLink',
  props: {
    linkInfo: {
      type: Object,
    }
  },
  template: `<%- include('./previewLinkTemplate.ejs') %>`,
  data() {
    return {
      isShowImage: true
    };
  },
  computed: {
    hasPreview: function() {
      return this.linkInfo && this.linkInfo.preview;
    },
    hasTitle: function() {
      return this.hasPreview && (this.linkInfo.preview['og:title'] || this.linkInfo.preview.title);
    },
    title: function() {
      return this.hasTitle ? this.normalizeString(
          (this.linkInfo.preview['og:title'] ?
              this.linkInfo.preview['og:title'] :
              this.linkInfo.preview.title)
      ) : '';
    },
    hasDescription: function() {
      return this.hasPreview && (this.linkInfo.preview['og:description'] || this.linkInfo.preview.description);
    },
    description: function() {
      return this.hasDescription ? this.normalizeString(
          (this.linkInfo.preview['og:description'] ?
              this.linkInfo.preview['og:description'] :
              this.linkInfo.preview.description)
      ) : '';
    },
    hasSiteName: function() {
      return this.hasPreview && (this.linkInfo.preview['og:site_name'] || this.linkInfo.preview.site_name);
    },
    siteName: function() {
      return this.hasSiteName ? (this.linkInfo.preview['og:site_name'] ?
          this.linkInfo.preview['og:site_name'] : this.cropSiteName(this.linkInfo.preview.site_name)) : '';
    },
    siteLink: function() {
      return this.linkInfo.normalizedURL ?
          this.cropSiteName(this.linkInfo.normalizedURL) :
          this.cropSiteName(this.linkInfo.url);
    },
    hasImage: function() {
      return this.isShowImage && this.hasPreview && (this.linkInfo.preview.image || this.linkInfo.preview['og:image']);
    },
    image: function() {
      return this.hasImage ? (this.linkInfo.preview['og:image'] ? this.linkInfo.preview['og:image'] : this.linkInfo.preview.image) : '';
    },
    previewImageStyle() {
      return {
        "background-image": `url(${this.image})`,
      };
    },
  },
  methods: {
    normalizeString(string) {
      var parser = new DOMParser;
      var dom = parser.parseFromString(
          '<!doctype html><body>' + string + '</body></html>',
          'text/html');
      return dom.body.textContent;
    },
    cropSiteName(link) {
      return link ? new URL(link).hostname : '';
    },
    handleErrorImage() {
      this.isShowImage = false;
    }
  }
}
</script>

<style scoped>

</style>
