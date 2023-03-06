<template>
  <div class="my-feed__item">
    <div class="my-feed__item__header">
      <div class="d-flex align-items-center">
        <a v-bind:href="authorUrl" class="my-feed__item__author d-flex align-items-center">
          <div class="my-feed__item__author__avatar">
            <img class="my-feed__item__author__avatar--img" :src="story.author.avatar.url" :alt="story.author.username" />
            <img v-if="story.author.badge.icon && story.author.badge.label"
                 class="my-feed__item__author__avatar--badge"
                 :src="badgeIconUrl"
                 :alt="story.author.badge.label" />
          </div>

          <span>{{ story.author.username }}</span>
        </a>
        <span class="my-feed__item__date ml-1">&#183; {{ formattedPublishedDate }}</span>
      </div>

      <div v-if="isDraft" class="profile--actions-block">
        <profile-actions
            :slug="story.slug"
            :storyType="story.storyType"
            :username="story.author.username"
            :is-contributor="user && user.userSubTypes > 0"
            @onEdit="handleEditFeed"
            :type="getProfileActionsType()">
        </profile-actions>
      </div>
    </div>
    <post-description v-if="isPostcard && story.description" :story="story"></post-description>

    <div class="my-feed__item__object">
      <slot name="player"></slot>
      <div v-if="Boolean(story.coverImage) && !Boolean(story.coverVideo)" class="my-feed__item__img">
        <a v-if="!isPostcard" v-bind:href="storyUrl" class="my-feed__item__cover">
          <img class="my-feed__item__cover__img" :src="story.coverImage.url ? story.coverImage.url : story.coverImage">
        </a>
        <div v-else class="my-feed__item__cover my-feed__item__cover__postcard mb-4" :style="postcardImageStyle"  @click="handleOpenPreview">
          <img  v-if="isShowPostcardImage" class="my-feed__item__cover__postcard__img" :src="story.coverImage">
        </div>
      </div>
    </div>

    <slot name="comment"></slot>
  </div>
</template>

<script>
import PostDescription from './PostDescription.vue'

