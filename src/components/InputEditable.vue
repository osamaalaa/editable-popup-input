<!-- src/components/FleetEditable.vue -->
<template>
  <span class="parent">
    <template v-if="fieldType == 'units'">
      <p class="text-start m-0 pointer-cursor font-bold text-primary" style="display:inline" v-if="!isDisabled"
        @click="() => { editedValue = value; show = !show; }">
        {{ value && value.abbreviation ? value.abbreviation : null }}&nbsp;
      </p>
      <p class="text-start m-0" style="display:inline;" v-else-if="visible">
        <span>{{ value && value.abbreviation ? value.abbreviation : null }}</span>&nbsp;
      </p>
    </template>
    <template v-else>
      <slot name="rendered-value" v-bind="{value,classes:!isDisabled ? 'pointer-cursor font-bold text-primary':'',onClick:isDisabled?()=>{}:() => { editedValue = value; show = !show; }}">
        <div v-if="renderValue" :class="!isDisabled ? 'pointer-cursor font-bold text-primary':'text--disable'" @click="() => { if(isDisabled)return; editedValue = value; show = !show; }">
          {{ value ? displayLabel(value) : '--' }}
        </div>
        <template v-else>
          <p class="text-start m-0 pointer-cursor font-bold text-primary" :style="`color: ${textColor} !important`" style="display:inline;" v-if="!isDisabled && !currency"
            @click="() => { editedValue = value; show = !show; }">
            {{ value || (value == 0 && valueType == 'number') ? parseValue(value) : placeHolder }}&nbsp;
          </p>
          <p class="text-start m-0 pointer-cursor font-bold text-primary" style="display:inline" v-else-if="!isDisabled && currency"
            @click="() => { editedValue = value; show = !show; }">
            <span v-if="value || (value == 0 && valueType == 'number')">{{ value | currency }}</span><span v-else>{{ placeHolder }}</span>&nbsp;
          </p>
          <p class="text-start m-0" :style="`color: ${textColor} !important`" style="display:inline;" v-else-if="visible && !currency">
            {{ value ? parseValue(value) : placeHolder }}&nbsp;
          </p>
          <p class="text-start m-0" :style="`color: ${textColor} !important`" style="display:inline;" v-else-if="visible && currency">
            <span v-if="value">{{ value | currency }}</span><span v-else>{{ placeHolder }}</span>&nbsp;
          </p>
        </template>
      </slot>
    </template>

    <div class="popup p-2 shadow rounded white" :style="popupExtraStyle" style="background: white;" :class="[className != '' ? className : '']" v-if="show">
      <template v-if="fieldType == 'radio' && options && Object.keys(options).length > 0 && label !== ''">
        <div class="radio-group mt-0">
          <label>{{ label }}</label>
          <div v-for="(option, i) in options" :key="i">
            <input type="radio" :value="option.value" v-model="editedValue">
            <span>{{ option.label }}</span>
          </div>
        </div>
      </template>
      <template v-else-if="fieldType == 'select' && options && Object.keys(options).length > 0 && label !== ''">
        <select v-model="editedValue" class="form-select">
          <option v-for="(option, i) in options.items" :value="option.value" :key="i">{{ option[options.labelKey || 'label'] }}</option>
        </select>
      </template>
      <template v-else-if="fieldType == 'number'">
        <input type="number" :min="min" :max="max" v-model="editedValue" @input="validateNumber" class="form-control" />
      </template>
      <template v-else>
        <input :type="valueType" v-model="editedValue" @input="validateRequired" class="form-control" />
      </template>
      <div class="d-flex justify-content-end mt-3">
        <button @click="cancel" class="btn btn-secondary">Cancel</button>
        <button :disabled="!isValid" @click="update" class="btn btn-primary">Update</button>
      </div>
    </div>
  </span>
</template>

