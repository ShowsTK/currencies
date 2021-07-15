<!--Диалоговое окно формы ввода/корректировки записей-->
<template>
  <div class="modal is-active">
    <div class="modal-background">
      <div class="modal-box">
        <div class="modal-content">
          <button class="modal-close-btn"
                  @click.prevent="dialogClose">
            <img src="../assets/icons/png/close.png"
                 alt="Закрыть">
          </button>
          <form action="#"
                @submit.prevent>
            <div class="field">
              <label class="field__lbl">Буквенный код валюты</label>
              <input type="text"
                     class="field__txt"
                     v-model.trim="item.Cur_Abbreviation"
                     placeholder=""
                     pattern="^[a-zA-Z]{3}$"
              >
            </div>
            <div class="field">
              <label class="field__lbl">Наименование валюты</label>
              <input type="text"
                     class="field__txt"
                     v-model.trim="item.Cur_Name"
                     placeholder=""
                     pattern="^[а-яА-Я]$"
              >
            </div><div class="field">
              <label class="field__lbl">Количество единиц валюты</label>
              <input type="text"
                     class="field__txt"
                     v-model.number="item.Cur_Scale"
                     placeholder=""
                     pattern="[0-9]+"
              >
            </div><div class="field">
              <label class="field__lbl">Курс к бел. рублю</label>
              <input type="text"
                     class="field__txt"
                     v-model.number="item.Cur_OfficialRate"
                     placeholder="0.0000"
                     pattern="[0-9].[0-9]{2,4}"
              >
            </div>
            <div class="buttons-modal">
              <button type="submit"
                      class="btn btn-btn btn-modal"
                      value="Сохранить"
                      @click="dialogSave">
                Сохранить</button>
              <button type="reset"
                      class="btn btn-btn btn-modal"
                      value="Отмена"
                      @click.prevent="dialogCancel">
                Отмена
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "FormItem",
    props: ['item'],
    data() {
      return {
        isValid: true,
        isNewItem: false
      }
    },
    mounted() {
      document.querySelector(".field__txt").focus();
    },
    methods: {
      dialogSave() {
        this.$emit('dialog-save', this.item);
      }
      ,
      dialogCancel() {
        this.$emit('dialog-cancel');
      }
      ,
      dialogClose() {
        this.$emit('dialog-close');
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "../styles/formItem";
</style>
