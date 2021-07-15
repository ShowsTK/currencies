<!--Диалоговое окно для подтверждения действий-->
<template>
  <div class="modal is-active">
    <div class="modal-background">
      <div class="modal-box">
        <div class="modal-content">
          <slot>Вы уверены?</slot>
        </div>
        <div class="modal-buttons">
          <button class="btn btn-btn btn-modal btn-custom-self"
                  @click.prevent="confirmYes">
            Да
          </button>
          <button class="btn btn-btn btn-modal btn-custom-self"
                  @click.prevent="confirmNo">
            Нет
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ConfirmBox",
    methods: {
      //генерирует событие при нажатии на кнопку "Да"
      confirmYes() {
        this.$emit('confirm-yes');
      },
      //генерирует событие при нажатии на кнопку "Нет"
      confirmNo() {
        this.$emit('confirm-no');
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "../styles/confirm";
</style>
