<template lang="html">
  <div class="content">
    <div class="left-content">
      <board :sizes="getBoardSize"></board>

      <div class="bottom-bar">
        <chat-panel></chat-panel>
        <fieldinfo-panel></fieldinfo-panel>
        <!-- <file-select></file-select> -->
      </div>
    </div>

    <div class="right-bar">
      <charactor-detail></charactor-detail>

      <initiative></initiative>
      <boardselect-panel></boardselect-panel>
    </div>

    <componentadd-dialog v-if="stores.ApplicationStore.dialogState === 'componentadd'"></componentadd-dialog>
    <file-dialog v-if="stores.ApplicationStore.dialogState === 'file'"></file-dialog>
  </div>
</template>

<style scoped>
.content{
  width: 100%;
  height: 100%;

  display: flex;
  /*flex-direction: column;*/
  align-items: flex-start;
  justify-content: flex-start;
}

.left-content{
  width: calc(100% - 300px);
  height: 100%;
}

.bottom-bar{
  width: 100%;
  height: 180px;
  display: flex;
}

.right-bar{
  width: 350px;
  height: 100%;
}
</style>

<script>
const WSManager = require("../../utilities/WSManager")();
module.exports = {
  data: () => {
    return {
      stores: require("../../stores/Stores.js")
    };
  },
  beforeRouteEnter(from, to, next){
    next(vm=>{
      vm.initialize();
    });
  },
  computed: {
    getBoardSize(){
      return {
        x: this.stores.BoardsStore.boards[0].x,
        y: this.stores.BoardsStore.boards[0].y
      };
    }
  },
  methods: {
    initialize(){

      WSManager.database().ref("/boards/state").once("value")
      .then((data)=>{
        this.stores.BoardsStore.boards[0].background = data.val().background;
        this.stores.BoardsStore.boards[0].music = data.val().music;
        console.log(
          this.stores.BoardsStore.boards[0]
        );
      });

      const components = WSManager.database().ref("boards/components/components");

      components.on("child_added", (data) => {
        if(!this.stores.ComponentsStore.components.find((component)=>{
          return data.key == component.key;
        })){
          console.log("Add");
          this.stores.ComponentsStore.components.push(
            Object.assign(
              Object.create(null),
              data.val(),
              {
                key: data.key
              }
            )
          );
        }
      });

      components.on("child_changed", (data) => {
        console.log("get", data.val());
        const target = data.val();
        this.stores.ComponentsStore.components = this.stores.ComponentsStore.components.map((component)=>{
          if(component.key == target.key){
            component = target;
            console.log("Search done");
          }
          return component;
        });

        if(this.stores.ComponentsStore.active.key == target.key){
          this.stores.ComponentsStore.active = target;
        }

        console.log(this.stores.ComponentsStore.components);
      });

      components.on("child_removed", (data) => {
        this.stores.ComponentsStore.components = this.stores.ComponentsStore.components.filter((component)=>{
          return component.key != data.key;
        });
      });
    }
  }
};
</script>
