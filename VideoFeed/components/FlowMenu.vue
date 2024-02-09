<template>
  <div class="flow-menu">
    <!--        user button -->
    <button
      type="button"
      @click.stop="followUser"
      class="flow-menu__button flow-menu__button--user"
    >
      <span class="flow-menu-avatar">
        <b-avatar
          @click.stop
          :src="user.avatar"
          :text="user.initials"
          :to="user.url"
          size="38px"
        />
      </span>

      <span
        v-if="!isOwner && !isFollowed"
        class="flow-menu__icon-wrapper flow-menu__icon-wrapper--user"
      >
        <CIcon
          name="plus"
          path="flows"
          class="flow-menu__icon flow-menu__icon--user"
        />
        <!--        :class="{ [`flow-menu__icon&#45;&#45;user&#45;&#45;active`]: isFollowed }"-->
      </span>
    </button>

    <button
      v-for="button in menuButtons"
      :key="button.id"
      type="button"
      @click.stop="button.action"
      class="flow-menu__button"
      :class="{ 'flow-menu__button--disable': button.disabled }"
    >
      <span class="flow-menu__icon-wrapper">
        <CIcon
          :name="button.icon"
          path="flows"
          class="flow-menu__icon"
          :class="{
            [`flow-menu__icon--${button.icon}`]: button.icon,
            [`flow-menu__icon--${button.icon}--active`]: button?.liked,
          }"
        />
      </span>

      <span class="flow-menu__font flow-menu__font--text">
        {{ button.text }}
      </span>
    </button>

    <button
      type="button"
      @click.stop="tipPostOpen"
      class="flow-menu__button"
      :class="{ 'flow-menu__button--disable': isOwner || !allowTips }"
    >
      <span class="flow-menu__icon-wrapper">
        <CIcon
          name="tip"
          path="flows"
          class="flow-menu__icon flow-menu__icon--tip"
        />
      </span>

      <span class="flow-menu__font flow-menu__item-text">
        {{ $t("general.flow.menu.button.tip") }}
      </span>
    </button>

    <button
      type="button"
      @click.stop="settingsPopupOpen"
      class="flow-menu__button flow-menu__button--settings"
    >
      <span class="flow-menu__icon-wrapper flow-menu__icon-wrapper--settings">
        <CIcon
          name="settings"
          path="flows"
          class="flow-menu__icon flow-menu__icon--settings"
        />
      </span>
    </button>
  </div>
</template>
<script>
import User from "@/components/models/User";

import CIcon from "@/features/ui/CIcon.vue";

import waitRequest from "@/mixins/waitRequest";
import validationError from "@/mixins/validationError";
import follow from "@/mixins/follow";
import like from "@/mixins/like";

import { likeBaseUrls } from "@/mixins/like";
import { mapGetters, mapMutations } from "vuex";
import Post from "@/components/models/Post";

