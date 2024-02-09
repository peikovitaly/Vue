<template>
  <div>
    <div class="flex items-center relative">
      <div>
        <div
          class="phoneCode flex items-center cursor-pointer"
          @click="isOpenList = !isOpenList"
        >
          <img
            :src="iconUrl(activeCountry.icon)"
            class="img-country margin-right-2"
            alt="flag"
          />
          <div class="code margin-right-2" v-text="activeCountry.phone" />
          <IconArrow class="icon-chevron" :class="{ open: isOpenList }" />
        </div>
        <div
          v-if="isOpenList"
          v-on-clickaway="onClickOutside"
          class="country-list"
        >
          <div class="margin-x-3 margin-y-2">
            <input
              v-model.trim="search"
              type="text"
              class="input-search"
              :placeholder="$t('Search')"
            />
          </div>
          <div class="country-list__search">
            <div
              v-for="item in countryListAutocomplete"
              :key="item.id"
              class="country-list__item"
              :class="{ active: item.alpha2 === phoneCode }"
              @click="changeCountry(item.alpha2)"
            >
              <img
                class="img-country margin-right-2"
                :src="iconUrl(item.icon)"
                alt="flag"
              />
              <div class="code margin-right-2" v-text="item.phone" />
              <div v-text="item.name" />
            </div>
          </div>
        </div>
      </div>

      <input
        v-model="number"
        v-maska="mask"
        type="tel"
        name="phone"
        :placeholder="placeholder"
        class="input-fio"
        :class="{ error: errorText.number }"
        :disabled="isLoading"
        @input="inputMask"
        @keyup="errorValidation('number')"
        @blur="$emit('change-number', number)"
      />
    </div>
    <span
      v-if="errorText.number"
      class="error-message"
      v-text="errorText.number"
    />
  </div>
</template>

<script>
import {
  isPossiblePhoneNumber,
  isValidPhoneNumber,
  Metadata,
  validatePhoneNumberLength,
} from 'libphonenumber-js';
import arr from './dataset.js';

