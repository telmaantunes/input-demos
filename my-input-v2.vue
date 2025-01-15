<template>
  <div
    class="my-input"
    :class="extraClass"
  >
    <my-title :title="title">
      <slot
        slot="top"
        name="top"
      />
    </my-title>

    <div
      class="my-input__element"
      @click.self.prevent.stop="onClick"
    >
      <div
        class="my-input__icons-left"
        @click.prevent.stop="onClick"
      >
        <MyIcon
          v-if="isPicker && isTime"
          name="clock"
        />
        <MyIcon
          v-else-if="isPicker"
          name="calendar"
        />
        <slot name="icons-left" />
      </div>

      <Datetime
        v-if="isPicker"
        ref="input"
        :value="value"
        v-bind="attributes"
        v-on="inputListeners"
      />
      <template v-else>
        <span
          v-if="isReadonly"
          class="my-input__input"
          @click.self.prevent.stop="onClick"
        >
          {{ formattedValue }}
        </span>
        <input
          ref="input"
          class="my-input__input"
          :class="{'my-input__input--hidden': isReadonly}"
          :value="formattedValue"
          v-bind="attributes"
          v-on="inputListeners"
          @keypress.stop.self="onKeypress"
        >
      </template>
      <div
        class="my-input__icons-right"
        @click.prevent.stop="onClick"
      >
        <MyIcon
          v-if="showSuccess"
          class="my-input__success"
          name="check-simple"
        />

        <MyIcon
          v-if="showError"
          class="my-input__error"
          name="close"
        />

        <MyIcon
          v-if="isLoading"
          name="loading"
        />

        <MyIcon
          v-if="isPercentage"
          name="percent"
        />

        <slot
          name="icons-right"
        />

        <button
          v-if="isPassword"
          type="button"
          class="my-input__password"
          @click.prevent="togglePassword"
        >
          <MyIcon
            v-if="showPassword"
            name="password-off"
          />
          <MyIcon
            v-else
            name="password"
          />
        </button>
      </div>
    </div>

    <slot />
  </div>
</template>

<script>
import isNaN from 'lodash/isNaN';
import isNil from 'lodash/isNil';
import { Datetime } from 'vue-datetime';
import InputTypeEnum from '../../enums/input-type-enum';
import MyIcon from '../my-icon';
import MyTitle from '../my-title';

const dateTypes = [InputTypeEnum.DATE, InputTypeEnum.DATETIME, InputTypeEnum.TIME];