export default {
  name: "FlowMenu",

  components: { CIcon },
  mixins: [waitRequest, validationError, follow, like],

  props: {
    post: {
      type: Object,
      required: true,
    },
  },

  data() {
    return {};
  },

  computed: {
    ...mapGetters({ currentUser: "user" }),
    currentPost() {
      return new Post(this.post);
    },

    menuButtons() {
      return [
        {
          id: 1,
          icon: "video",
          action: this.videoPopupOpen,
          text: this.currentPost?.views || 0,
        },
        {
          id: 2,
          icon: "like",
          action: this.addCurrentLike,
          liked: this.likeObject.isLike,
          text: this.likeObject?.likesCount || 0,
        },
        {
          id: 3,
          icon: "comments",
          action: this.commentsPopupOpen,
          text: this.currentPost?.commentsCount || 0,
          disabled: !this.allowComments,
        },
      ];
    },

    user() {
      return new User(this.currentPost?.user) || {};
    },

    isOwner() {
      return this.currentPost?.user?.id === this.currentUser.id;
    },

    allowTips() {
      return this.currentPost.allowTips;
    },

    allowComments() {
      return this.currentPost.allowComments;
    },
  },

  mounted() {
    this.initState();
  },

  methods: {
    ...mapMutations({
      setUserParams: "flows/setUserParams",
      setParams: "flows/setParams",
    }),
    initState() {
      this.settFollowedUser(this.currentPost?.user);

      const { likesCount, isLiked, id } = this.currentPost;
      this.setLikeData({
        baseRoute: likeBaseUrls.posts,
        id: id,
        isLike: isLiked,
        likesCount: likesCount,
      });
    },

    addCurrentLike() {
      this.addLike();
      this.setParams({
        key: "likes_count",
        postId: this.currentPost.id,
        value: this.likeObject.likesCount,
      });
      this.setParams({
        key: "is_liked",
        postId: this.currentPost.id,
        value: this.likeObject.isLike,
      });
    },

    followUser() {
      this.follow();
      this.settFollowedUser(this.currentPost?.user);
    },

    videoPopupOpen() {
      this.$popup.open("VideoCountPopup", {
        amount: this.currentPost.views || 0,
      });
    },

    callbackOnMessageSend() {
      this.setParams({
        key: "comments_count",
        postId: this.currentPost.id,
        value: this.currentPost?.commentsCount + 1,
      })
    },

    callbackOnMessageRemove() {
      this.setParams({
        key: "comments_count",
        postId: this.currentPost.id,
        value: this.currentPost?.commentsCount - 1,
      })
    },

    commentsPopupOpen() {
      this.$popup.open("FlowCommentsPopup", {
        postId: this.currentPost.id,
        callbackOnMessage: this.callbackOnMessageSend,
        callbackOnMessageRemove: this.callbackOnMessageRemove,
      });
    },

    settingsPopupOpen() {
      this.$popup.open("SettingsFlowPopup", { postId: this.currentPost.id });
    },

    tipPostOpen() {
      this.$popup.open("TipPopup");
    },
  },
};
</script>

<style lang="scss" scoped>
.flow-menu {
  &__button {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: $app-font-color-gray-1;
    margin-top: em(14);

    &--settings {
      margin-top: em(20);
      width: em(38);
      height: em(38);
    }

    &--user {
      margin-bottom: em(24);
    }

    &--disable {
      pointer-events: none;
      opacity: 0.4;
    }
  }

  &__icon-wrapper {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: em(34);
    height: em(34);

    &--settings {
      border-radius: 50%;
      background: radial-gradient(
        70.71% 70.71% at 50% 50%,
        #292929 0%,
        #000 100%
      );
      filter: drop-shadow(#{$app-shadow});
    }

    &--user {
      position: absolute;
      bottom: em(-9);
      z-index: 3;
      width: em(19);
      height: em(19);
      color: $app-blue-3;

      &:before {
        content: "";
        position: absolute;
        width: 200%;
        height: 150%;
      }

      &--active {
        color: $app-red;
      }
    }
  }

  &__icon {
    color: rgba($white, 0.85);
    width: em(24);
    height: em(26);
    filter: drop-shadow(#{$app-shadow});;

    &--like {
      width: em(32);
      height: em(32);

      &--active {
        color: $app-red;
      }
    }

    &--user {
      color: $app-blue-3;
      width: em(19);
      height: em(19);

      &--active {
        color: $app-red;
      }
    }

    &--comments {
      width: em(32);
      height: em(26);
    }

    &--tip {
      width: em(32);
      height: em(32);
    }

    &--settings {
      width: em(34);
      height: em(34);
    }
  }

  &__font {
    color: $white;
    text-shadow: $app-shadow-2;
    text-align: center;
    font-family: $font-default;
    font-size: em(12);
    font-style: normal;
    font-weight: 600;
    line-height: normal;
  }
}

.flow-menu-avatar {
  position: relative;
  border-radius: 50%;
  border: 1px solid $white;
  z-index: 3;
  filter: drop-shadow(#{$app-shadow});
}
</style>
