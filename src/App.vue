<template>
  <div id="app">
    <h1>Editable Component Demo</h1>
    <h2>Text Field</h2>
    <input-editable
      label="Text Field"
      :value="textValue"
      fieldType="text"
      :valueType="'text'"
      :placeHolder="'Enter text'"
      :styles="'minWidth: 215px; marginLeft: 200px'"
      @update="updateValue"
    />
    <h2>Select Field</h2>
    
    <input-editable
      label="Select Field"
      :value="selectValue"
      fieldType="select"
      :options="selectOptions"
      :placeHolder="'Select an option'"
      :styles="'minWidth: 215px; marginLeft: 200px'"
      @update="updateValue"
    /> 
  </div>
</template>

<script>
import InputEditable from './components/InputEditable.vue';

export default {
  name: 'App',
  components: {
    InputEditable
  },
  data() {
    return {
      textValue: '',
      selectValue: '',
      selectOptions: {
        items: [
          { value: 'option1', label: 'Option 1' },
          { value: 'option2', label: 'Option 2' },
          { value: 'option3', label: 'Option 3' }
        ],
        labelKey: 'label'
      }
    };
  },
  methods: {
    updateValue(newValue) {
      console.log('Updated value:', newValue);
    }
  }
};
</script>
