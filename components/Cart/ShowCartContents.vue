/* eslint-disable no-console */
<template>
  <div>
    <SpinnerLoadingSpinner v-if="loading" />
    <div>
      <h1 class="h-10 p-6 text-3xl font-bold text-center">Cart</h1>
      <section class="mt-10">
        <div
          v-for="products in cartProducts"
          :key="products.id"
          class="container mx-auto mt-4 flex-container"
        >
          <div v-if="displayRemove" class="item">
            <span class="block mt-2 font-extrabold">Remove: <br /></span>
            <span class="item-content">
              <img
                class="mt-2 ml-4 cursor-pointer"
                :class="{ removing: removingCartItem }"
                alt="Remove icon"
                aria-label="Remove"
                src="@/assets/svg/Remove.svg"
                @click="handleRemoveProduct(products)"
              />
            </span>
          </div>
          <div class="item">
            <span class="block mt-2 font-extrabold">Name: <br /></span>
            <span class="item-content">{{ products.product.name }}</span>
          </div>
          <div class="item">
            <span class="block mt-2 font-extrabold">Quantity: <br /> </span>
            <span class="item-content">
              {{ products.quantity }}
            </span>
          </div>
          <div class="item">
            <span class="block mt-2 font-extrabold">Subtotal: <br /></span>
            <span class="item-content"> {{ products.total }} </span>
          </div>
        </div>
      </section>
      <h2 v-if="!cartProducts.length" class="m-4 text-3xl text-center">
        Cart is currently empty
      </h2>
      <CartCheckoutButton v-if="displayRemove && cartProducts.length" />
    </div>
  </div>
</template>

<script>
import GET_CART_QUERY from '@/apollo/queries/GET_CART_QUERY'
import UPDATE_CART_MUTATION from '@/apollo/mutations/UPDATE_CART_MUTATION'

export default {
  props: {
    displayRemove: { type: Boolean, required: false },
  },
  data() {
    return {
      remoteCart: null,
      remoteError: null,
      subTotal: null,
      loading: true,
      removingCartItem: false,
    }
  },
  computed: {
    cartProducts() {
      if (this.remoteCart) {
        return this.remoteCart.cart.contents.nodes
      }
      return null
    },
  },
  apollo: {
    cart: {
      prefetch: true,
      query: GET_CART_QUERY,
      result({ data, loading, networkStatus }) {
        const cartIsReady = data && networkStatus === 7
        if (cartIsReady && !loading) {
          this.loading = false
          this.remoteCart = data
          this.subTotal = data.cart.total
          this.cartLength = data.cart.contents.nodes.reduce(
            (accumulator, argument) => accumulator + argument.quantity,
            0
          )
        }
      },
      error(error) {
        this.remoteError = error
      },
    },
  },
  methods: {
    async handleRemoveProduct(products) {
      this.removingCartItem = true
      const updatedItems = []
      updatedItems.push({
        key: products.key,
        quantity: 0,
      })
      try {
        await this.$apollo
          .mutate({
            mutation: UPDATE_CART_MUTATION,
            variables: {
              input: {
                items: updatedItems,
              },
            },
          })
          .then(() => {
            this.removingCartItem = false
            this.$apollo.queries.cart.refetch()
          })
      } catch (error) {
        this.remoteError = error
      }
    },
  },
}
</script>

<style scoped>
.flex-container {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  justify-content: space-around;
  align-items: center;
  align-content: center;
  max-width: 1380px;
  @apply border border-gray-300 rounded-lg shadow;
}

.item {
  @apply lg:m-2 xl:m-4 xl:w-1/6 lg:w-1/6 sm:m-2 w-auto;
}

.item-content {
  @apply inline-block mt-4 w-20 h-12 md:w-full lg:w-full xl:w-full;
}

.removing {
  @apply animate-spin cursor-not-allowed;
}
</style>
