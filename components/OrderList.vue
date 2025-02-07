<template>
  <v-card class="mx-auto">
    <v-list v-if="getPendingOrders.length > 0">
      <div v-for="order in getPendingOrders" :key="order.id">
        <v-list-item
          three-line
          link
        >
          <v-list-item-title class="d-flex justify-space-between">
            {{ order.fiat_amount }} {{ order.fiat_code.toUpperCase() }} {{ getFlag(order.fiat_code) }}
            <v-chip
              class="ma-2"
              rounded
              size="small"
              style="cursor: pointer"
              :color="order.kind === 'Sell' ? 'red' : 'green'"
            >
              {{ order.kind.toUpperCase() }}
            </v-chip>
          </v-list-item-title>
          <v-list-item-subtitle>
            {{ summary(order) }}
          </v-list-item-subtitle>
          <v-list-item-subtitle>
            <div class="d-flex justify-space-between">
              Payment via: {{ order.payment_method }} - id: {{ order.id }}
              <take-sell-order-dialog v-if="showTakeSell(order)" :order="order"/>
              <take-buy-order-dialog v-if="showTakeBuy(order)" :order="order"/>
            </div>
          </v-list-item-subtitle>
        </v-list-item>
        <v-divider/>
      </div>
    </v-list>
    <no-orders v-else/>
  </v-card>
</template>

<script lang="ts">
import { defineComponent, computed, ref } from 'vue'
import { Order, OrderType } from '@/stores/types'
import { useOrders } from '@/stores/orders'
import { useAuth } from '@/stores/auth'
import fiat from '~/assets/fiat.json'

type FiatData = {
  symbol: string,
  name: string,
  symbol_native: string,
  decimal_digits: number,
  rounding: number,
  code: string,
  emoji: string
  name_plural: string,
  price: boolean
}

export default defineComponent({
  setup() {
    const authStore = useAuth()
    const ordersStore = useOrders()
    const getPendingOrders = computed(() => ordersStore.getPendingOrders)

    return {
      fiatMap: fiat as { [key: string]: Partial<FiatData> },
      headerHeight: 64,
      authStore,
      getPendingOrders
    }
  },
  methods: {
    getFlag(fiatCode: string) {
      return this.fiatMap[fiatCode?.toUpperCase()].emoji ?? ''
    },
    showTakeSell(order: Order) : boolean {
      const isLocked: boolean = this.authStore.isLocked
      return order.kind === OrderType.SELL && !isLocked
    },
    showTakeBuy(order: Order) : boolean {
      const isLocked = this.authStore.isLocked
      return order.kind === OrderType.BUY && !isLocked
    },
    summary(order: Order) {
      if (order.amount === 0) {
        if (order.kind === OrderType.SELL)
          return `Selling sats for ${this.tradeIn(order)}`
        else
          return `Buying sats for ${this.tradeOut(order)}`
      } else {
        if (order.kind === OrderType.SELL)
          return `Selling ${this.tradeOut(order)} for ${this.tradeIn(order)}`
        else
          return `Buying ${this.tradeIn(order)} por ${this.tradeOut(order)}`
      }
    },
    tradeIn(order: Order) {
      if (order.kind === OrderType.SELL) {
        return `${order.fiat_amount} ${order.fiat_code.toUpperCase()}`
      } else {
        return `${order.amount} sats`
      }
    },
    tradeOut(order: Order) {
      if (order.kind === OrderType.SELL) {
        return `${order.amount} sats`
      } else {
        return `${order.fiat_amount} ${order.fiat_code.toUpperCase()}`
      }
    }
  },
})
</script>