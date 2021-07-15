<template>
  <div id="app">
    <header><h1 class="caption">Курсы валют</h1></header>
    <main>
      <section id="actions">
        <form action="#" class="actions-form">
          <div class="buttons">
            <button id="addRow"
                    class="btn btn-btn btn-serve"
                    value="AddItem"
                    @click.prevent="callItemForm('new')">
              <img src="./assets/icons/png/plus.png"
                   alt="Создать запись">
            </button>
            <label for="addRow">
              Добавить
            </label>
            <button id="updateData"
                    class="btn btn-btn btn-serve"
                    value="UpdateData">
              <img src="./assets/icons/png/reload.png"
                   alt="Обновить данные"
                   @click="load"
                   :class="{rotation: loading}">
            </button>
            <label for="updateData">
              Обновить данные
            </label>
          </div>
          <form action="#" class="search-box">
            <label for="search-field">Поиск</label>
            <input type="text"
                   placeholder=""
                   :value="searchText"
                   @input.prevent="search($event.target.value)"
                   id="search-field"
                   class="search-field">
          </form>
        </form>
      </section>
      <section id="data">
        <table>
          <thead>
            <tr>
              <thf v-for="(item, index) in tHeaders"
                   :key="index"
                   :paramsData="item"
                   @change-order="sort">
              </thf>
            </tr>
          </thead>
          <tr v-for="(rate) in ratesData"
              :key="rate.ID"
              @click.prevent="activeRowId = rate.ID">
            <td :class="{active: activeRowId === rate.ID}">{{rate.ID}}</td>
            <td :class="{active: activeRowId === rate.ID}">{{rate.Cur_Abbreviation}}</td>
            <td :class="{active: activeRowId === rate.ID}">{{rate.Cur_Name}}</td>
            <td :class="{active: activeRowId === rate.ID}">{{rate.Cur_Scale}}</td>
            <td :class="{active: activeRowId === rate.ID}">{{rate.Cur_OfficialRate}}</td>
            <td :class="{active: activeRowId === rate.ID}">
              <div class="controls-inline">
                <button class="btn btn-pict"
                        @click.prevent="callItemForm('edit', rate.ID)">
                  <img src="./assets/icons/png/edit.png"
                             alt="Изменить">
                </button>
                <button class="btn btn-pict"
                        @click.prevent="callConfirmBox(rate.ID)">
                  <img src="./assets/icons/png/delete.png"
                             alt="Удалить">
                </button>
              </div>
            </td>
          </tr>
        </table>
      </section>
    </main>
    <confirm-box v-if="isConfirmBox"
                 @confirm-no="confirmBoxNo"
                 @confirm-yes="confirmBoxYes">
      Вы действительно хотите удалить запись?
    </confirm-box>
    <form-item v-if="isItemForm"
               :item="rateEdit"
               @dialog-cancel="isItemForm = !isItemForm"
               @dialog-close="isItemForm = !isItemForm"
               @dialog-save="onSave"
    ></form-item>
  </div>
</template>

