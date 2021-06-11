<script>
import Vue from "vue";
import VueNano from "@/vue-nano.vue";

export default Vue.extend({
  name: "ServeDev",
  data: () => ({
    active: false,
    received: 0.0
  }),
  components: {
    VueNano
  }
});
</script>

<template>
  <div id="app">
    <vs-button @click="active = !active">Click me!</vs-button>
    <vs-button @click="received += 0.001">Send Money</vs-button>
    <vue-nano
      v-model="active"
      address="nano_1zfnwwk5xq74tddsq9t13oemxfkyqhz6f6jzej3wfqe4q3y39381qtq98dmo"
      :amount="0.1"
      :received="received"
    />
  </div>
</template>
