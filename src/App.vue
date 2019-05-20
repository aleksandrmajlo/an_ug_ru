<template>
    <div id="app">
        
        <div class="count_block">
            <div class="item">
                <span class="title">Количество обектов (в базе разработчиков) :</span><strong>{{animatedcountObject}}</strong>
            </div>
            <div class="item">
                <span class="title">Количество обектов (в базе сайте) :</span><strong>{{animatedcountSite}}</strong>
            </div>
        </div>

        <div class="block_tab">
           <vue-tabs>
              <v-tab title="Обновление">

                <div class="btn_block">
                     <button :disabled="disabled" @click.prevent="send" class="button">{{textUpdate}}</button>
                 </div>

                 <div class="result_block" v-show="sendAction">
                       <a class="button stopUpdate" @click.prevent="stopUpdate">{{stopUpdateText}}</a>
                 </div>

                  <div class="flex-block">


                      <post title="Добавлено записей"  :data="counterAdddata"></post>
                      <post title="Обновлено записей"   :data="counterUpdatedate"></post>

                      <div class="result_block" >
                          <remove v-show="dataRemove.length>0" :data="dataRemove"></remove>
                          <error v-show="error.title.length>0" :data="error.title" title="title"></error>
                          <error v-show="error.object_code.length>0" :data="error.object_code" title="object_code"></error>
                      </div>

                  </div>


              </v-tab>
              <v-tab title="Очистка">
                      <div class="remove_block">
                        <span>Удалить все данные связанные с объектами</span>
                        <button href="#" class="button" @click.prevent="removeAll">{{textRemove}}</button>
                      </div>
              </v-tab>
              
              <v-tab title="Агенты">
                      <div class="alert_info" v-show="countAgentDelete">
                         Удалено <b>{{countAgentDelete}}</b> агента с базы данных
                      </div>  
                      <div class="remove_block">
                        <span>Удалить всех агентов</span>
                        <button href="#" class="button" @click.prevent="removeAgent">{{textRemove}}</button>
                      </div>
              </v-tab>
              

           </vue-tabs>
        </div>      

    </div>
</template>

<script>
import axios from "axios";
import { TweenMax } from "greensock";
import { VueTabs, VTab } from "vue-nav-tabs/dist/vue-tabs.js";

import error from "./components/Error.vue";
import post from "./components/Post.vue";
import remove from "./components/Remove.vue";

let fullAddress =
  location.protocol +
  "//" +
  location.hostname +
  (location.port ? ":" + location.port : "");
if (location.port) {
  var ajax_pl_url = "http://anugorod.local/wp-admin/admin-ajax.php";
  var site_pl_url = "http://anugorod.local/wp-json/wp/v2/tm-property";
} else {
  var ajax_pl_url = fullAddress + "/wp-admin/admin-ajax.php";
  var site_pl_url = fullAddress + "/wp-json/wp/v2/tm-property";
}

const limitSet = 1;

const textUpdate = {
  default: "Обновить",
  check: "Удаление неактивных объектов",
  send: "Идет обновление ...."
};

const textRemove = {
  default: "Удалить",
  send: "Идет удаление ...."
};

