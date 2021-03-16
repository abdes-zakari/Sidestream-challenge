<template>
  <div class="card__ctn">
    <div class="card__title">
      <div>
        {{ params.title }}
        <span
          class="badge__item"
          v-bind:class="params.errorClass"
          >{{  dataset.length }}</span
        >
      </div>
      <button
        v-bind:class="params.btnClass"
        :title="'Make it '+ params.btnName"
        @click="moveItemByCheckbox"
      >
        {{ params.btnName }}
      </button>
    </div>
    <draggable
      class="card__body scrollbar"
      v-bind:id="params.key"
      tag="div"
      group="errorList"
      :list="dataset"
      @end="onEnd"
      @add="onAdd"
    >
      <transition-group type="transition" name="list-complete">
        <div
          class="card__body__item list-complete-item"
          v-for="error in dataset"
          :key="error.index"
          :data-id="error.index"
          :data-object="JSON.stringify(error)"
        >
          <div class="wrapper__error">
            <div
              class="error__code badge__item"
              v-bind:class="params.errorClass"
            >
              {{ error.code }}
            </div>
            <div
              class="error__text badge__item"
              v-bind:class="params.errorClass"
            >
              {{ error.text | setText(params.textPart)}}
            </div>
          </div>
          <div class="error__text badge__item check__input">
            <input
              type="checkbox"
              name=""
              v-model="checkedItems"
              v-bind:value="error.index"
            />
          </div>
        </div>
      </transition-group>
    </draggable>
  </div>
</template>

<script>
import draggable from "vuedraggable";
export default {
  name: 'CardItem',
  props: {
    dataset: Array,
    params: Object
  },
   data() {
    return {
              checkedItems: []
          }
  },
  filters: {
  	setText(text,textPart){
  		return text.split(",")[0]+", "+textPart;
  	}
  },
  components: {
    draggable
  },
  methods:{
  	/**
  	* on end drag event from origin list 
  	*
  	*/
  	onEnd(e) {
  		let movedItem = JSON.parse(e.item.dataset.object);
  		if (movedItem) {
	  		let lastActions = [];
	  		let action = {}
	  		action.movedItem = movedItem
	  		action.from = e.from.parentElement.id
	  		action.to = e.to.parentElement.id
	  		action.newIndex = e.newIndex
	  		action.oldIndex = e.oldIndex
	  		lastActions.push(action);
	  		/**
	  		* Triggers addActions in index.vue
	  		*
	  		*/
	  		$nuxt.$emit('addActions',lastActions)
	  		/**
	  		* Triggers showToastUndoAction action
	  		*
	  		*/
	  		$nuxt.$emit('showToastUndoAction')
  		}
  	},
  	/**
  	* after drag item to new list
  	*
  	*/
  	onAdd(e){
  	},
  	/**
  	* when items moved per checkboxes
  	*
  	*/
  	moveItemByCheckbox(){
  		if (this.checkedItems.length>0) {
  			let _checkeds = [];

  			 _checkeds = this.checkedItems.map((index) => {
  				return  this.dataset.find(x => x.index === index)
  			})

  			let lastActions = [];

  			this.checkedItems.forEach((value) => {
  				let action = {}
  				action.movedItem = this.dataset.find(x => x.index === value)
  				action.from = this.params.from
  				action.to = this.params.to
  				action.oldIndex = this.dataset.findIndex(x => x.index === value)
  				lastActions.push(action);
  			})
  			/**
  			* Triggers moveItemByCheckbox in index.vue
  			*
  			*/
  			$nuxt.$emit('moveItemByCheckbox',this.params,_checkeds)
  			/**
  			* Triggers addActions in index.vue
  			*
  			*/

  			$nuxt.$emit('addActions',lastActions)
  			/**
  			* Triggers showToastUndoAction action
  			*
  			*/
  			$nuxt.$emit('showToastUndoAction')

  			this.checkedItems = [];
  		}
  	}
  }

};
</script>

<style>
.list-complete-item {
  transition: all 1s;
}
.list-complete-enter,
.list-complete-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}
</style>
