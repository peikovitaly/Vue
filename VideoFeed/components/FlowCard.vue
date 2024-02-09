<template>
  <div class="flow-card">
    <div class="flow-card__container">
      <MuxVideoPlayer
        v-if="video"
        :video="video"
        title="video"
        class="flow-card__player"
        :post-id="post.id"
        :is-active="isActive"
        @half-watched="halfWatched"
      />

      <FlowInfo class="flow-card__info" :info="post" />

      <FlowMenu class="flow-card__menu" :post="post" />
    </div>
  </div>
</template>

<script>
import MuxVideoPlayer from "@/features/ui/common/MuxVideoPlayer.vue";
import FlowMenu from "@/features/containers/Flow/components/FlowMenu.vue";
import FlowInfo from "@/features/containers/Flow/components/FlowInfo.vue";

import validationError from "@/mixins/validationError";
import waitRequest from "@/mixins/waitRequest";

import { mapMutations } from "vuex";
import { postType } from "@/config/postsType";

export default {
  name: "FlowCard",
  components: {
    FlowInfo,
    FlowMenu,
    MuxVideoPlayer,
  },
  mixins: [validationError, waitRequest],
  props: {
    post: {
      type: Object,
      required: true,
    },
    isActive: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    video() {
      return this.post?.media?.at(0);
    },
  },

  methods: {
    ...mapMutations({
      setParams: "flows/setParams",
    }),

    halfWatched() {
      if (this.requestInProgress) return;

      return this.waitRequest(() => {
        return this.$post(
          `/posts/${this.post.id}/views`,
          {},
          this.successHalfWatched,
          () => {},
          this.checkErrors
        );
      });
    },

    successHalfWatched() {
      this.setParams({
        type: postType.flows,
        key: "views",
        value: this.post.views + 1,
        postId: this.post.id,
      });
    },
  },
};
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/vendors/_variables.scss";

.flow-card {
  background-color: $black;
  width: 100%;
  height: 100%;
  will-change: tansform;
  transform: translate3d(0, 0, 1px);

  @include media-breakpoint-up(sm) {
    display: flex !important; // add cover video behavior on desktop
    max-width: em(393);
  }

  &__container {
    height: 100%; // need to video cover on mobile
    position: relative;

    @include media-breakpoint-up(sm) {
      height: auto; // add cover video behavior on desktop
    }
  }

  &__player {
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: 3;
    position: absolute;
    top: 0px;
    left: 0px;

    @include media-breakpoint-up(sm) {
      position: relative;
      min-height: 500px;
    }
  }

  &__menu {
    position: absolute;
    bottom: em(13);
    right: em(10);
    z-index: 3;
  }

  &__info {
    max-width: 80%;
    position: absolute;
    left: em(15);
    bottom: em(16);
    z-index: 3;
  }

  @include media-breakpoint-up(sm) {
    display: block;
    width: 75%;
    height: 100%;
    margin: 21px auto 0;
    border-radius: em(15);
    overflow: hidden;
  }
}
</style>
