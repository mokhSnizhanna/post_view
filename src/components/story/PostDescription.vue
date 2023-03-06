<template>
  <div v-if="description"
       v-html="description"
       :class="['my-feed__item__description', isFeedWithOutFile ? 'my-feed__item__description__big' : 'my-feed__item__description__small']"
  ></div>
  <div v-else
       :class="['my-feed__item__description', isFeedWithOutFile ? 'my-feed__item__description__big' : 'my-feed__item__description__small']">
    <div v-for="div in postDescription">
      <template v-for="part in div.context">
        <template v-if="part.htmlText === 'br'">
          <br>
        </template>
        <template v-else>
          {{ part.htmlText }}
          <a v-if="part.preview" :href="part.preview.link" target="_blank">{{part.preview.link}}</a>
          <preview-link v-if="part.preview" :link-info="part.preview"></preview-link>
        </template>
      </template>
    </div>
  </div>
</template>

<script>
import PreviewLink from './PreviewLink.vue'

export default {
  name: 'PostDescription',
  components: {
    PreviewLink,
  },
  props: {
    story: {
      type: Object,
      required: true,
    }
  },
  template: `<%- include('./descriptionTemplate.ejs') %>`,
  data() {
    return {
      postDescription: [],
      description: '',
    };
  },
  computed: {
    isFeedWithOutFile: function() {
      return !this.story.coverVideo && !this.story.coverImage;
    },
  },
  mounted() {
    if (this.story.description && this.story.description.length > 0) {
      const regLink = /(?<url>(?:https?:\/\/){0,1}(?:[a-zA-Z0-9_-]+\.)+(?:[a-zA-Z]{2,10})(?:(?:[\/?#=_\-\.]*?)(?:\w*?))+)/g;
      const matchLink = /((?:https?:\/\/){0,1}(?:[a-zA-Z0-9_-]+\.)(?:\w{2,10})\S+)/;
      const regDivTag = /<div[^>]*>(.*?)<\/div>/g;

      let splits = this.story.description.split(regLink).filter(n => { return n; });
      if (splits && splits.length > 0) {
        const divSplits = this.story.description.split(regDivTag).filter(n => { return n; });
        for (let divText of divSplits) {
          let parts = divText.split(regLink).filter(n => { return n; });
          let context = [];
          parts.map(part => {
            const match = part.match(matchLink);
            if(part && match && match[0].length === part.trim().length && window.HT.linkPreviews[part] && window.HT.linkPreviews[part].isValid) {
              if(!part.match('https?://')) {
                part = 'https://'+part.trim();
              }
              context.push({htmlText: '', preview: {link: part.trim(), ...window.HT.linkPreviews[match[0]]}});
            } else {
              let brSplits = part.split('<br>');
              if (brSplits.length <= 1) {
                context.push({htmlText: part});
              } else {
                brSplits = brSplits.filter(n => { return n; });
                brSplits.map(part => {
                  context.push({htmlText: part});
                  context.push({htmlText: 'br'});
                });

              }
            }
          });
          this.postDescription.push({context});
        }
      } else {
        this.description = this.story.description;
      }
    }
  },
}
</script>

<style scoped>

</style>
