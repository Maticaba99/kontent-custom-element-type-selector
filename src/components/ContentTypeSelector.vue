<template>
  <div>
    <multiselect
      v-model="selectedTypes"
      :key="option"
      placeholder="Select products"
      open-direction="bottom"
      :hide-selected="true"
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
      :limit-text="limitText"
      :custom-label="customLabel"
      :show-labels="false"
    >
      <template slot="singleLabel" slot-scope="props">
        <img
          class="option__image"
          :src="props.option.image"
          :alt="props.option.name"
        />
        />
        <span class="option__desc">
          <span class="option__title">{{ props.option.name }}</span></span
        ></template
      >

      <template slot="option" slot-scope="props"
        ><img
          class="option__image"
          :src="props.option.image"
          :alt="props.option.name"
        />
        <div class="option__desc">
          <span class="option__title">{{ props.option.name }}</span
          ><span class="option__small">{{ props.option.id }}</span>
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
    async fetchTypes(searchTerm) {
      // eslint-disable-next-line no-console
      console.log(searchTerm);
      const POST_BODY = {
        from: 0,
        size: 100,
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
      };
      this.isLoading = true;
      try {
        const fetching = await fetch(
          `https://6d1a49bf49e44b74a37bcaa0edf1d9e7.eastus2.azure.elastic-cloud.com:9243/products/_search`,
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Basic ZWxhc3RpYzpXNFRDNTVFTUdRUmx5a0F2ZVZaOVVnTjM`
            },
            data: JSON.stringify(POST_BODY)
          }
        )
          .then(response => response.json())
          .then(json => {
            this.options = json.hits.hits.map(product => {
              // eslint-disable-next-line no-console
              console.log(product._source.productfields.product_name["en-us"]);
              return {
                id: product._id,
                name: product._source.productfields.product_name["en-us"],
                image: product._source.images && product._source.images[0]
              };
            });
            this.isLoading = false;
          });
        // eslint-disable-next-line no-console
        console.log(fetching);
      } catch (err) {
        // eslint-disable-next-line no-console
        console.log(err);
      }
    },
    onSelect: function() {
      this.save(this.selectedTypes);
    },
    save: function(value) {
      this.$emit("update:value", value);
    },
    customLabel({ name, id }) {
      return `${name} – ${id}`;
    }
  }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>

<style scoped></style>
