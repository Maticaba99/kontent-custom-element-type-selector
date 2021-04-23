<template>
  <div>
    <multiselect
      v-model="selectedTypes"
      placeholder="Select types"
      open-direction="bottom"
      :options="options"
      :multiple="true"
      :loading="isLoading"
      label="name"
      track-by="codename"
      :disabled="element.disabled"
      @input="onSelect"
    >
      <template slot="singleLabel" slot-scope="{ option }">
        <strong>{{ option.sys.name }}</strong>
      </template>
    </multiselect>
  </div>
</template>

<script>
import Multiselect from "vue-multiselect";

export default {
  components: {
    Multiselect
  },
  data() {
    return {
      selectedTypes: this.value || [],
      options: [],
      isLoading: true
    };
  },
  created() {
    this.fetchTypes();
  },
  props: {
    element: {
      type: Object,
      required: true
    },
    context: {
      type: Object,
      required: true
    },
    value: {
      type: Object
    }
  },
  methods: {
    limitText(count) {
      return `and ${count} other countries`;
    },
    fetchTypes() {
      fetch(
        `https://deliver.kontent.ai/${
          this.context.projectId
        }/items?depth=0&elements=null${
          this.element.config.filter ? "&" + this.element.config.filter : ""
        }`,
        {
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${this.element.config.secureAccess}`
          }
        }
      )
        .then(response => response.json())
        .then(json => {
          /* eslint-disable no-console */
          console.log(json, "that", this.element.config.filter);
          this.options = json.items.map(type => type.system);
          this.isLoading = false;
        });
    },
    onSelect: function() {
      this.save(this.selectedTypes);
    },
    save: function(value) {
      this.$emit("update:value", value);
    }
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped></style>
