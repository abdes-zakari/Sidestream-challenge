<template>
  <div class="msg-ctn" @click.stop="undo">
    <div class="text-msg">Undo last action</div>
    <div class="svg-ctn">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="#57e7be"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="feather feather-rotate-ccw"
      >
        <polyline points="1 4 1 10 7 10"></polyline>
        <path d="M3.51 15a9 9 0 1 0 2.13-9.36L1 10"></path>
      </svg>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ToastUndoBtn',
  methods: {
      /**
      * undo action from toast
      *
      */
      undo() {
        /**
        * Triggers undo action
        *
        */
        this.$emit("undo");
        /**
        * Triggers when close toast
        *
        */
        this.$emit('close-toast');
      }
  }
};
</script>

<style>
.msg-ctn {
  display: flex;
  margin-top: 10px;
  font-weight: 600;
  cursor: pointer;
}

svg,
.text-msg {
  color: #57e7be;
}
.svg-ctn {
  margin-left: 15px;
}

svg:hover {
  stroke: #fff;
}
</style>
