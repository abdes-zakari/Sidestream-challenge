<template>
  <div>
    <div class="sidenav">
      <a href="./">
        <img
          class="logo"
          src="https://avatars.githubusercontent.com/u/59832481?s=200&v=4"
          style="width: 50px"
      /></a>
      <a href="./"><Icons :name="'homeIcon'" /></a>
    </div>
    <div class="main">
      <div class="card-header">
        <h3>Errors list</h3>
        <div style="display: flex">
          <Modal :dataset="resolved" />
          <button class="btn__show" @click="showCountRequests">
            Count requests
          </button>
          <button class="btn__show" @click="showIntersection">
            Count intersections
          </button>
          <button class="btn__action" @click="undo" title="Undo actions">
            <Icons :name="'undoIcon'" />
          </button>
        </div>
      </div>
      <div class="main__ctn">
        <CardItem :dataset="backlog" :params="params.backlog" />
        <CardItem :dataset="unresolved" :params="params.unresolved" />
        <CardItem :dataset="resolved" :params="params.resolved" />
      </div>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import CardItem from '../components/CardItem.vue'
import Icons from '../components/Icons.vue'
import ToastUndoBtn from '../components/ToastUndoBtn.vue'
import Modal from '../components/Modal.vue'

export default {
  fetch() {
    this.params.resolved.textPart = (this.resolved) ? this.resolved[0].text.split(",")[1] : ""
    this.params.unresolved.textPart = (this.unresolved) ? this.unresolved[0].text.split(",")[1] : ""
    this.params.backlog.textPart = (this.backlog) ? this.backlog[0].text.split(",")[1] : ""
  },
  created() {

    this.$nuxt.$on('showToastUndoAction', () => {

      this.undoFromToast();
    })

    this.$nuxt.$on('moveItemByCheckbox', (params,items) => {

      this.updateDataset(params,items);
    })

    this.$nuxt.$on('addActions', (lastActions) => {
      if (lastActions.length>0) this.actionsHistory.push(lastActions);
    })
  },
  async asyncData({ $axios }) {

    try {
      const { resolved, unresolved, backlog } = await $axios.$get(
        "http://localhost:8000/get_lists"
      );

      const { count_requests } = await $axios.$get(
        "http://localhost:8000/get_count_requests"
      );

      const { resolved_unresolved, resolved_backlog, unresolved_backlog } = await $axios.$get(
        "http://localhost:8000/get_list_intersection_counts"
      );


      return {
        resolved,
        unresolved,
        backlog,
        count_requests,
        resolved_unresolved,
        resolved_backlog,
        unresolved_backlog
      };

    } catch (error) {
      console.log(
        `Couldn't get error lists:\n${error}\nDid you start the API?`
      );
      console.log(
        "HINT: You can comment out the full `asyncData` method and work with mocked data for UI/UX development, if you want to."
      );
    }
  },
  data() {
    return {
      resolved: [],
      unresolved: [],
      backlog: [],
      actionsHistory:[],
      resolved_unresolved: 0,
      resolved_backlog: 0,
      unresolved_backlog: 0,
      count_requests:0,
      params:{
          resolved: {
            key:"resolved",
            from: "resolved",
            to: "unresolved",
            title: "Resolved",
            errorClass:"item__resolved",
            textPart: null,
            btnName:"Unresolved",
            btnClass:"btn__unresolved"
          },
          unresolved: {
            key:"unresolved",
            from: "unresolved",
            to: "resolved",
            title: "Unresolved",
            errorClass:"item__unresolved",
            textPart: null,
            btnName:"Resolved",
            btnClass:"btn__resolved"
          },
          backlog: {
            key:"backlog",
            from: "backlog",
            to: "unresolved",
            title: "Backlog",
            errorClass:"item__backlog",
            textPart: null,
            btnName:"Unresolved",
            btnClass:"btn__unresolved"
          }
        },
    };
  },
  components: {
    CardItem,draggable,Icons,Modal
  },
  methods: {
    updateDataset: function(params,items){
        items.forEach((item) => {this.[params.to].push(item)})
        setTimeout(()=> { this.scrollToEnd(params.to,items.length)}, 100);

        items.forEach((item) => {
            this.[params.from] = this.[params.from].filter(_item => _item.index !== item.index);
        })

    },
    undo: function(){
      if (this.actionsHistory.length>0) {
        let lastActions = this.actionsHistory[this.actionsHistory.length-1];

        if (lastActions.length>0) {
          lastActions.forEach((action) => {
            this.[action.to] = this.[action.to].filter(_item => _item.index !== action.movedItem.index);
            this.[action.from].splice(action.oldIndex, 0, action.movedItem)
          })

          this.actionsHistory.pop();
        }
      }
    },
    scrollToEnd: function(elId,countEls) {
      var container = this.$el.querySelector("#"+elId);
      container.scrollTop = container.scrollHeight;
    },
    undoFromToast: function(){
      const content = {
        component: ToastUndoBtn,
        props: {
            name: 'undoIcon'
          },
          listeners: {
            undo: () => {
              this.undo()
            }
          }
      }
      this.$toast(content, {
        timeout: 4000
      });
    },
    showIntersection: function(){
      setTimeout(() => {
        this.$toast("Resolved/Unresolved: " + this.resolved_unresolved, {
          timeout: 4000
        });
      },100)

      setTimeout(() => {
        this.$toast("Resolved/Backlog: " + this.resolved_backlog, {
          timeout: 4000
        });
      },3800)

      setTimeout(() => {
        this.$toast("Unesolved/Backlog: " + this.unresolved_backlog, {
          timeout: 4000
        });
      },7000)
    },
    showCountRequests: function(){
      this.$toast("Total requests: " + this.count_requests, {
        timeout: 4000
      });
    }
  }


};
</script>
<style>
.row {
  margin: 50px;
  margin-left: 150px;
}
.Vue-Toastification__toast--default {
  background-color: rgba(43, 55, 63, 0.92) !important;
  padding: 10px 15px;
  color: #57e7be;
}

.Vue-Toastification__toast-body {
  padding-top: 10px;
  font-weight: 600;
}
</style>
