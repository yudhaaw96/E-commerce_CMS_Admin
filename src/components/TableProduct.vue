<template>
  <div>
    <div>
      <h3><font-awesome-icon icon="cubes" /> Products (total: {{ totalProducts }})</h3>
      <b-row>
        <b-col lg="3" class="my-1">
          <b-form-group
            label="Filter"
            label-cols-sm="2"
            label-align-sm="right"
            label-size="sm"
            label-for="filterInput"
            class="mb-0"
          >
            <b-input-group size="sm">
              <b-form-input
                v-model="filter"
                type="search"
                id="filterInput"
                placeholder="Type to Search"
              ></b-form-input>
              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''"
                  >Clear</b-button
                >
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>
        <b-col lg="3" class="my-1">
          <b-form-group
            label="Per page"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="perPageSelect"
            class="mb-0"
          >
            <b-form-select
              v-model="perPage"
              id="perPageSelect"
              size="sm"
              :options="pageOptions"
            ></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>
      <b-table
        id="products"
        class="table"
        bordered="bordered"
        head-variant="dark"
        tbody-tr-class="text-center text-justify"
        tbody-td-class="align-middle"
        thead-tr-class="text-center text-justify"
        :items="products"
        :fields="fields"
        :filter="filter"
        :sort-by.sync="sortBy"
        :sort-desc.sync="sortDesc"
        responsive="md"
        :per-page="perPage"
        :current-page="currentPage"
      >
        <template v-slot:cell(Name)="data">
          <p class="font-weight-bold">{{ data.value }}</p>
        </template>
        <template v-slot:cell(Image)="data">
          <img :src="data.value" width="200" height="auto"/>
        </template>
        <template v-slot:cell(Price)="data">
          {{ convertToCurrency(data.value) }}
        </template>
        <template v-slot:cell(id)="data">
          <b-button
            variant="warning"
            @click="getProductBeforeUpdateProduct(data.value)"
            ><font-awesome-icon icon="edit" /> Edit</b-button
          >
          -
          <b-button variant="danger" @click="showMsgBoxDelete(data.value)"><font-awesome-icon icon="trash-alt" /> Delete</b-button>
        </template>
      </b-table>
    </div>
    <div class="d-flex justify-content-center">
      <b-pagination
        v-model="currentPage"
        :total-rows="rows"
        :per-page="perPage"
        aria-controls="my-table"
      ></b-pagination>
    </div>
    <!-- // ========================== // Modal // ========================== -->
    <div>
      <b-modal
        id="modal-edit"
        ref="modal-edit"
        title="Edit a product"
        header-bg-variant="warning"
        header-text-variant="light"
        footerBgVariant="secondary"
      >
        <FormUpdateProduct
          :selectedProduct="selectedProduct"
          @closeModal="closeModal"
        />
        <template v-slot:modal-footer>
          <div class="w-100 h-auto"></div>
        </template>
      </b-modal>
    </div>
  </div>
</template>

<script>
import FormUpdateProduct from '@/components/forms/FormUpdateProduct'
import currency from '@/helpers/currency'

export default {
  name: 'TableProduct',
  props: ['perPage', 'pageOptions'],
  data () {
    return {
      idOfSelectedProduct: '',
      selectedProduct: '',
      show: false,
      bordered: true,
      filter: null,
      currentPage: 1,
      sortBy: 'id',
      sortDesc: true,
      fields: [
        { key: 'Name', sortable: true },
        { key: 'Image', sortable: false },
        { key: 'Price', sortable: true },
        { key: 'Stock', sortable: true },
        { key: 'id', label: 'Options' }
      ],
      boxDelete: ''
    }
  },
  components: {
    FormUpdateProduct
  },
  methods: {
    fetchProducts () {
      this.$store
        .dispatch('fetchProducts')
        .then(({ data }) => {
          this.$store.commit('SET_PRODUCTS', data.Products)
        })
        .catch(({ err }) => {
          console.log(err)
        })
    },
    getProductBeforeUpdateProduct (id) {
      this.idOfSelectedProduct = id
      this.$store
        .dispatch('getProductById', id)
        .then(({ data }) => {
          this.selectedProduct = data.Product
        })
        .catch(({ err }) => {
          console.log(err)
        })
      this.$refs['modal-edit'].show()
    },
    showMsgBoxDelete (id) {
      this.boxDelete = ''
      this.$bvModal.msgBoxConfirm('Please confirm that you want to delete?', {
        title: 'Please confirm',
        size: 'sm',
        buttonSize: 'sm',
        okVariant: 'danger',
        okTitle: 'YES',
        cancelTitle: 'NO',
        footerClass: 'p-2',
        hideHeaderClose: false,
        centered: true
      })
        .then(value => {
          this.boxDelete = value
          if (value === true) {
            this.deleteProduct(id)
          }
        })
        .catch(err => {
          console.log(err)
        })
    },
    deleteProduct (id) {
      this.$store
        .dispatch('deleteProduct', id)
        .then(({ data }) => {
          const deletedProductName = data.DeletedProduct.name
          this.fetchProducts()
          this.$swal.fire(
            `Success delete "${deletedProductName}"`,
            'You just deleted a product!',
            'success'
          )
        })
        .catch(({ err }) => {
          console.log(err)
        })
    },
    convertToCurrency (money) {
      return currency(money)
    },
    closeModal () {
      this.$refs['modal-edit'].hide()
    }
  },
  computed: {
    products () {
      const products = []
      this.$store.state.products.forEach(el => {
        products.push({
          Name: el.name,
          Image: el.image_url,
          Price: el.price,
          Stock: el.stock,
          id: el.id
        })
      })
      return products
    },
    totalProducts () {
      return this.$store.state.products.length
    },
    rows () {
      return this.products.length
    }
  },
  created () {
    this.fetchProducts()
  }
}
</script>

<style>
.table > tbody > tr > td {
     vertical-align: middle;
     background-color: rgba(0, 0, 0, 0.020);
}
</style>
