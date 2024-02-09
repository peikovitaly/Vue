<template>
  <div class="flow">
    <div class="flow__header">
      <NSFWToggle class="flow__toggle" @change="nfswChange" />

      <FlowTabs
        :key="flowType"
        class="flow__tabs"
        @tab-change="followingChange"
        :selected-tab="flowType"
      />

      <button
        type="button"
        @click="$router.push({ name: 'discover' })"
        class="flow__search"
      >
        <CIcon name="search" class="flow__icon flow__icon--search" />
      </button>
    </div>

    <div class="flow__desktop-content">
      <transition name="fade">
        <InfinityScroll
          v-if="flows.length"
          :key="flowType"
          :items="flows"
          :has-more="true"
          :initial-page="initialPage"
          @rich-end="loadMore"
        >
          <template #default="{ item }">
            <FlowCard :post="item" :key="item.id"/>
          </template>
        </InfinityScroll>
      </transition>
    </div>

    <div class="flow__mobile-content">
      <CSwiper
        :key="key"
        v-if="flows.length"
        :items="flows"
        :slider-config="sliderConfig"
        class="flow__swiper"
        @change="handleSwipe"
        v-touch:swipe.bottom="toTop"
      >
        <template #slide="{ itemData: post } = {}">
          <FlowCard :post="post" :is-active="checkIsActive(post.id)" />
        </template>
      </CSwiper>
    </div>

    <div class="flow__footer">
      <FooterMenu is-flow-menu />
    </div>
  </div>
</template>

<script>
import FooterMenu from "@/layout/FooterMenu.vue";

import CSwiper from "@/features/ui/CSwiper.vue";
import CIcon from "@/features/ui/CIcon.vue";
import NSFWToggle from "@/features/containers/NSFWToggle.vue";
import FlowTabs from "@/features/containers/Flow/components/FlowTabs.vue";
import FlowCard from "@/features/containers/Flow/components/FlowCard.vue";
import InfinityScroll from "@/features/components/InfinityScroll.vue";

import validationError from "@/mixins/validationError";
import waitRequest from "@/mixins/waitRequest";
import flow from "@/mixins/flow";

import { mapGetters, mapMutations } from "vuex";

const flowSwiperConfig = {
  direction: "vertical",
  slidesPerView: 1,
  speed: 900,
  navigation: false,
  centeredSlides: true,
  observer: true,
  virtual: {
    enabled: true,
  },
};

