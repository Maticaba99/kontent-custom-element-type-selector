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
        <div class="title">{{ option.name }}</div>
        <div class="title">{{ option.id }}</div>
        <!-- eslint-disable-next-line prettier/prettier -->
        <img class="preview" src={{option.image}} />
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
        `https://6d1a49bf49e44b74a37bcaa0edf1d9e7.eastus2.azure.elastic-cloud.com:9243/products/_search`,
        {
          headers: {
            "Content-Type": "application/json",
            Authorization: `Basic ZWxhc3RpYzpXNFRDNTVFTUdRUmx5a0F2ZVZaOVVnTjM`
          }
        }
      )
        .then(response => response.json())
        .then(json => {
          this.options = json.hits.hits.map(product => {
            return {
              id: product._id,
              name: product._source.productfields.product_name["en-us"],
              image: product._source.productcard.featureimage
            };
          });
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
