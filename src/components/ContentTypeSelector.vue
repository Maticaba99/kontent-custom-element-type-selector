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
    },
    filter: {
      type: String
    },
    methods: {
      limitText(count) {
        return `and ${count} other countries`;
      },
      fetchTypes() {
        fetch(
          `https://deliver.kontent.ai/${this.context.projectId}/types?elements=null`,
          {
            headers: {
              "Content-Type": "application/json",
              Authorization:
                "Bearer ew0KICAiYWxnIjogIkhTMjU2IiwNCiAgInR5cCI6ICJKV1QiDQp9.ew0KICAianRpIjogIjcxMjE2OGY5MTkxYzRiMWI5OWUzYmZmYWZiMjMxZDAxIiwNCiAgImlhdCI6ICIxNjE1NDI1MjkzIiwNCiAgImV4cCI6ICIxOTYxMDI1MjkzIiwNCiAgInByb2plY3RfaWQiOiAiMzMyMGM0NTBkMGZjMDAzZTA0NTA5ODVjYzczOWVmNDciLA0KICAidmVyIjogIjEuMC4wIiwNCiAgImF1ZCI6ICJkZWxpdmVyLmtlbnRpY29jbG91ZC5jb20iDQp9.6CBhVGn4KC3uJeUrD-9XOOTY9Zt81LCegZTzateG5LM"
            }
          }
        )
          .then(response => response.json())
          .then(json => {
            this.options = json.types.map(type => type.system);
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
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped></style>
