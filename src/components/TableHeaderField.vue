<!--ячейка хедера таблицы-->
<template>
  <th @click="changeOrderStatus">{{paramsData.text}}
    <img v-show="sortable && orderStatus === 0" src="../assets/icons/png/arrow-sort-no.png" alt="no sort">
    <img v-show="sortable && orderStatus === 1" src="../assets/icons/png/arrow-up.png" alt="asc">
    <img v-show="sortable && orderStatus === -1" src="../assets/icons/png/arrow-down.png" alt="desc">
  </th>
</template>

<script>
  export default {
    name: "tableHeaderField",
    props: ['paramsData'],
    data() {
      return {
        sortable: false,
        orderStatus: 0 // 0 - no sort, 1 - ascending, -1 - descending
      }
    },
    created() {
      this.sortable = this.paramsData.sortable;
    },
    methods: {
      changeOrderStatus() {
        this.orderStatus = (this.orderStatus === 1) ? -1 : this.orderStatus + 1;
        this.$emit("change-order", this.orderStatus, this.paramsData.name);
      }
    }
  }
</script>

<style scoped>

</style>
