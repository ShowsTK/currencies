<template>
  <div class="paginator">
    <span>Количество элементов на странице: </span>
    <select name=""
            id="perpage"
            v-model.number="perPage"
            @change="changeShowedItemsRange"
    >
      <option value=5>5</option>
      <option value=10 selected>10</option>
      <option value=25>25</option>
      <option value=50>50</option>
    </select>
    <span> {{from}}-{{to}} из {{countItems}}</span>

    <button class="btn btn-pict"
            title="Предыдущая страница"
            :disabled="curPage === 1"
            @click="prev">
      <img src="../assets/icons/png/arrow-prev.png"
           alt="Пред.">
    </button>

    <button class="btn btn-pict"
            title="Следующая страница"
            :disabled="to >= countItems"
            @click="next">
      <img src="../assets/icons/png/arrow-next.png"
           alt="След.">
    </button>
  </div>
</template>

<script>
  export default {
    name: "Paginator",
    props: ['items'],
    data() {
      return {
        curPage: 1,
        perPage: 10
      }
    },
    computed: {
      from() {
        return this.curPage * this.perPage - this.perPage + 1;
      },
      to() {
        return this.curPage * this.perPage;
      },
      countItems() {
        return this.items.length;
      }
    },
    methods: {
      next() {
        this.curPage++;
        this.changePage();
      },
      prev() {
        this.curPage--;
        this.changePage();
      },
      changeShowedItemsRange() {
        const maxPage = Math.ceil(this.countItems / this.perPage);
        if (this.curPage > maxPage) {
          this.curPage = maxPage;
        }
        this.changePage();
      },
      changePage() {
        const showedData = this.changeRangeData();
        this.$emit('change-page', showedData);
      },
      changeRangeData() {
        return this.items.slice(this.from - 1, this.to);
      }
    }
  }
</script>

<style scoped>
  button:disabled {
    opacity: 0.4;
  }
  .paginator {
    font-size: 12px;
  }
</style>