<script>
// import { getUnitTypes } from "@/directives/units.js";
export default {
  name: "input-editable",
  props: {
    label: { type: String, default: "" },
    value: null,
    extraValueItem: { type: String, default: "OPERATION" },
    className: { type: String, default: "" },
    options: Object,
    extraOptions: { type: Array, default: () => [] },
    fieldType: { type: String, default: "" },
    searchable: { type: Boolean, default: true },
    extraValue: { type: Boolean, default: false },
    extraLabel: { type: String, default: "" },
    taggable: { type: Boolean, default: false },
    extrasWithStyle: { type: Boolean, default: false },
    valueType: { type: String, default: "" },
    placeHolder: { type: String, default: "" },
    textColor: { type: String, default: "" },
    disabled: { type: Boolean, default: false },
    visible: { type: Boolean, default: true },
    currency: { type: Boolean, default: false },
    groupValues: { type: String, default: "" },
    groupLabel: { type: String, default: "" },
    callback: { type: String, default: "" },
    styles: { type: String, default: "" },
    changingLabelKey: { type: Boolean, default: false },
    min: { type: Number },
    max: { type: Number },
    noResult: { type: String, default: "No Elements Found" },
    renderValue: { type: Boolean, default: false }
  },
  data() {
    return {
      show: false,
      isValid: true,
      disabledInternal: null,
      editedExtraValue: "",
      editedValue: "",
      editedValueSelected: null
    };
  },
  computed: {
    isEmptyString() {
      const notText = ['units', 'number', 'select'];
      if (!notText.includes(this.fieldType))
        return this.editedValue?.trim?.() === '';
      else return false;
    },
    isDisabled() {
      return this.disabledInternal !== null ? this.disabledInternal : this.disabled;
    },
    popupExtraStyle() {
      if (this.extraValue && !this.extrasWithStyle) {
        return {
          width: 'max-content',
          left: '35px'
        };
      } else {
        return this.styles;
      }
    }
  },
  watch: {
    editedValue: {
      handler(val) {
        this.validateNumber();
        if (this.fieldType == 'units' && val && typeof val == 'object' && val !== null && val !== undefined && ((typeof this.value == 'object' && Object.keys(this.value).length == 0) || this.value == null || this.value == undefined)) {
          this.$emit("update", val, this.editedExtraValue);
          this.$emit("updateExtra", this.editedExtraValue);
        }
      }
    },
    visible: {
      handler(val) {
        this.show = val ? true : false;
      }
    },
    show: {
      handler(val) {
        if (this.show) this.editedValue = (this.valueType == 'number') ? parseFloat(this.value) : this.value;

        if (val) {
          setTimeout(() => {
            if (this.$refs && this.$refs['editable-input']) {
              this.$refs['editable-input'].focus();
            }
          }, 100);
        }
      }
    }
  },
  methods: {
    displayLabel(val) {
      return this.options.items.find(option => option.value == val)?.[this.options.labelKey || "label"] || "--";
    },
    validateNumber() {
      this.isValid = true;
      if (this.fieldType == 'number') {
        if (this.editedValue < 0 || this.editedValue === 0 || this.editedValue == null || isNaN(this.editedValue) || !this.editedValue) {
          this.isValid = false;
        } else if (this.min !== undefined && this.max !== undefined) {
          if (this.editedValue < this.min || this.editedValue > this.max) {
            this.isValid = false;
          } else {
            this.isValid = true;
          }
        }
      }
    },
    validateRequired() {
      if (!this.editedValue || this.editedValue.trim() === '') {
        this.isValid = false;
      } else {
        this.isValid = true;
      }
    },
    parseValue(value) {
      if (this.fieldType == 'select') {
        return typeof value == 'object' ? value[this.options.key] : value;
      } else {
        return value;
      }
    },
    update() {
      this.$nextTick(() => {
        if (this.valueType == 'number') {
          this.editedValue = parseFloat(this.editedValue);
        }
        if (this.callback !== '') {
          this.$emit(this.callback, this.editedValue);
        } else {
          this.$emit("update", this.editedValue, this.editedExtraValue);
          this.$emit("updateExtra", this.editedExtraValue);
        }
      });
      this.show = false;
    },
    cancel() {
      this.$emit("cancel", false);
      this.show = this.show ? false : true;
    }
  },
  mounted() {
    this.editedExtraValue = this.extraValueItem;
  
      this.editedValue = (this.valueType == 'number') ? parseFloat(this.value) : this.value;
    
  }
};
</script>

<style>
.setting-multiselect {
  width: 100% !important;
}
.setting-multiselect .multiselect__tags .multiselect__tag {
  background: #d2caca !important;
  color: initial;
}
.parent {
  position: relative;
}
.popup {
  position: absolute;
  top: 30px !important;
  left: 0;
  z-index: 999999999;
  transform: translate(-30%, -102%);
}
.pointer-cursor {
  cursor: pointer;
}
.text--disable {
  color: #9e9e9e !important;
}
.form-select {
  width: 100%;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.form-control {
  width: 100%;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.btn {
  padding: 5px 10px;
  margin-left: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.btn-primary {
  background-color: #007bff;
  color: white;
}
.btn-secondary {
  background-color: #6c757d;
  color: white;
}
.d-flex {
  display: flex;
}
.justify-content-end {
  justify-content: flex-end;
}
.mt-3 {
  margin-top: 1rem;
}
.mt-2 {
  margin-top: 0.5rem;
}
.mt-0 {
  margin-top: 0;
}
</style>