export default {
  name: 'MyInput',
  components: {
    Datetime,
    MyIcon,
    MyTitle,
  },
  inheritAttrs: false,
  props: {
    id: {
      type: String,
      default: undefined,
    },
    name: {
      type: String,
      default: undefined,
    },
    value: {
      type: [String, Number],
      default: undefined,
    },
    type: {
      type: String,
      default: 'text',
      validator(value) {
        return InputTypeEnum.values().indexOf(value) !== -1;
      },
    },
    title: {
      type: String,
      default: undefined,
    },
    maxlength: {
      type: [String, Number],
      default: undefined,
    },
    placeholder: {
      type: String,
      default: undefined,
    },
    isRequired: {
      type: Boolean,
      default: false,
    },
    isReadonly: {
      type: Boolean,
      default: false,
    },
    isDisabled: {
      type: Boolean,
      default: false,
    },
    isLoading: {
      type: Boolean,
      default: false,
    },
    isUnderlined: {
      type: Boolean,
      default: false,
    },
    isFocused: {
      type: Boolean,
      default: false,
    },
    showError: {
      type: Boolean,
      default: false,
    },
    showSuccess: {
      type: Boolean,
      default: false,
    },
    /**
     * Behaves as the isFloat (which means it allows cents) but
     * visually adds the dollar sign
     *
     * Combine with the isInteger option to return an integer and
     * disable all non-numeric characters (which means it disables cents)
     */
    isCurrency: {
      type: Boolean,
      default: false,
    },
    /**
     * Disables all non-numeric character and parses the string to an integer
     */
    isInteger: {
      type: Boolean,
      default: false,
    },
    /**
     * Disables all non-numeric characters except the '.' character and
     * parses the string to a float
     */
    isFloat: {
      type: Boolean,
      default: false,
    },
    /**
     * Behaves as the isFloat (which means it allows cents) but
     * visually adds the percent sign
     */
    isPercentage: {
      type: Boolean,
      default: false,
    },
    currencyOptions: {
      type: Object,
      default() {
        return {
          locale: 'en-AU',
          currency: 'AUD',
          minimumFractionDigits: 0,
        };
      },
    },
    datetimeFormat: {
      type: String,
      default: undefined,
    },
    datetimeZone: {
      type: String,
      default: 'local',
    },
    datetimeValueZone: {
      type: String,
      default: 'UTC',
    },
    /**
     * Customize steps flow, steps available: time, date, month, year.
     *
     * Example to show year first on a dob: ['year', 'date']
     */
    datetimeFlow: {
      type: Array,
      default: undefined,
    },
    /**
     * Mostly for demo & storybook purposes
     */
    datetimeStartOpen: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      inputType: this.type,
      showPassword: false,
      focused: this.isFocused,
    };
  },

  computed: {
    attributes() {
      if (this.isPicker) {
        return {
          type: this.inputType,
          'input-id': this.id,
          'hidden-name': this.name,
          'use12-hour': true,
          auto: true,
          flow: this.datetimeFlow,
          'value-zone': this.datetimeValueZone,
          zone: this.datetimeZone,
        };
      }

      return {
        ...this.$attrs,
        type: this.inputType,
        id: this.id,
        name: this.name,
        disabled: this.isDisabled,
        required: this.isRequired,
        placeholder: this.placeholder,
        readonly: this.isReadonly,
        maxlength: this.maxlength,
      };
    },

    extraClass() {
      return {
        'my-input--error': this.showError,
        'my-input--success': this.showSuccess,
        'my-input--focused': this.focused,
        'my-input--disabled': this.isDisabled,
        'my-input--picker': this.isPicker,
        'my-input--underlined': this.isUnderlined,
      };
    },

    isPicker() {
      return dateTypes.indexOf(this.type) !== -1;
    },

    isTime() {
      return this.type === InputTypeEnum.TIME;
    },

    isPassword() {
      return this.type === 'password';
    },

    inputListeners() {
      const listeners = { ...this.$listeners };
      delete listeners.change; // Change should not be bound externally

      return Object.assign(listeners, {
        input: this.onInput,
        focus: this.onFocus,
        blur: this.onBlur,
      });
    },

    formattedValue() {
      if (isNil(this.value) || this.value === '') {
        return this.value;
      }

      if (this.isCurrency) {
        const formatter = new Intl.NumberFormat(this.currencyOptions.locale, {
          style: 'currency',
          currency: this.currencyOptions.currency,
          minimumFractionDigits: this.currencyOptions.minimumFractionDigits,
        });

        return formatter.format(this.value);
      }

      if (this.isFloat || this.isPercentage) {
        const formatter = new Intl.NumberFormat(this.currencyOptions.locale, {
          maximumFractionDigits: 2,
        });

        return formatter.format(this.value);
      }

      return this.value;
    },

    isNumeric() {
      return this.isCurrency
       || this.isInteger
       || this.isFloat
       || this.isPercentage;
    },
  },

  mounted() {
    if (this.datetimeStartOpen) {
      this.onClick();
    }
  },

  methods: {
    onInput(event) {
      let value = this.isPicker ? event : event.target.value;

      if (this.isNumeric) {
        value = this.parseNumerics(value);
      }

      if (this.isInvalidNumber(value)) {
        value = null;
      }

      this.$emit('input', value);
    },

    isInvalidNumber(value) {
      if (value === 'NaN') {
        return true;
      }

      if (this.isNumeric) {
        return isNaN(value);
      }

      return false;
    },

    parseNumerics(value) {
      if (this.isInteger) {
        return parseInt(value.replace(/\D/g, ''), 10);
      }

      return parseFloat(value.replace(/[^\d.]/g, ''), 10);
    },

    onKeypress(event) {
      const pressedKey = event.key;

      if (pressedKey === 'Tab' || pressedKey === 'Enter') {
        this.onClick();

        return true;
      }

      if (this.isInteger && !this.isNumericKey(pressedKey)) {
        event.preventDefault();

        return false;
      }

      if ((this.isCurrency || this.isPercentage || this.isFloat)
        && !this.isNumericKey(pressedKey, ['.'])) {
        event.preventDefault();

        return false;
      }

      return true;
    },

    isNumericKey(key, extraKeys = []) {
      const allowedKeys = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'];

      return allowedKeys.concat(extraKeys).includes(key);
    },

    onClick() {
      if (this.isDisabled) {
        return;
      }

      if (this.isPicker) {
        this.$refs.input.$el.firstElementChild.click();

        return;
      }

      this.$refs.input.focus();
      this.$refs.input.click();
    },

    onFocus(event) {
      if (this.isFocused) {
        return;
      }

      this.focused = true;
      this.$emit('focus', event);
    },

    onBlur(event) {
      this.focused = false;
      this.$emit('blur', event);
    },

    togglePassword() {
      this.showPassword = !this.showPassword;
      if (this.showPassword) {
        this.inputType = 'text';

        return;
      }

      this.inputType = 'password';
    },
  },
};
</script>

<style lang="scss">
@import '~vue-datetime/dist/vue-datetime.min.css';
@import '@design';

$initial: initial;
$my-input-height: 48px;
$my-label-height: 16px;
$component-palette: (
  'primary': color('text', 'form'),
  'border': color('border'),
  'text': color('text', 'secondary'),
  'placeholder': color('text', 'subtle'),
  'background': color('background')
);

/* Variables needed for underlined style */
$char-w: 1ch;
$gap: 0.5 * $char-w;
$n-char: 4;
$in-w: $n-char * ($char-w + $gap);
$char-size: 7ch;
$placeholder-font-size: 56px;

