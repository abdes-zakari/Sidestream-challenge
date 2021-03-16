<template>
  <div>
    <button
      id="show-modal"
      @click="openModal"
      class="btn__show"
      style="height: 34px"
    >
      Count resolved
    </button>
    <transition name="slide-fade" mode="out-in">
      <div class="modal-mask" v-if="showModal">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="table-wrapper scrollbar">
              <table class="styled-table">
                <thead>
                  <tr>
                    <th>Code error</th>
                    <th>How many times resolved</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="list in listcount">
                    <td>{{ list.code }}</td>
                    <td>{{ list.count }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
            <button
              class="modal-default-button btn__show"
              @click="showModal=false"
            >
              CLOSE
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'Modal',
  data: function() {
      return {
        showModal: false,
        listcount:[]
      }
    },
  props: {
    name: String,
    dataset:Array
  },
  methods:{
  	/**
  	* openModal click
  	*
  	*/
  	openModal: function(){
  		this.showModal = true
  		
  		/*set listcount from getData()*/
  		this.getData().then(list => this.listcount = list.sort((a, b) => a.count > b.count ? -1 : 1))

  	},
  	/**
  	* get how many times a certain error.code was resolved list 
  	*
  	*/
  	getData: async function(){
  		const { data } = await this.$axios.$post(
        	"http://localhost:8000/get_resolved_code",
        	{
			    resolved: this.dataset
			},
			{
				headers: {
			    'Content-Type': 'application/json'
			  }
			}
        );
        return data;
  	}
  }

};
</script>

<style>
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  width: 460px;
  margin: 0px auto;
  max-height: 500px;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
  padding-bottom: 20px;
}
td,
th {
  text-align: center;
}

.table-wrapper {
  display: block;
  overflow-y: auto;
  height: 500px;
  padding: 20px;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.modal-default-button {
  float: right;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.5s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter,
.slide-fade-leave-to {
  transform: translateY(10px);
  opacity: 0;
}

.styled-table {
  border-collapse: collapse;
  font-size: 0.9em;
  min-width: 400px;
  width: 100%;
  margin-bottom: 20px;
}
.styled-table thead tr {
  background-color: #2b373f;
  color: #57e7be;
  text-align: left;
}

.styled-table th,
.styled-table td {
  padding: 12px 15px;
}

.styled-table tbody tr {
  border-bottom: 1px solid #dddddd;
}

.styled-table tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

.styled-table tbody tr:last-of-type {
  border-bottom: 2px solid #2b373f;
}
</style>