export default {
  name: "Flow",
  components: {
    FlowCard,
    InfinityScroll,
    CIcon,
    FlowTabs,
    NSFWToggle,
    FooterMenu,
    CSwiper,
  },

  props: {
    postId: {
      type: String,
      default: null,
    },
  },

  mixins: [validationError, waitRequest, flow],

  computed: {
    ...mapGetters({ activeVideoIndex: "flows/activeVideoIndex" }),
  },

  data() {
    return {
      key: Math.random(),
      sliderConfig: null,
      initialPage: 1,
      isBeginning: true,
      activeSliderVideoIndex: 0,
      thresholdForNewSlides: 4,
    };
  },

  mounted() {
    this.initState();
  },

  methods: {
    ...mapMutations({
      setActiveVideoIndex: "flows/setActiveVideoIndex",
      setContentSettings: "flows/setContentSettings",
      setSettings: "flows/setSettings",
      setSettingsSex: "flows/setSettingsSex",
    }),

    async initState() {
      await this.fetchSettings();
      this.key = Math.random();

      if (this.flows.length && !this.postId) {
        this.activeSliderVideoIndex = parseInt(this.getStorageSliderIndex());
        this.key = this.activeSliderVideoIndex;
      } else {
        this.initialLoadPost(this.postId);
      }

      this.sliderConfig = {
        ...flowSwiperConfig,
        initialSlide: this.activeSliderVideoIndex,
      };
    },

    async fetchSettings() {
      await this.getContentSettings();
      await this.getContentSettingsCategories();
      await this.getContentSettingsSex();
    },

    checkIsActive(postId) {
      return this.activeVideoIndex === postId;
    },

    nfswChange() {
      this.initialLoadPost();
    },

    toTop() {
      if (this.$route.name === "flow-post") {
        this.$router.push({ name: "flow" });
        this.initialLoadPost();
      }

      if (this.isBeginning) {
        this.initialLoadPost();
      }
    },

    handleSwipe(swiperEvent) {
      this.setActiveVideoIndex(swiperEvent.item.id);
      this.isBeginning = swiperEvent.isBeginning;
      this.setSliderIndex(swiperEvent.index);

      if (swiperEvent.isEnd || swiperEvent.slidesLeftToEnd <= this.thresholdForNewSlides) return this.loadMore();
    },

    setSliderIndex(currentIndex) {
      const storageSwiperIndex = parseInt(this.getStorageSliderIndex());
      const jump = currentIndex - storageSwiperIndex;

      if (jump > 1) {
        // prevent swiper change event on onmount when set last slider index
        this.setStorageSliderIndex(storageSwiperIndex);
      } else {
        this.setStorageSliderIndex(currentIndex);
      }
    },

    followingChange({ key }) {
      if (this.flowType === key) return;

      this.flowType = key;
      this.setStorageFlowType(this.flowType);

      this.scrollTop();
      this.initialLoadPost();
    },

    scrollTop() {
      this.$nextTick(function () {
        scrollTo({
          top: document.body.scrollHeight,
          behavior: "smooth",
        });
      });
    },

    async getContentSettings() {
      return this.waitRequest(() => {
        const url = `/profile/settings`;

        return this.$get(
          url,
          this.successSettingsCallback,
          this.failureSettingsCallback,
          this.checkErrors
        );
      });
    },

    async getContentSettingsCategories() {
      return this.waitRequest(() => {
        const url = `/collect/settings`;

        return this.$get(
          url,
          this.successSettingsListCallback,
          this.failureSettingsCallback,
          this.checkErrors
        );
      });
    },

    async getContentSettingsSex() {
      return this.waitRequest(() => {
        const url = `/collect/sex`;

        return this.$get(
          url,
          this.successSettingsSexCallback,
          this.failureSettingsCallback,
          this.checkErrors
        );
      });
    },

    successSettingsListCallback(data) {
      this.setSettings({ settings: data?.settings });
    },

    successSettingsSexCallback(data) {
      this.setSettingsSex({ settingsSex: data });
    },

    successSettingsCallback(data) {
      this.setContentSettings({ contentSettings: data?.userSettings });
    },

    failureSettingsCallback(value) {
      this.updateContentSettings(value);
    },
  },
};
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/vendors/_variables.scss";

.flow {
  $parent: &;

  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;

  z-index: 1020; // above left side desktop menu below right side modal menu

  @include media-breakpoint-up(sm) {
    position: relative;
    margin: 0 -15px; // remove container border on parent
  }

  &__header {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: 2;
    display: flex;
    box-sizing: content-box;
    flex-direction: row;

    justify-content: space-between;
    align-items: center;
    pointer-events: none;
    padding: em(34) em(13) 0;

    @include media-breakpoint-up(sm) {
      position: sticky;
      z-index: 1040;
      background-color: $white;
      padding: em(19) em(23) em(25);
      border-bottom: 1px solid $app-gray-3;
    }
  }

  &__toggle,
  &__tabs,
  &__search {
    pointer-events: auto;
  }

  &__tabs {
    padding-top: em(4);
    margin-right: em(32);
  }

  &__search {
    width: auto;
    color: $white;
  }

  &__desktop-content {
    display: none;

    @include media-breakpoint-up(sm) {
      display: block;
    }
  }

  &__mobile-content {
    position: relative;
    z-index: 1;
    flex: 1;
    max-height: calc(100% - 61px);
  }

  &__swiper {
    z-index: 4;
    height: 100% !important;
    will-change: tansform;

    @include media-breakpoint-up(sm) {
      display: none;
    }
  }

  &__player-wrapper {
    background-color: $black;
    width: 100%;
    height: 100%;
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

  &__footer {
    display: block;
    position: relative;
    flex: 0 0 auto;

    @include media-breakpoint-up(sm) {
      display: none;
    }
  }

  &__icon {
    &--search {
      width: em(19);
      height: em(19);
      filter: drop-shadow(#{$app-shadow});

      @include media-breakpoint-up(sm) {
        color: $black;
      }
    }
  }
}
</style>
