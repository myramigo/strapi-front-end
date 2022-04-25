<template>
  <div>
    <h1>Products</h1>
    <div v-for="c in categories" :key="c.id" @click="pickCategory(c.id)">
      <div class="category_nav">
        {{ c.title }}
      </div>
    </div>

    <input type="search" id="title_search" placeholder="search"/>
    <button @click="titleSearch">search</button>

    <input type="num" id="price_gte" />
    <input type="num" id="price_lte" />
    <button @click="priceFilter">filter</button>

    <select v-model="priceSort" @change="onPriceSortChange">
      <option value="price:asc">priceAsc</option>
      <option value="price:desc">priceDesc</option>
      <option value="updatedAt:desc">newest</option>
    </select>

    <div v-for="p in products" :key="p.id" class="card" @click="move(p.id)">
      <p>{{ p.title }}</p>
      <p>{{ p.price }}</p>
    </div>

     pages:
    <div v-for="page in pages" :key="page" @click="moveToPage(page)" style="display: inline-block;">
      <button>{{ page }}</button>
    </div>

  </div>
</template>

<script>
import axios from "axios";
import qs from "qs";
export default {
  name: "Products",
  data() {
    return {
      products: [],
      priceSort: "updatedAt:desc",
      categories: [],
      pages: 1,
      selPage: 1,
    };
  },

  async mounted() {
    const c_query = qs.stringify(
      {
        sort: ["title"],
      },
      { encodeValuesOnly: true }
    );
    const { data: c } = await axios.get("http://localhost:1337/api/categories?" + c_query);
    this.categories = c.data.map((product) => {
      return {
        id: product.id,
        ...product.attributes,
      };
    });
    const id = this.categories[0].id
    this.$router.push("?categoryId=" + id)
    
    const p_query = qs.stringify(
      {
        populate: "category",
        sort: ["updatedAt:desc"],
        filters: {
          category: {
            id:{
              $eq: Number(id),
            }
          },
        },
        pagination: {
          page: this.page,
          pageSize: 5,
        },
      },
      { encodeValuesOnly: true }
    );
    const { data: p } = await axios.get("http://localhost:1337/api/products?" + p_query);
    this.products = p.data.map((product) => {
      return {
        id: product.id,
        ...product.attributes,
      };
    });

  },
  
  methods: {
    move(id) {
      this.$router.push("/products/" + id);
    },

    async onPriceSortChange() {
      const id = this.$route.query.categoryId;
      const query = qs.stringify(
        {
          sort: [this.priceSort],
          filters: {
            category: {
              id:{
                $eq: Number(id),
              }
            },
          },
        },
        { encodeValuesOnly: true }
      );
      const { data } = await axios.get("http://localhost:1337/api/products?" + query);
      this.products = data.data.map((product) => {
        return {
          id: product.id,
          ...product.attributes,
        };
      });
    },

    async priceFilter() {
      const id = this.$route.query.categoryId;
      const query = qs.stringify(
        {
          filters: {
            $and: [
              {
                price: {
                  $gte: Number(document.getElementById("price_gte").value),
                },
              },
              {
                price: {
                  $lte: Number(document.getElementById("price_lte").value),
                },
              },
            ],
            category: {
            id:{
              $eq: Number(id),
            }
          },
          },
        },
        { encodeValuesOnly: true }
      );
      const { data } = await axios.get("http://localhost:1337/api/products?" + query);
      this.products = data.data.map((product) => {
        return {
          id: product.id,
          ...product.attributes,
        };
      });
    },

    async pickCategory(id) {
      const p_query = qs.stringify(
        {
          populate: "category",
          sort: ["updatedAt:desc"],
          filters: {
            category: {
              id:{
                $eq: Number(id),
              }
            },
          },
          pagination: {
            page: this.selPage,
            pageSize: 5,
          },
        },
        { encodeValuesOnly: true }
      );
      const { data: p } = await axios.get("http://localhost:1337/api/products?" + p_query);
      this.products = p.data.map((product) => {
        return {
          id: product.id,
          ...product.attributes,
        };
      });
     this.pages = p.meta.pagination.pageCount
      this.$router.push("/products?categoryId=" + id);
    },

    async moveToPage(page) {
      this.selPage = page
      const p_query = qs.stringify(
        {
          populate: "category",
          sort: ["updatedAt:desc"],
          filters: {
            category: {
              id: {
                $eq: Number(this.$route.query.categoryId),
              },
            },
          },
          pagination: {
            page: this.selPage,
            pageSize: 5,
          },
        },
        { encodeValuesOnly: true }
      );
      const { data: p } = await axios.get("http://localhost:1337/api/products?" + p_query);
      this.products = p.data.map((product) => {
        return {
          id: product.id,
          ...product.attributes,
        };
      });
    },

    async titleSearch() {
      const id = this.$route.query.categoryId;
      const query = qs.stringify(
        {
          filters: {
            title: {
              $contains: document.getElementById("title_search").value,
            },
          },
          category: {
            id:{
              $eq: Number(id),
            }
          },
        },
        { encodeValuesOnly: true }
      );
      const { data } = await axios.get("http://localhost:1337/api/products?" + query);
      this.products = data.data.map((product) => {
        return {
          id: product.id,
          ...product.attributes,
        };
      });
    },

  },
};
</script>

<style scoped>
.card {
  padding: 20px;
  border-radius: 5px;
  margin-bottom: 20px;
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
}
.card:hover {
  cursor: pointer;
  box-shadow: 0 8px 17px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.category_nav {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 20px;
  border: 3px solid black;
}
.category_nav:hover {
  cursor: pointer;
}
</style>