export default {
  name: 'StoryListItem',
  components: {
    PostDescription,
  },
  props: {
    story: {
      type: Object,
      default: () => {},
    },
    user: {
      type: Object,
      default: () => {},
    },
    isDraft: {
      type: Boolean,
      default: false,
    }
  },
  data() {
    return {
      likeCount: this.story ? this.story.likeCount : 0,
      commentCount: this.story ? this.story.commentCount : 0,
      isCommentInputFocus: false,
      isOpenModal: false,
      postcardImageStyle: {},
      isShowPostcardImage: true,
      previewLinks: [],
      postDescription: '',
      autoPlayVideoId: ''
    };
  },
  async mounted() {
    if (this.isPostcard) {
      this.initBackShowVideo();
      this.initVideoFullScreen();
      if (Boolean(this.story.coverImage) && !Boolean(this.story.coverVideo)) {
        this.postcardImageStyle = await this.getPostcardImageStyle();
      }
    }
  },
  computed: {
    channelUrl: function() {
      return this.story.channel.channelUrl ? this.story.channel.channelUrl : `/${this.story.channel.channelName}`;
    },
    formattedPublishedDate: function() {
      if(this.story.publishedDate){
        return moment(this.story.publishedDate).format('MMM DD');
      }
      return moment(this.story.updatedAt).format('MMM DD');
    },
    isFeedWithOutFile: function() {
      return !this.story.coverVideo && !this.story.coverImage;
    },
    isPostcard: function() {
      return this.story.storyType === 'Postcard';
    },
    showCategoryBlock: function() {
      return this.story.categories.length > 0;
    },
    showLikeCount: function() {
      return this.likeCount;
    },
    showCommentCount: function() {
      return this.commentCount;
    },
    commentButtonClass: function() {
      return {
        'feed-actions__buttons__comment': true,
        'mx-3': true
      };
    },
    isSharePostcard: function() {
      return Boolean(this.story.storyShare);
    }
  },
  methods: {
    handleLike: function() {
      this.likeCount++;
      this.$emit('liked', true);
    },
    handleUnlike: function() {
      this.likeCount--;
      this.$emit('liked', false);
    },
    handleEditFeed: function() {
      if (this.isPostcard) {
        this.isOpenModal = true;
        this.$emit('openModal', {
          modalId: 'createPostcard', story: this.story, user: this.user,
        });
      } else {
        const storyUrl = this.story.storyType === 'Episode' ?
            (this.story.storyUrl).replace('/story/', '/video/') : this.story.storyUrl;
        window.location.href = storyUrl+'/edit';
      }
    },
    handleCommentFocus: function() {
      if (this.isPostcard) {
        this.isCommentInputFocus = !this.isCommentInputFocus;
      } else {
        const storyUrl = this.story.storyType === 'Episode' ?
            (this.story.storyUrl).replace('/story/', '/video/') : this.story.storyUrl;
        localStorage.setItem('scrollToComment', true);
        window.location.href = storyUrl;
      }
    },
    handleChangeCountComment: function(data) {
      this.commentCount = data.count;
    },
    initBackShowVideo: function() {
      if(!!this.story.coverVideo && this.story.storyType === 'Postcard') {
        this.$on('closePostcardModal', (data) => {
          if (data.slug === this.story.slug) {
            this.isOpenModal = false;
          }
        });
      }
    },
    initVideoFullScreen: function() {
      if(!!this.story.coverVideo && this.story.storyType === 'Postcard') {
        this.$on('brightcove.fullscreenc', (data) => {
          if (data.videoId === this.story.coverVideo.video_id && !this.isOpenModal) {
            this.isOpenModal = true;
            setTimeout(() => {
              this.$emit('openModal', {
                modalId: 'postcardPreview',
                story: this.story,
                user: this.user,
                authorUrl: this.authorUrl,
                type: this.type,
                isShowPostShare: this.isPreview,
              });
            }, 10);

          }
        });
      }
    },
    handleOpenPreview: function() {
      this.isOpenModal = true;
      this.$emit('openModal', {
        modalId: 'postcardPreview',
        story: this.story,
        user: this.user,
        authorUrl: this.authorUrl,
        type: this.type,
        isShowPostShare: this.isShowPostShare,
      });
    },
    getProfileActionsType: function() {
      switch (this.story.storyType) {
        case 'Story':
          return 'story';
        case 'Episode':
          return 'video';
        case 'Postcard':
        default:
          return 'post';
      }
    },
    getPostcardImageStyle: async function() {
      const imageSize = await this.getImageSize(this.story.coverImage);
      let calculateHeight = 500;
      this.isShowPostcardImage = true;

      const ratio16to9 = 16 / 9;
      const coefficient16to9 = 9 / 16;
      const ratio4to5 = 4 / 5;
      const coefficient4to5 = 5 / 4;
      const ration4to3 = 4  / 3;
      const coefficient4to3 = 3 / 4;

      const ratio = (imageSize.width / imageSize.height);
      if (imageSize.width - imageSize.height <= 100 && (imageSize.width - imageSize.height) >= 0) { //1:1
        calculateHeight = this.defaultWidth - (Math.trunc((imageSize.width - imageSize.height) / 2));
      } else if (imageSize.width < imageSize.height) { //4:5
        calculateHeight = Math.ceil(this.defaultWidth * coefficient4to5);
      } else if ((ratio <= ratio16to9 && ratio > 1.5) || ratio > ratio16to9) { //16:9
        calculateHeight = Math.ceil(this.defaultWidth * coefficient16to9);
        if (ratio > Math.ceil(ratio16to9)) {
          this.isShowPostcardImage = false;
          return {
            minHeight: calculateHeight+'px',
            backgroundImage: `url('${this.story.coverImage}')`,
            display: 'block',
            backgroundPosition: '50%',
            backgroundRepeat: 'no-repeat',
            backgroundSize: 'cover',
          };
        }
      } else {
        calculateHeight = Math.ceil(this.defaultWidth * coefficient4to3); //4:3
      }

      return {
        maxHeight: calculateHeight+'px',
      };
    },
    getImageSize: async function(url) {
      return new  Promise(resolve => {
        const image = new Image();
        image.onload = function() {
          resolve({width: this.width, height: this.height});
        };
        image.src = url;
      });
    },
    handleUpdateAutoPlayVideoId: function({id}) {
      this.autoPlayVideoId = id;
    }
  },
}
</script>
