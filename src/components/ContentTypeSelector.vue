<template>
  <div>
    <label class="typo__label">Custom option template</label>
    <multiselect
      v-model="selectedTypes"
      placeholder="Fav No Man’s Sky path"
      label="title"
      track-by="title"
      :options="options"
      :option-height="104"
      :multiple="true"
      :loading="isLoading"
      :custom-label="customLabel"
      :show-labels="false"
    >
      <template slot="singleLabel"
        ><img
          class="option__image"
          :src="options.img"
          alt="No Man’s Sky"
        /><span /><span class="option__desc"
          ><span class="option__title">{{ options.title }}</span></span
        ></template
      >
      <template slot="option"
        ><img class="option__image" :src="options.img" alt="No Man’s Sky" />
        <div class="option__desc">
          <span class="option__title">{{ options.title }}</span
          ><span class="option__small">{{ options.desc }}</span>
        </div>
        <script>
          console.log("ndea");
        </script>
      </template>
    </multiselect>
    <pre class="language-json"><code>{{ value  }}</code></pre>
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
            // eslint-disable-next-line no-console
            console.log(
              product._source.productfields &&
                product._source.productfields.image_closeup &&
                product._source.productfields.image_closeup.salsifysource_url
            );
            return {
              id: product._id,
              name: product._source.productfields.product_name["en-us"],
              image:
                product._source.productfields &&
                product._source.productfields.image_closeup &&
                product._source.productfields.image_closeup.salsifysource_url
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
