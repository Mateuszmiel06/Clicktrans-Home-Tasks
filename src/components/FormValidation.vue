<script setup>
import { ref, shallowRef } from "@vue/reactivity";
import { onMounted, watch } from "@vue/runtime-core";
import RadioButton from "./tasks/RadioButton.vue";
import Select from "./tasks/Select.vue";
import Textarea from "./tasks/Textarea.vue";
import TextInput from "./tasks/TextInput.vue";
import Loader from "./Loader.vue";
import Modal from "./Modal.vue";

const radioButton = shallowRef(RadioButton);
const select = shallowRef(Select);
const textarea = shallowRef(Textarea);
const textInput = shallowRef(TextInput);

const loading = ref(true);
const showModal = ref(false);
const fields = ref({
  description: {
    label: "Description",
    fieldComp: textarea,
    maxLength: 255,
    value: "",
    errorMsg: "",
    validate: function () {
      this.errorMsg = "";
      if (!this.value) {
        this.errorMsg = "Text is required";
        return false;
      } else if (this.value.length > this.maxLength) {
        this.errorMsg = `You can’t enter more than 255 characters`;
        return false;
      }
      return true;
    },
  },
  confirmation: {
    label: "Send confirmation",
    fieldComp: radioButton,
    value: null,
    errorMsg: "",
    validate: function () {
      this.errorMsg = "";
      if (this.value === null) {
        this.errorMsg = "Text is required";
        return false;
      }
      return true;
    },
  },
  vat: {
    label: "VAT",
    fieldComp: select,
    placeholder: "Choose VAT",
    options: [
      { value: 19, text: "19%" },
      { value: 21, text: "21%" },
      { value: 23, text: "23%" },
      { value: 25, text: "25%" },
    ],
    value: null,
    errorMsg: "",
    validate: function () {
      this.errorMsg = "";
      if (!this.value) {
        this.errorMsg = "Text is required";
        return false;
      }
      return true;
    },
  },
  priceNetto: {
    label: "Price netto EUR",
    fieldComp: textInput,
    disabled: true,
    disabledTooltip: "First choose VAT",
    value: null,
    errorMsg: "",
    validate: function () {
      this.errorMsg = "";
      if (!this.value || Number.isNaN(this.value * 1)) {
        this.errorMsg = "Please, input number";
        return false;
      }
      return true;
    },
  },
  priceBrutto: {
    label: "Price brutto EUR",
    fieldComp: textInput,
    disabled: true,
    disabledTooltip: "You can't edit this field",
    value: null,
  },
});

let modalProps = {};
let activeValidation = false;

const validateFields = () => {
  let isFormValid = true;
  for (const field of Object.values(fields.value)) {
    if (field.validate && !field.validate()) {
      isFormValid = false;
    }
  }
  return isFormValid;
};

const mockRequest = async function () {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      Math.round(Math.random())
        ? reject({ status: 500, statusText: "Internal Server Error" })
        : resolve({ status: 200, statusText: "OK" });
    }, 700);
  });
};

const handleSubmit = async () => {
  loading.value = true;
  activeValidation = true;
  if (validateFields()) {
    await mockRequest()
      .then((res) => {
        modalProps = {
          message: "The form has been successfully sent.",
          success: true,
        };
      })

      .catch((err) => {
        modalProps = {
          message: "Error while submitting form!",
          error: true,
        };
      })
      .finally(() => {
        showModal.value = true;
      });
  }
  loading.value = false;
};

watch(
  () => [
    ...Object.values(fields.value).map(
      (item) => !item.label.includes("brutto") && item.value
    ),
  ],
  () => {
    const { vat, priceNetto, priceBrutto } = fields.value;
    if (vat.value && priceNetto.disabled) {
      priceNetto.disabled = false;
    } else if (priceNetto.value && priceNetto.validate()) {
      priceBrutto.value = +(priceNetto.value * (vat.value / 100 + 1)).toFixed(
        2
      );
    } else {
      priceBrutto.value = "";
    }

    if (activeValidation) {
      validateFields();
    }
  }
);
onMounted(() => {
  loading.value = false;
});
</script>

<template>
  <Loader v-if="loading"></Loader>
  <Modal v-else-if="showModal" v-bind="modalProps" @closeModal="() => (showModal = false)" />
  <form id="form" class="form" v-else>
    <h1>Clicktrans - Home task</h1>
    <component v-for="(field, key) in fields" :key="key" :is="field.fieldComp" v-bind="field" v-model="field.value" />
    <button type="submit" form="form" value="Submit" class="a-formButton" @click.prevent="handleSubmit">
      Submit
    </button>
  </form>
</template>

<style lang="scss">
.form {
  width: 70vw;
  max-width: 550px;
}

.a-form {
  &Input {
    margin-bottom: 1.5rem;
    display: flex;
    flex-flow: column nowrap;
    align-items: flex-start;

    &__label {
      font-size: 1.4rem;
    }

    &__helpers {
      margin-top: 0.45rem;
      width: 100%;
      display: flex;
      flex-flow: row nowrap;
      justify-content: space-between;
      font-size: 1rem;
    }

    &__option {
      display: flex;
      margin: 0.2rem;

      &Label {
        padding-left: 0.5rem;
      }
    }

    &__error {
      font-weight: 600;
      color: var(--error-color);
    }

    &__textarea,
    &__select,
    &__textInput {
      width: 100%;
    }

    &.-invalid {

      textarea,
      input {
        border-color: var(--error-color);
      }
    }
  }

  &Button {
    background-color: var(--extra-color);
    width: 100%;
    color: white;
    padding: 10px;
    font-size: 16px;
  }
}
</style>
