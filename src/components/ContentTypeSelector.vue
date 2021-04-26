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
      <template slot="tag" slot-scope="props">
        <div class="container">
          <div class="imageContainer">
            <img
              class="option__image"
              :src="props.option.image"
              :alt="props.option.name"
            />
            <span class="remove" @click="props.remove(props.option)">❌</span>
            <span class="add" @click="props.option.quantity + 1">❌</span>
          </div>

          <span class="option__desc">
            <span class="option__title"
              ><strong>Name:</strong> {{ props.option.name }}</span
            >
            <span class="option__title"
              ><strong>Quantity: </strong> {{ props.option.quantity }}</span
            >
          </span>
        </div>
      </template>

      <template slot="option" slot-scope="props"
        ><img
          class="option__image"
          :src="props.option.image"
          :alt="props.option.name"
        />
        <div class="option__desc">
          <span class="option__title">Title: {{ props.option.name }}</span
          ><span class="option__small">ID: {{ props.option.id }}</span>
          <span class="option__small"
            >dimensions: {{ props.option.dimensions }}</span
          >
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
    async fetchTypes(query) {
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
                exists: {
                  field: "productcard"
                }
              }
            ],
            should: [
              {
                wildcard: {
                  "productfields.product_name.en-us": query && query
                }
              },
              {
                wildcard: {
                  "productfields.unique_id": query && query
                }
              }
            ]
          }
        }
      };
      this.isLoading = true;
      await fetch(this.element.config.API, {
        method: "post",
        headers: {
          Authorization: `Basic ${this.element.config.API_AUTH}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify(POST_BODY)
      })
        .then(response => response.json())
        .then(json => {
          // eslint-disable-next-line no-console
          console.log(json);
          this.options = json.hits.hits.map(product => {
            return {
              id: product._source.productfields.unique_id,
              name: product._source.productfields.product_name["en-us"],
              dimensions:
                product._source.productcard &&
                product._source.productcard.dimensionsin,
              image:
                product._source.productcard &&
                product._source.productcard.featureimage,
              quantity: 1
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