.my-input {
  width: 100%;
  min-height: $my-input-height;
  display: inline-block;
  position: relative;
  font-family: inherit;
  margin-bottom: 16px;

  /* stylelint-disable-next-line */
  .vdatetime {
    display: inline-block;
  }

  .vdatetime-input {
    @include component;

    width: 100%;
    padding: 0 8px;
  }

  .vdatetime-popup {
    background: color('background', 'dark');
    color: color('text', 'negative');
    -webkit-text-fill-color: color('text', 'negative');
  }

  .vdatetime-popup__header {
    background: color('background', 'dark');
    color: color('text', 'negative');
    -webkit-text-fill-color: color('text', 'negative');
  }

  .vdatetime-calendar__month__day--selected span > span,
  .vdatetime-calendar__month__day:hover span > span {
    background: color('background', 'dark', 80%);
  }

  .vdatetime-popup__actions__button,
  .vdatetime-time-picker__item--selected {
    color: color('primary');
  }

  .vdatetime-calendar__navigation--previous,
  .vdatetime-calendar__navigation--next {
    svg,
    path {
      stroke: color('background');
    }
  }

  &__element {
    @include animate;
    @include border-radius;
    @include animate($transition-default-duration ease-in);

    position: relative;
    display: flex;
    align-items: center;
    height: $my-input-height;
    transition-property: color, border;
    border: 1px solid transparent;
    border-color: component-color('border');
    color: component-color();
    -webkit-text-fill-color: component-color();
    background: component-color('background');

    &:hover {
      background: color('background', 'barely');
    }

    .my-icon {
      width: 16px;
      height: 16px;
      fill: component-color();
      stroke: component-color();
      margin: 12px;

      + .my-icon {
        margin-left: 0;
      }
    }
  }

  &__input {
    @include component;

    width: 100%;
    padding: 12px;
    display: inline-block;
    color: inherit;
    transition-property: font-size, padding-top, color, border;

    .my-icon {
      background: none;
    }

    &--hidden {
      width: 0;
      height: 0;
      padding: 0;
    }
  }

  input {
    -webkit-text-fill-color: inherit;
    box-shadow: 0 0 transparent; // firefox required fix
    background: none;
    border: none;

    &:focus {
      outline: none;
      background: none;
    }

    &::-webkit-input-placeholder,
    &::placeholder {
      @include component(component-color('placeholder'));

      -webkit-text-fill-color: component-color('placeholder');
      text-shadow: none;
    }

    &:-webkit-autofill {
      @include component;

      background: none;
      color: inherit;
      -webkit-text-fill-color: inherit;
      text-shadow: none;
    }
  }

  &--has-icon {
    input {
      padding-right: $my-input-height;
    }
  }

  &__label {
    @include label-form;
  }

  &__password {
    cursor: pointer;
    background: none;
    border: none;
    margin: 0;
    padding: 0;
  }

  &__success.my-icon {
    fill: color('feedback', 'positive');
    stroke: none;
  }

  &__error.my-icon {
    fill: color('feedback', 'negative');
    stroke: color('feedback', 'negative');
  }

  &--focused {
    .my-input__element {
      border-color: component-color();
      transition-property: border;
      color: component-color();
      -webkit-text-fill-color: component-color();
      background: color('background', 'barely');

      &:hover {
        background: color('background', 'barely');
      }
    }
  }

  &--disabled {
    opacity: 0.5;

    .my-input__element {
      &:hover {
        background: color('background');
      }
    }

    input {
      cursor: default;
      pointer-events: none;
    }
  }

  &--underlined {
    .my-input__element {
      background: transparent;
      border: none;
    }

    // this makes the characters be same-width and underlined (e.g. mobile verification)
    input {
      height: auto;
      padding: 0;
      border: none;
      width: $in-w;
      /* stylelint-disable-next-line */
      background:
        repeating-linear-gradient(
          90deg,
          component-color('border') 0,
          component-color('border') $char-w,
          transparent 0,
          transparent $char-w + $gap
        )
        0 100% / 100% 3px no-repeat;
      font: $char-size consolas, monospace;
      letter-spacing: $gap;
      border-radius: 0;

      &::placeholder {
        font-size: 100%;
      }

      &:focus {
        outline: none;
        color: component-color();

        /* stylelint-disable-next-line */
        background:
          repeating-linear-gradient(
            90deg,
            component-color('border') 0,
            component-color('border') $char-w,
            transparent 0,
            transparent $char-w + $gap
          )
          0 100% / 100% 3px no-repeat;
      }
    }
  }

  &--success {
    &.my-input--underlined {
      input {
        color: component-color();

        /* stylelint-disable-next-line */
        background:
          repeating-linear-gradient(
            90deg,
            component-color() 0,
            component-color() $char-w,
            transparent 0,
            transparent $char-w + $gap
          )
          0 100% / 100% 3px no-repeat;
      }
    }
  }

  &__icons-left,
  &__icons-right {
    height: 100%;
    width: auto;
    display: inline-flex;
    align-items: center;
  }
}
</style>
