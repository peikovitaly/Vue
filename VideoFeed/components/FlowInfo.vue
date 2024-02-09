<template>
  <div class="flow-info">
    <div class="flow-info__location" v-if="hasLocation">
      <LocationMark :location="location" />
    </div>

    <div class="flow-info__user">
      <CLink :to="`/${user}`" class="flow-info__font flow-info__font--user">
        @{{ user }}
      </CLink>
    </div>

    <div class="flow-info__content" v-if="message.base">
      <span class="flow-info__font flow-info__font--info">
        {{ message.base }}
      </span>

      <b-collapse
        v-if="message.addition"
        v-model="visible"
        :id="`collapse-show-more-${info.id}`"
        class="flow-info__collapse"
      >
        <span class="flow-info__font flow-info__font--info">
          {{ message.addition }}
        </span>
      </b-collapse>

      <button
        v-if="message.addition"
        type="button"
        v-b-toggle="`collapse-show-more-${info.id}`"
        class="button flow-info__button flow-info__button--more"
      >
        <span class="flow-info__font flow-info__font--info">
          {{ !visible ? "...more" : "less" }}
        </span>
      </button>
    </div>

    <div class="flow-info__footer">
      <button
        v-if="hasRefPost"
        @click="$router.push(`/${refId}/${userId}`)"
        type="button"
        class="button flow-info__button flow-info__button--video"
      >
        <span class="flow-info__font flow-info__font--button">
          {{ $t("general.flow.info.button") }}
        </span>

        <CIcon name="hex-video" path="flows" class="flow-info__icon" />
      </button>
    </div>
  </div>
</template>
<script>
import CIcon from "@/features/ui/CIcon.vue";
import LocationMark from "@/features/components/LocationMark.vue";
import CLink from "@/features/ui/CLink.vue";
import { formatDistance } from "@/tools/helpers";

export default {
  name: "FlowInfo",
  components: { CLink, LocationMark, CIcon },
  props: {
    info: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      visible: false,
    };
  },
  computed: {
    user() {
      return this.info?.user?.username;
    },
    userId() {
      return this.info.user.id;
    },
    refId() {
      return this.info?.ref_post_id;
    },
    hasRefPost() {
      return Boolean(this.info?.ref_post_id);
    },

    distance() {
      return formatDistance(this.info?.distance) || 0;
    },
    hasLocation() {
      return Boolean(this.info?.location);
    },
    location() {
      if(!this.distance) return `${this.info?.location?.address}`

      return `${this.info?.location?.address} · ${this.distance} away`;
    },
    message() {
      const message = this.info?.message;
      const baseMessageMaxLength = 80;
      const isLongMessage = message.length >= baseMessageMaxLength;
      const messageBase = isLongMessage ? message.slice(0, 80) : message;
      const messageAddition = isLongMessage ? message.slice(80) : "";

      return { base: messageBase, addition: messageAddition };
    },
  },
};
</script>

<style lang="scss" scoped>
.flow-info {
  &__location {
  }

  &__user {
    margin-top: em(5);
  }

  &__content {
    position: relative;
    margin-top: em(6);
  }

  &__collapse {
    display: inline;
  }

  &__footer {
    margin-top: em(10);
  }

  &__button {
    &--more {
      display: inline-flex;
      width: auto;
      position: relative;
      bottom: 0;
      right: 0;
    }

    &--video {
      align-items: center;
      justify-content: space-between;
      padding: em(9) em(9) em(10) em(14);
      border-radius: em(100);
      background: $app-blue-gradient-2;
      min-width: em(172);
    }
  }

  &__font {
    font-family: $font-default;
    font-style: normal;
    line-height: normal;
    color: $white;

    &--button {
      font-size: em(12);
      font-weight: 700;
    }

    &--info {
      word-break: break-word;
      color: white;
      text-shadow: $app-shadow-2;
    }

    &--user {
      font-size: em(16);
      font-weight: 500;
      color: $white;
      text-shadow: $app-shadow-2;
    }

    &--font {
      display: inline-block;
    }
  }

  &__icon {
    color: $white;
    width: em(21);
    height: em(21);
  }
}
</style>