export default {
  data() {
    return {
      countObject: 0,
      countSite: 0,
      textUpdate: textUpdate.default,
      textRemove: textRemove.default,

      // отправка
      stopUpdatevalue: false,
      disabled: true,
      sendAction: false,
      // остановка
      stopUpdateText: "Остановить",
    

      countPage: 0,
      counter: 0,

      counterAdd: 0,
      counterAdddata: [],
      counterUpdate: 0,
      counterUpdatedate: [],

      dataRemove: [],
      
      limit: limitSet,
      //error
      error: {
        title: [],
        object_code: []
      },
      // количество удаленных агентов
      countAgentDelete: false,
    };
  },
  components: {
    VueTabs,
    VTab,
    error,
    post,
    remove
  },
  mounted() {

    // console.log("type ajax");

    let form_data = new FormData();
    form_data.append("action", "count_ob");
    axios
      .post(ajax_pl_url, form_data)
      .then(response => {
        TweenLite.to(this.$data, 5, { countObject: response.data });
        this.disabled = false;
        this.countPage = Math.ceil(response.data / this.limit);
      })
      .catch(error => {
        console.log(error);
      });
    this.countSitef();
  },
  computed: {

    animatedcountObject: function() {
      return this.countObject.toFixed(0);
    },
    animatedcountSite: function() {
      return this.countSite.toFixed(0);
    }
  },
  methods: {

    send() {
      this.disabled = true;
      this.checkBd();
    },

    // аякс на сверку
    checkBd() {

      let form_data = new FormData();

      form_data.append("action", "property_check");
      form_data.append("type", "ajax");



      this.textUpdate = textUpdate.check;
      axios.post(ajax_pl_url, form_data).then(response => {
        this.textUpdate = textUpdate.send;
        this.sendAction = true;
        if(response.data){
          this.dataRemove=response.data;
        }
        this.loops();
      });
    },

    loops() {
      let form_data = new FormData();
      form_data.append("action", "property_update");
      form_data.append("limit", this.limit);
      form_data.append("counter", this.counter++);

      axios.post(ajax_pl_url, form_data).then(response => {
        if (response.data.add && response.data.add.length > 0) {
          response.data.add.forEach(element => {
            this.counterAdddata.push(element);
          });
        }

        if (response.data.update && response.data.update.length > 0) {
          response.data.update.forEach(element => {
            this.counterUpdatedate.push(element);
          });
        }


        if (response.data.title && response.data.title.length > 0) {
          response.data.title.forEach(element => {
            this.error.title.push(element);
          });
        }
        if (response.data.object_code && response.data.object_code.length > 0) {
          response.data.object_code.forEach(element => {
            this.error.object_code.push(element);
          });
        }
        if (this.counter < this.countPage && !this.stopUpdatevalue) {
          this.loops();
          this.countSitef();
        } else {
          this.disabled = false;
          this.textUpdate = textUpdate.default;
          this.sendAction = false;
          this.stopUpdatevalue = false;
          this.stopUpdateText="Остановить";
          this.countSitef();
        }
      });
    },
    // количество постов
    countSitef() {
      axios
        .get(site_pl_url)
        .then(response => {
          TweenLite.to(this.$data, 1, {
            countSite: response.headers["x-wp-total"]
          });
        })
        .catch(error => {
          console.log(error);
        });
    },
    // остановка процесса добавлени
    stopUpdate() {
      this.stopUpdatevalue = true;
      this.stopUpdateText="Идет завершение цикла - не обновляйте страницу ...";
    },
    // удаление записей
    removeAll() {
      let form_data = new FormData();
      form_data.append("action", "property_remove");
      this.textRemove = textRemove.send;
      axios.post(ajax_pl_url, form_data).then(response => {
        this.textRemove = textRemove.default;
        this.countSitef();
      });
    },
    // удаление агентов
    removeAgent(){
      let form_data = new FormData();
      form_data.append("action", "agent_remove");
       this.textRemove = textRemove.send;
      axios.post(ajax_pl_url, form_data).then(response => {
        this.textRemove = textRemove.default;
        this.countAgentDelete= response.data.count;
      });
    }

  },
  watch: {}
};
</script>

<style lang="scss">
#app {
  margin-top: 20px;
  padding: 1em;
  a {
    box-shadow: inherit !important;
  }
  .tab-content {
    background: #fff;
    box-sizing: border-box;
    padding: 2em;
  }
  .btn_block {
     margin-bottom:20px;
  }
  .item {
    margin-bottom: 10px;
  }
  .title {
    font-weight: bold;
    margin-right: 10px;
  }
  .result_block {

    .item {
      margin-bottom: 10px;
      span {
        font-size: 120%;
        font-weight: bolder;
      }
    }
  }
  .stopUpdate {
    background: rgb(28, 28, 189);
    color: #fff;
    display: inline-block;
    margin-bottom: 20px;
    min-width: 140px;
    text-align: center;
  }
  .remove_block {
    margin-top: 20px;
    display: flex;
    align-items: center;
    button {
      margin-left: 10px;
      border-color: red;
      background: red;
      color: #fff;
    }
  }
}

.vue-tabs.stacked {
  display: flex;
}
.vue-tabs .tabs__link {
  text-decoration: none;
  color: gray;
}
.vue-tabs .nav {
  margin-bottom: 0;
  margin-top: 0;
  padding-left: 0;
  list-style: none;
}
.vue-tabs .nav:before,
.vue-tabs .nav:after {
  content: " ";
}
.vue-tabs .nav {
  margin-bottom: 0;
  margin-top: 0;
  padding-left: 0;
  list-style: none;
}
.vue-tabs .nav:before,
.vue-tabs .nav:after {
  content: " ";
  display: table;
}
.vue-tabs .nav:after {
  clear: both;
}
.vue-tabs .nav > li {
  position: relative;
  display: block;
}
.vue-tabs .nav > li > a {
  position: relative;
  display: block;
  padding: 10px 15px;
}
.vue-tabs .nav > li > a:hover,
.vue-tabs .nav > li > a:focus {
  text-decoration: none;
  background-color: #eeeeee;
}
.vue-tabs .nav > li span.title {
  display: flex;
  justify-content: center;
}
.vue-tabs .nav > li.disabled > a {
  color: #777777;
}
.vue-tabs .nav > li.disabled > a:hover,
.vue-tabs .nav > li.disabled > a:focus {
  color: #777777;
  text-decoration: none;
  cursor: not-allowed;
  background-color: transparent;
  border-color: transparent;
}
.vue-tabs .nav .nav-divider {
  height: 1px;
  margin: 9px 0;
  overflow: hidden;
  background-color: #e5e5e5;
}
.vue-tabs .nav > li > a > img {
  max-width: none;
}

