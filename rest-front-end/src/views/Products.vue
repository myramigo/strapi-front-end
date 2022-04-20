<template>
  <div>
      <h1>Products</h1>
      <input type="num" id="price_gte">
      <input type="num" id="price_lte">
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
  </div>
</template>

<script>
import axios from 'axios'
import qs from 'qs'
export default {
  name: 'Products',
  data () {
    return {
      products: [],
      priceSort: "updatedAt:desc"
    }
  },
  async mounted() {
    const query = qs.stringify({
      sort: ['updatedAt:desc']
    }, {encodeValuesOnly: true});
    const { data } = await axios.get('http://localhost:1337/api/products?' + query);
    this.products = data.data.map(product => {
      return {
        id: product.id,
        ...product.attributes
      }
    });
  },
  methods: {
    move(id) {
      this.$router.push('/products/' + id);
    },
    async priceFilter(){
    const query = qs.stringify({
      filters: {
        $and: [
          {
            price: {
              $gte: Number(document.getElementById('price_gte').value)
            }
          },
          {
            price: {
              $lte: Number(document.getElementById('price_lte').value)
            }
          }
        ]
      }
    }, {encodeValuesOnly: true});
    const { data } = await axios.get('http://localhost:1337/api/products?' + query);
    this.products = data.data.map(product => {
      return {
        id: product.id,
        ...product.attributes
      }
    });
    },
     async onPriceSortChange(){
    const query = qs.stringify({
      sort: [this.priceSort]
    }, {encodeValuesOnly: true});
    const { data } = await axios.get('http://localhost:1337/api/products?' + query);
    this.products = data.data.map(product => {
      return {
        id: product.id,
        ...product.attributes
      }
    });
    }

  }
}
</script>

<style scoped>
.card {
  padding: 20px;
  border-radius: 5px;
  margin-bottom: 20px;
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
}
.card:hover {
  box-shadow: 0 8px 17px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
</style>