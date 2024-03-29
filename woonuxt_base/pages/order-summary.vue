<script lang="ts" setup>
const route = useRoute();
const order = ref() as Ref<null | Order>;
const { customer } = useAuth();

onMounted(() => {
  if (route.params.orderId) getOrder();
});

async function getOrder() {
  try {
    const data = await GqlGetOrder({ id: route.params.orderId as string });
    if (data.order) order.value = data.order;
  } catch (err: any) {
    console.error(err);
  }
}

function formatDate(date = '') {
  return new Date(date).toLocaleDateString('en-US', {
    day: 'numeric',
    month: 'short',
    year: 'numeric',
  });
}

function formatPrice(price: string) {
  return parseFloat(price).toLocaleString('en-US', { style: 'currency', currency: 'EUR' });
}

const isGuest = computed(() => {
  return !customer.value.databaseId;
});
</script>

<template>
  <div class="w-full min-h-[600px] flex justify-center items-center p-8 text-gray-800 md:bg-white md:rounded-xl md:mx-auto md:shadow-lg md:my-24 md:mt-8 md:max-w-3xl md:p-16">
    <div v-if="order?.databaseId && !isGuest" class="w-full">
      <h1 class="mb-2 text-xl font-semibold">{{ $t('messages.shop.orderSummary') }}</h1>
      <p>{{ $t('messages.shop.orderThanks') }}</p>

      <hr class="my-8" />

      <div class="flex justify-between items-center">
        <div>
          <div class="text-xs text-gray-400 uppercase mb-2">{{ $t('messages.shop.order') }}</div>
          <div class="leading-none">#{{ order.databaseId! }}</div>
        </div>
        <div>
          <div class="text-xs text-gray-400 uppercase mb-2">{{ $t('messages.general.date') }}</div>
          <div class="leading-none">{{ formatDate(order.date!) }}</div>
        </div>
        <div>
          <div class="text-xs text-gray-400 uppercase mb-2">{{ $t('messages.general.status') }}</div>
          <OrderStatusLabel :status="order.status" />
        </div>
        <div>
          <div class="text-xs text-gray-400 uppercase mb-2">{{ $t('messages.general.paymentMethod') }}</div>
          <div class="leading-none">{{ order.paymentMethodTitle }}</div>
        </div>
      </div>

      <hr class="my-8" />

      <div class="grid gap-2">
        <div v-if="order.lineItems" v-for="item in order.lineItems.nodes" :key="item.product?.databaseId!" class="flex items-center justify-between gap-8">
          <img
            v-if="item.product?.node"
            class="w-16 h-16 rounded-xl"
            :src="item.variation?.node?.image?.sourceUrl || item.product.node?.image?.sourceUrl || '/images/placeholder.png'"
            :alt="item.variation?.node?.image?.altText || item.product.node?.image?.altText || 'Product image'"
            :title="item.variation?.node?.image?.title || item.product.node?.image?.title || 'Product image'"
            width="64"
            height="64"
            loading="lazy" />
          <div class="flex-1 leading-tight">
            {{ item.variation ? item.variation?.node?.name : item.product?.node.name! }}
          </div>
          <div class="text-sm text-gray-600">Qty. {{ item.quantity }}</div>
          <span class="text-sm font-semibold">{{ formatPrice(item.total!) }}</span>
        </div>
      </div>

      <hr class="my-8" />

      <div>
        <div class="flex justify-between">
          <span>{{ $t('messages.shop.subtotal') }}</span>
          <span>{{ order.subtotal }}</span>
        </div>
        <div class="flex justify-between">
          <span>{{ $t('messages.general.tax') }}</span>
          <span>{{ order.totalTax }}</span>
        </div>
        <div class="flex justify-between">
          <span>{{ $t('messages.general.shipping') }}</span>
          <span>{{ order.shippingTotal }}</span>
        </div>
        <hr class="my-8" />
        <div class="flex justify-between">
          <span class>{{ $t('messages.shop.total') }}</span>
          <span class="font-semibold">{{ order.total }}</span>
        </div>
      </div>
    </div>
    <div v-else-if="isGuest && !order" class="w-full text-center">
      <h1 class="mb-2 text-xl font-semibold">{{ $t('messages.shop.orderSummary') }}</h1>
      <p class="mb-8">{{ $t('messages.shop.orderThanks') }}</p>
      <NuxtLink to="/" class="bg-primary rounded-lg font-bold text-white text-center min-w-[150px] p-2.5 focus:outline-noney">
        {{ $t('messages.general.goHome') }}
      </NuxtLink>
    </div>
    <LoadingIcon v-else />
  </div>
</template>