.vue-tabs .nav-tabs {
  border-bottom: 1px solid #ddd;
}
.vue-tabs .nav-tabs > li {
  float: left;
  margin-bottom: -1px;
}
.vue-tabs .nav-tabs > li > a {
  margin-right: 2px;
  line-height: 1.42857;
  border: 1px solid transparent;
  border-radius: 4px 4px 0 0;
}
.vue-tabs .nav-tabs > li > a:hover {
  border-color: #eeeeee #eeeeee #ddd;
}
.vue-tabs .nav-tabs > li.active > a,
.vue-tabs .nav-tabs > li.active > a:hover,
.vue-tabs .nav-tabs > li.active > a:focus {
  color: #555555;
  background-color: #fff;
  border: 1px solid #ddd;
  border-bottom-color: transparent;
  cursor: default;
}

.vue-tabs .nav-pills > li {
  float: left;
}
.vue-tabs .nav-pills > li > a {
  border-radius: 4px;
}
.vue-tabs .nav-pills > li + li {
  margin-left: 2px;
}
.vue-tabs .nav-pills > li.active > a,
.vue-tabs .nav-pills > li.active > a:hover,
.vue-tabs .nav-pills > li.active > a:focus {
  color: #fff;
  background-color: #337ab7;
}

.vue-tabs .nav-stacked > li {
  float: none;
}
.vue-tabs .nav-stacked > li + li {
  margin-top: 2px;
  margin-left: 0;
}

.vue-tabs .nav-justified,
.vue-tabs .nav-tabs.nav-justified {
  width: 100%;
}
.vue-tabs .nav-justified > li,
.vue-tabs .nav-tabs.nav-justified > li {
  float: none;
}
.vue-tabs .nav-justified > li > a,
.vue-tabs .nav-tabs.nav-justified > li > a {
  text-align: center;
  margin-bottom: 5px;
}
.vue-tabs .nav-justified > .dropdown .dropdown-menu {
  top: auto;
  left: auto;
}
@media (min-width: 768px) {
  .vue-tabs .nav-justified > li,
  .vue-tabs .nav-tabs.nav-justified > li {
    display: table-cell;
    width: 1%;
  }
  .vue-tabs .nav-justified > li > a,
  .vue-tabs .nav-tabs.nav-justified > li > a {
    margin-bottom: 0;
  }
}

.vue-tabs .nav-tabs-justified,
.vue-tabs .nav-tabs.nav-justified {
  border-bottom: 0;
}
.vue-tabs .nav-tabs-justified > li > a,
.vue-tabs .nav-tabs.nav-justified > li > a {
  margin-right: 0;
  border-radius: 4px;
}
.vue-tabs .nav-tabs-justified > .active > a,
.vue-tabs .nav-tabs.nav-justified > .active > a,
.vue-tabs .nav-tabs-justified > .active > a:hover,
.vue-tabs .nav-tabs.nav-justified > .active > a:hover,
.vue-tabs .nav-tabs-justified > .active > a:focus,
.vue-tabs .nav-tabs.nav-justified > .active > a:focus {
  border: 1px solid #ddd;
}
@media (min-width: 768px) {
  .vue-tabs .nav-tabs-justified > li > a,
  .vue-tabs .nav-tabs.nav-justified > li > a {
    border-bottom: 1px solid #ddd;
    border-radius: 4px 4px 0 0;
  }
  .vue-tabs .nav-tabs-justified > .active > a,
  .vue-tabs .nav-tabs.nav-justified > .active > a,
  .vue-tabs .nav-tabs-justified > .active > a:hover,
  .vue-tabs .nav-tabs.nav-justified > .active > a:hover,
  .vue-tabs .nav-tabs-justified > .active > a:focus,
  .vue-tabs .nav-tabs.nav-justified > .active > a:focus {
    border-bottom-color: #fff;
  }
}
.vue-tabs .tab-content > .tab-pane {
  display: none;
}
.vue-tabs .tab-content > .active {
  display: block;
}
.vue-tabs section[aria-hidden="true"] {
  display: none;
}
.alert_info{
    background: rgb(28, 28, 189);
    color: #fff;
    display: inline-block;
    margin-bottom: 20px;
    min-width: 140px;
    text-align: center;
    padding: 20px;
    text-transform: uppercase;
}
.flex-block{
    display:flex;  
    align-items: flex-start;
    .postBlock{
       width: 32%;
       margin-right: 1%;
    }
}
</style>