export default {
  name: 'PhoneNumberInput',
  components: {
    IconArrow: () => import('@/assets/icons/lang-arrow.svg?inline'),
  },
  props: {
    defaultCountry: { type: String, default: '' },
    defaultNumber: { type: String, default: '' },
  },
  data() {
    return {
      isLoading: false,
      isOpenList: false,
      phoneCode: '',
      phoneCodeNumber: '',
      number: '',
      search: '',
      lengthNumber: '',
      mask: '',
      testNumber: '',
      placeholder: '',
      errorText: {
        number: '',
      },
    };
  },
  computed: {
    listCountry: () => arr,
    activeCountry: (self) =>
      self.listCountry.filter((el) => el.alpha2 === self.phoneCode)[0],
    countryListAutocomplete: (self) => {
      if (!self.search) return self.listCountry;
      return self.listCountry.filter((o) => {
        if (
          o.name.toLowerCase().includes(self.search.toLowerCase()) ||
          o.alpha2.toLowerCase().includes(self.search.toLowerCase()) ||
          o.phone.toLowerCase().includes(self.search.toLowerCase())
        ) {
          return true;
        }
        return false;
      });
    },
    isValidationNumber: (self) =>
      self.isPossiblePhoneNumber && self.isValidPhoneNumber,
    isPossiblePhoneNumber: (self) =>
      isPossiblePhoneNumber(self.number, self.phoneCode),
    isValidPhoneNumber: (self) =>
      isValidPhoneNumber(self.number, self.phoneCode),
    validatePhoneNumberLength: (self) =>
      validatePhoneNumberLength(self.number, self.phoneCode),
  },
  watch: {
    defaultCountry: {
      handler() {
        const initLocal = 'UA';
        this.phoneCode = this.defaultCountry ? this.defaultCountry : initLocal;
        this.phoneCodeNumber = this.activeCountry.phone;
        const form = {
          alpha2: this.phoneCode,
          phone: this.phoneCodeNumber,
        };
        this.$emit('change-country', form);
        this.initMask();
      },
      immediate: true,
      deep: true,
    },
    defaultNumber: {
      handler() {
        this.mask = '### ### ### ### ###';
        setTimeout(() => {
          this.mask =
            this.phoneCode === 'UA'
              ? this.maskList('UA')
              : this.maskList(this.lengthNumber);
          this.number = this.defaultNumber;
        }, 10);
      },
      immediate: true,
      deep: true,
    },
  },
  mounted() {},
  methods: {
    onClickOutside() {
      this.isOpenList = false;
      this.search = '';
    },
    errorValidation(name) {
      if (this[name].length < 1) {
        this.errorText[name] = '';
        this.$emit('changeErrorText', this.errorText[name]);
        return;
      }
      if (!this.isValidationNumber) {
        this.errorText[name] = this.$t('Invalid number');
        this.$emit('changeErrorText', this.errorText[name]);
        return;
      }
      this.errorText[name] = '';
      this.$emit('changeErrorText', this.errorText[name]);
    },
    initMask() {
      const number = new Metadata().metadata.countries[this.phoneCode][3];
      this.lengthNumber = number[number.length - 1];
      this.mask =
        this.phoneCode === 'UA'
          ? this.maskList('UA')
          : this.maskList(this.lengthNumber);
      this.placeholder =
        this.phoneCode === 'UA'
          ? this.maskList('UA')
          : this.maskList(this.lengthNumber);
      this.placeholder =
        this.phoneCode === 'UA'
          ? this.maskList('UA')
          : this.maskList(this.lengthNumber);
    },
    changeCountry(alpha2) {
      this.phoneCode = alpha2;
      this.phoneCodeNumber = this.activeCountry.phone;
      const form = {
        alpha2: this.phoneCode,
        phone: this.phoneCodeNumber,
      };
      this.initMask();
      this.errorValidation('number');
      this.isOpenList = false;
      this.search = '';
      this.$emit('change-country', form);
    },
    iconUrl(name) {
      return `./country/${name}.svg`;
    },
    inputMask() {
      const amount = this.number.slice(0, this.mask.length);
      this.number = amount;
    },
    maskList(key) {
      switch (key) {
        case 5:
          return '### ##';
        case 6:
          return '### ###';
        case 7:
          return '### ### #';
        case 8:
          return '### ### ##';
        case 9:
          return '### ### ###';
        case 10:
          return '### ### ### #';
        case 11:
          return '### ### ### ##';
        case 12:
          return '### ### ### ###';
        case 13:
          return '### ### ### ### 0';
        case 14:
          return '### ### ### ### ##';
        case 15:
          return '### ### ### ### ##';
        case 'UA':
          return '## ### ## ##';
        default:
          return '### ### ### ###';
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.phoneCode {
  padding: 0.77rem 1.8rem 0.77rem 1.2rem;
  background: #ebeff3;
  color: $color-dark;
  border: 1px solid #ebeff3;
  border-right: 1px solid $color-white;
  border-radius: 3px 0 0 3px;
  font-size: 0.9rem;
}
.input-fio {
  width: 100%;
  background: #ebeff3;
  outline: none;
  padding: 0.77rem 1.8rem 0.77rem 1.2rem;
  border: 1px solid #ebeff3;
  box-sizing: border-box;
  border-radius: 0 0.3rem 0.3rem 0;
  color: $color-dark;
  font-size: 0.9rem;

  &::placeholder {
    color: $color-default;
  }
  &.error {
    border: 1px solid $color-red;
  }
}
.img-country {
  width: 20px;
}
.code {
  width: 3.5rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.country-list {
  position: absolute;
  top: 100%;
  left: 0;
  display: block;
  width: 100%;
  margin: 0;
  background: #ebeff3;
  color: $color-dark;
  border: 1px solid #ebeff3;
  border-top-style: none;
  border-radius: 0 0 4px 4px;
  box-shadow: 0 3px 6px 0 rgb(0 0 0 / 15%);
  z-index: 9;
  &__item {
    display: flex;
    align-items: center;
    padding: 0.5rem 1rem;
    cursor: pointer;
    transition: all 0.5s ease;
    &.active,
    &:hover {
      background: $color-lite-gray;
    }
  }
  &__search {
    max-height: 20rem;
    font-size: 14px;
    box-sizing: border-box;
    overflow-y: auto;
  }
  .code {
    width: 4rem;
  }
}
.icon-chevron {
  min-width: 0.8rem;
  width: 0.8rem;
  height: 0.8rem;
  fill: #999faa;
  transition: all 0.5s ease;
  &.open {
    transform: rotate(-180deg);
  }
}
.input-search {
  width: 100%;
  padding: 0.535rem 0.85rem;
  background: #ebeff3;
  color: $color-dark;
  border: none;
  border-bottom: 1px solid #999faa;
  &::placeholder {
    color: $color-dark;
  }
}
.error-message {
  display: block;
  font-size: 12px;
  line-height: 14px;
  color: $color-red;
  text-align: right;
}
</style>
