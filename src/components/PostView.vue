<template>
  <div class="postcard--preview--page__central">
    <div class="postcard--preview--page__block">
      <story-list-item
          :is-draft="isDraft"
          :story="story"
          :user="user" />
    </div>
  </div>
</template>

<script>
import StoryListItem from './story/StoryListItem.vue'

export default {
  name: 'PostView',
  components: {
    StoryListItem,
  },
  props: {},
  data() {
    return {
      user: null,
      story: null,
    }
  },
  computed: {
    isDraft() {
      if (!this.story || !this.user) return false;
      return this.story.author.username === this.user.username
    }
  }
}
</script>

<style scoped>

</style>
