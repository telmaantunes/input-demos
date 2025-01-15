<template>
  <div class="input-block">
    <TitleItem v-if="props.title || $slots.title">
      {{ props.title }}<span v-if="!props.required"> (opcional)</span>
      <slot name="title" />
    </TitleItem>
    <v-text-field
      v-model="internalModel"
      v-maska:[maskOptions]
      v-bind="$attrs"
      :name="props.name"
      :placeholder="props.placeholder"
      :rules="props.rules"
      :disabled="props.disabled"
      :prefix="props.prefix"
      :sufix="props.sufix"
      density="compact"
      rounded="4px"
      base-color="blue-dark"
      bg-color="white"
      color="blue-dark"
      variant="outlined"
      @update:model-value="updateModel"
    >
      <template v-for="(_, slotName) in $slots" #[slotName]="slotData">
        <slot :name="slotName" v-bind="slotData" />
      </template>
    </v-text-field>

    <slot />
  </div>
</template>

<script setup>
import { vMaska } from 'maska';
import { AsYouType } from 'libphonenumber-js';

const props = defineProps({
  name: {
    type: String,
    default: undefined,
    description: 'The name of the input'
  },
  prefix: {
    type: String,
    default: undefined,
    description: 'The name of the input'
  },
  sufix: {
    type: String,
    default: undefined,
    description: 'The name of the input'
  },
  title: {
    type: String,
    required: false,
    default: undefined,
    description: 'The title of the input'
  },
  isLoading: {
    type: Boolean,
    required: false,
    default: false,
    description: 'The state of the loading option'
  },
  required: {
    type: Boolean,
    default: true,
    description: 'The state of the required option'
  },
  placeholder: {
    type: String,
    required: false,
    default: undefined,
    description: 'The placeholder of the input'
  },
  disabled: {
    type: Boolean,
    default: false,
    description: 'The state of the disabled option'
  },
  rules: {
    type: Array,
    required: false,
    default: undefined,
    description: 'An array with a set of rules '
  },
  mask: {
    type: [String, Function],
    required: false,
    default: undefined,
    description: 'The mask to be applied to the input'
  },
  minDate: {
    type: String,
    required: false,
    default: undefined,
    description: 'The minimum date used by the date picker input'
  },
  maxDate: {
    type: String,
    required: false,
    default: undefined,
    description: 'The maximum date used by the date picker input'
  },
  /**
   * Enforces format
   */
  isPhoneNumber: {
    type: Boolean,
    default: false
  },
  /**
   * Helper to format phone numbers
   */
  countryCode: {
    type: String,
    default: 'PT'
  },
  /**
   * Enforces format dd/mm/yyyy
   */
  isDate: {
    type: Boolean,
    default: false
  },
  /**
   * Disables all non-numeric character except the ',' character and
   * parses the string to an integer; Enforces format €
   */
  isCurrency: {
    type: Boolean,
    default: false
  },
  /**
   * Disables all non-numeric character and parses the string to an integer
   */
  isInteger: {
    type: Boolean,
    default: false
  },
  /**
   * Disables all non-numeric characters except the ',' character and
   * parses the string to a float; Enforces format
   */
  isFloat: {
    type: Boolean,
    default: false
  }
});

const model = defineModel({ type: [String, Number] });
const emit = defineEmits(['update:modelValue']);
const internalModel = defineModel('internalModel', { type: [String, Number] });

const maskOptions = computed(() => {
  let options = {};
  if (props.mask) {
    options.mask = props.mask;
  }

  if (props.isCurrency) {
    options = {
      postProcess: (value) => {
        if (!value) {
          return '';
        }
        value = value.toString().replace(/[^\d,]/g, '');
        let formattedValue = Intl.NumberFormat('es-ES', {
          style: 'currency',
          currency: 'EUR',
          minimumFractionDigits: 0
        }).format(value.replace(/,/g, '.'));

        if (value.endsWith(',')) {
          formattedValue = `${formattedValue.slice(0, -2)}, €`;
        }

        return formattedValue;
      }
    };
  }

  if (props.isFloat) {
    options = {
      postProcess: (value) => {
        if (!value) {
          return '';
        }

        value = value.toString().replace(/[^\d,]/g, '');
        let formattedValue = new Intl.NumberFormat('es-ES', {
          maximumFractionDigits: 2
        }).format(value.replace(/,/g, '.'));

        if (value > 9) {
          formattedValue = `${formattedValue},`;
        }
        if (value.endsWith(',')) {
          formattedValue = `${formattedValue},`;
        }

        return formattedValue;
      }
    };
  }

  if (props.isInteger) {
    options = {
      postProcess: (value) => {
        if (!value) {
          return '';
        }

        return value.toString().replace(/[^\d]/g, '');
      }
    };
  }

  if (props.isDate) {
    options = {
      mask: '##/##/####',
      eager: true
    };
  }

  if (props.isPhoneNumber) {
    options = {
      postProcess: (value) => {
        if (!value) {
          return '';
        }

        return new AsYouType(props.countryCode).input(value);
      }
    };
  }

  return options;
});

const isNumeric = computed(() => {
  return props.isCurrency || props.isInteger || props.isFloat;
});

function stringToNumber (value) {
  if (props.isInteger) {
    return parseInt(value.replace(/\D/g, ''), 10);
  }

  return parseFloat(
    value
      .toString()
      .replace(/[^\d,]/g, '')
      .replace(/,/g, '.'),
    10
  );
}

function numberToString (value) {
  if (!value) {
    return '';
  }

  value = value
    .toString()
    .replace(/\./g, ',')
    .replace(/[^\d,]/g, '');

  return value;
}

function updateModel (value) {
  let newValue = value;
  if (isNumeric.value) {
    newValue = stringToNumber(value);
  }

  if (props.isPhoneNumber) {
    newValue = parseInt(value.replace(/\D/g, ''), 10);
  }

  emit('update:modelValue', newValue);
}

function updateInternalModel (value) {
  let newValue = value;
  if (model.value && isNumeric.value) {
    newValue = numberToString(model.value);
  }

  internalModel.value = newValue;
}

watch(
  () => model.value,
  (value) => {
    updateInternalModel(value);
  }
);

onMounted(() => {
  updateInternalModel(model.value);
});
</script>

<style lang="scss" scoped>
@import "~/assets/styles/index";

.input-block {
  &__label {
    @include subtitle;

    margin-bottom: 8px;
  }

  :deep(.v-text-field) .v-input__details {
    padding: 4px 0 0;
    align-items: flex-start;
  }
}
</style>
