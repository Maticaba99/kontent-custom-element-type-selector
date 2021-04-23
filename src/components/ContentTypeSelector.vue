<template>
  <div>
    <multiselect
      v-model="selectedTypes"
      :key="option"
      placeholder="Select products"
      open-direction="bottom"
      :options="options"
      :option-height="104"
      :multiple="true"
      :loading="isLoading"
      :internal-search="false"
      :close-on-select="false"
      label="name"
      track-by="name"
      :disabled="element.disabled"
      @input="onSelect"
      @search-change="fetchTypes"
      :clear-on-select="false"
      :options-limit="300"
      :limit="3"
      :limit-text="limitText"
    >
      <template slot="singleLabel">
        <img class="option__image" :src="option.image" :alt="option.name" />
        />
        <span class="option__desc">
          <span class="option__title">{{ option.name }}</span></span
        ></template
      >

      <template slot="option"
        ><img class="option__image" :src="option.image" :alt="option.name" />
        <div class="option__desc">
          <span class="option__title">{{ option.name }}</span
          ><span class="option__small">{{ option.id }}</span>
        </div>
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
    fetchTypes(searchTerm) {
      fetch(
        `https://6d1a49bf49e44b74a37bcaa0edf1d9e7.eastus2.azure.elastic-cloud.com:9243/products/_search`,
        {
          headers: {
            "Content-Type": "application/json",
            Authorization: `Basic ZWxhc3RpYzpXNFRDNTVFTUdRUmx5a0F2ZVZaOVVnTjM`
          },
          data: JSON.stringify({
            from: 0,
            size: 10,
            query: {
              bool: {
                must: [
                  {
                    match: {
                      "productfields.salsifydata_inheritance_hierarchy_level_id":
                        "Base"
                    }
                  },
                  {
                    wildcard: {
                      "productfields.product_name.en-us": searchTerm
                    }
                  },
                  {
                    exists: {
                      field: "productcard"
                    }
                  }
                ]
              }
            }
          })
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
