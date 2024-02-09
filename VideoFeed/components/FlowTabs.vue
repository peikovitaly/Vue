<template>
  <CGroup v-model="tab" :items="tabs" class="flow-tab">
    <template #default="{ item: { isActive, value } }">
      <div
        class="flow-tab__item"
        :class="{ 'flow-tab__item--active': isActive }"
      >
        <span class="flow-tab__font">{{ value.name }}</span>
      </div>
    </template>
  </CGroup>
</template>

<script>
import CGroup from "@/features/ui/CGroup.vue";
import { followType } from "@/config/postsType";
export default {
  name: "FlowTabs",
  components: { CGroup },
  props: {
    selectedTab: {
      type: String,
      default: followType.recommend
    },
  },

  watch: {
    tab(newTab, prevTab) {
      if(prevTab) {
        this.$emit("tab-change", newTab);
      }
    },
  },

  data() {
    return {
      tab:  {
        id: 2,
        name: this.$t("general.flow.tabs.forYou"),
        key: followType.recommend,
      },
      tabs: [
        {
          id: 1,
          name: this.$t("general.flow.tabs.following"),
          key: followType.following,
        },
        {
          id: 2,
          name: this.$t("general.flow.tabs.forYou"),
          key: followType.recommend,
        },
      ],
    };
  },

  mounted() {
   this.initStat();
  },

  methods: {
    initStat() {
      this.tab = this.tabs.find(tab => tab.key === this.selectedTab)
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/vendors/_variables.scss";

.flow-tab {
  display: flex;

  &__item {
    margin: 0 em(8);

    &:after {
      position: absolute;
      top: 125%;
      left: 50%;
      color: $white;
      background-color: transparent;
      border-radius: 3px;
      transform: translate3d(-50%, -50%, 0);
      content: "";
      width: 50%;
      height: em(3);
      transition: background-color $time-fast $ease;

      @include media-breakpoint-up(sm) {
        color: $black;
      }
    }

    &--active {
      &:after {
        position: absolute;
        background-color: $white;
        height: em(3);
        box-shadow: $app-shadow-2;

        @include media-breakpoint-up(sm) {
          background-color: $black;
        }
      }
    }
  }

  &__font {
    color: $white;
    text-shadow: #{$app-shadow-2};
    text-align: center;
    font-family: $font-default;
    font-size: em(16);
    font-style: normal;
    font-weight: 500;
    line-height: normal;

    @include media-breakpoint-up(sm) {
      color: $black;
    }
  }
}
</style>