<script>
  import axios from 'axios';
  import TableHeaderField from './components/TableHeaderField';
  import ConfirmBox from './components/Confirm';
  import FormItem from './components/FormItem';

  export default {
    name: 'app',
    components: {'thf': TableHeaderField,
                  'confirm-box': ConfirmBox,
                  'form-item': FormItem
    },
    data () {
      return {
        ratesData: [],
        loading: false,
        isError: false,
        isConfirmBox: false,
        isItemForm: false,
        isFiltered: false,
        isActionNew: false,
        activeRowId: 0,
        rateEdit: {},
        delItemId: 0,
        sortedParams: {},
        searchText: '',
        tmpData: [],
        tHeaders: [
                    {text: 'Номер строки', sortable: true, name: 'ID'},
                    {text: 'Буквенный код валюты', sortable: true, name: 'Cur_Abbreviation'},
                    {text: 'Наименование валюты', sortable: true, name: 'Cur_Name'},
                    {text: 'Количество единиц валюты', sortable: true, name: 'Cur_Scale'},
                    {text: 'Курс к бел. рублю', sortable: true, name: 'Cur_OfficialRate'}
                  ],
        emptyItem: {
          Cur_ID: 0,
          Date: new Date(),
          Cur_Abbreviation: '',
          Cur_Scale: 1,
          Cur_Name: '',
          Cur_OfficialRate: 0,
          ID: 0
        }
      }
    },
    mounted() {
      if(sessionStorage.getItem('currencies')) {
        try {
          this.ratesData = JSON.parse(sessionStorage.getItem('currencies'));
        } catch(e) {
          sessionStorage.removeItem('currencies');
        }
      } else {
        this.load();
      }
    },
    methods: {
      load() {
        this.getRates();
        this.searchText = '';
        if (!this.isError) {
          setTimeout(() => {
            for(let idx in this.ratesData) {
              this.$set(this.ratesData[idx],'ID', +idx + 1);
            }
            const parsedData = JSON.stringify(this.ratesData);
            sessionStorage.setItem('currencies', parsedData);
            if(!this.isEmptyObject(this.sortedParams)) {
              this.sort();
            }
          }, 800)
        }
      },
      cloneObject(obj) {
        return JSON.parse(JSON.stringify(obj));
      },
      getRates() {
        this.loading = true;
        axios
          .get('https://www.nbrb.by/api/exrates/rates?periodicity=0')
          .then(response => {
            this.ratesData = response.data;
          })
          .catch(error => {
            console.log(error);
            this.isError = true;
          })
          .finally(() => (this.loading = false));
      },
      saveChangedData() {
        let currentData = this.getCurrentSource();
        const parsedData = JSON.stringify(currentData);
        sessionStorage.setItem('currencies', parsedData);
      },
      sort(statusOrder = -100, nameField = '') {
        if (statusOrder !== -100) {
          if (statusOrder === 0) {
            delete this.sortedParams[nameField];
          } else {
            this.sortedParams[nameField] = statusOrder;
          }
        }
        this.ratesData.sort(this.compare(this.isEmptyObject(this.sortedParams) ? {ID: 1} : this.sortedParams));
        if (this.tmpData.length) {
          this.tmpData.sort(this.compare(this.isEmptyObject(this.sortedParams) ? {ID: 1} : this.sortedParams));
        }
      },
      isEmptyObject(obj) {
        return Object.keys(obj).length === 0;
      },
      search(value) {
        this.searchText = value;
        if (this.tmpData.length === 0) {
          this.tmpData = this.cloneObject(this.ratesData);
        }
        this.ratesData = this.tmpData.filter((item) => {
          for(let key in item){
            if (this.tHeaders.findIndex((item) => item.name === key) === -1) {
              continue; // что бы искало только по нужным полям
            }
            if(String(item[key]).toLowerCase().indexOf(value.toLowerCase()) !== -1){
              return true;
            }
          }
          return false;
        });
        if (value === '') {
          this.tmpData = [];
          this.isFiltered = false;
        } else {
          this.isFiltered = true;
        }
      },
      //сортирует объект по его свойствам (по нескольким, если field - объект)
      compare(field, order) {
        var len = arguments.length;
        if(len === 0) {
          return (a, b) => (a < b && -1) || (a > b && 1) || 0;
        }
        if(len === 1) {
          switch(typeof field) {
            case 'number':
              return field < 0 ?
                ((a, b) => (a < b && 1) || (a > b && -1) || 0) :
                ((a, b) => (a < b && -1) || (a > b && 1) || 0);
            case 'string':
              return (a, b) => (a[field] < b[field] && -1) || (a[field] > b[field] && 1) || 0;
          }
        }
        if(len === 2 && typeof order === 'number') {
          return order < 0 ?
            ((a, b) => (a[field] < b[field] && 1) || (a[field] > b[field] && -1) || 0) :
            ((a, b) => (a[field] < b[field] && -1) || (a[field] > b[field] && 1) || 0);
        }
        let fields, orders;
        if(typeof field === 'object') {
          fields = Object.getOwnPropertyNames(field);
          orders = fields.map(key => field[key]);
          len = fields.length;
        } else {
          fields = new Array(len);
          orders = new Array(len);
          for(let i = len; i--;) {
            fields[i] = arguments[i];
            orders[i] = 1;
          }
        }
        return (a, b) => {
          for(let i = 0; i < len; i++) {
            if(a[fields[i]] > b[fields[i]]) return orders[i];
            if(a[fields[i]] < b[fields[i]]) return -orders[i];
          }
          return 0;
        };
      },
      callItemForm(mode, id = 0) {
        switch (mode) {
          case "new":
            this.isActionNew = true;
            this.rateEdit = this.cloneObject(this.emptyItem);
            this.rateEdit.ID = this.getCurrentSource().reduce((max, cur) => cur.ID > max ? cur.ID : max, 0) + 1;
            break;
          case "edit":
            this.isActionNew = false;
            const pos = this.ratesData.findIndex((item) => item.ID === id);
            this.rateEdit = this.cloneObject(this.ratesData[pos]);
            break;
        }
        this.isItemForm = true;
      },
      onSave(itemRate) {
        let currentData = this.getCurrentSource();
        if (this.isActionNew) {
          currentData.push(itemRate);
        } else {
          const pos = currentData.findIndex((item) => item.ID === itemRate.ID);
          currentData.splice(pos, 1, itemRate);
        }
        this.saveChangedData();
        if (this.isFiltered) {
          this.search(this.searchText);
        }
        if(!this.isEmptyObject(this.sortedParams)) {
          this.sort();
        }
        this.isItemForm = false;
      },
      getCurrentSource() {
        return this.isFiltered ? this.tmpData : this.ratesData;
      },
      callConfirmBox(id) {
        this.delItemId = id;
        this.isConfirmBox = true;
      },
      confirmBoxToggle() {
        this.isConfirmBox = !this.isConfirmBox;
      },
      confirmBoxNo() {
        this.isConfirmBox = false;
        this.delItemId = 0;
      },
      confirmBoxYes() {
        const pos = this.ratesData.findIndex((item) => item.ID === this.delItemId);
        if(pos !== -1) {
          this.ratesData.splice(pos, 1);
        }
        this.saveChangedData();
        this.delItemId = 0;
        this.confirmBoxToggle();
      },
    }
  }
</script>

<style lang="scss">
  @import './styles/app';
</style>
