<template>
  <div>
    <div class="div">
      <button @click="redcount">上一页</button>
      <button @click="addcount">下一页</button>
      {{tot}}页数
      <button style="" @click="scaleFun=scaleFun1">还原</button>
      <button @click="scaleFun=scaleFun2">缩小</button>
    </div>
    <vue-pdf-embed :source=source
      class="vue-pdf-embed"
      :style="scaleFun"
      :page="real_count" />
      </div>
</template> 

<script lang="ts" setup>
import type { AppContext } from "@netless/window-manager";
import VuePdfEmbed from "vue-pdf-embed";
import { createLoadingTask } from "vue3-pdfjs/esm"; // 获得总页数
import { computed, inject, onMounted, ref, watchEffect } from "vue";
const context = inject<AppContext>("context");
if (!context) throw new Error("must call provide('context') before mount App");
const source=ref("http://storage.xuetangx.com/public_assets/xuetangx/PDF/PlayerAPI_v1.0.6.pdf")
const storage = context.createStorage("counter", { count: 1 });
const real_count = ref(storage.state.count);
const tot = ref(0);
const scaleFun=ref(``)
const scaleFun1=ref(`transform:scale(1.1)`)
const scaleFun2=ref(`transform:scale(0.8)`)
const addcount=()=>{
  if(count.value<tot.value){
    count.value++
  }
}
const redcount=()=>{
  if(count.value>1){
    count.value--
  }
}
const count = computed<number>({
  get: () => real_count.value,
  set: (count) => storage.setState({ count }),
});
const zoom = computed<number>({
  get: () => real_count.value,
  set: (count) => storage.setState({ count }),
});
onMounted(() => { 
 
   const loadingTask = createLoadingTask(source.value);
    loadingTask.promise.then((pdf) => {
       tot.value= pdf.numPages;
       count.value=1
    });

  storage.addStateChangedListener(() => {
    real_count.value = storage.state.count;
  })
}
);

watchEffect(() => {
  // console.log("App.vue: count =", count.value);
});
</script>
<style scoped>
.video-js {
  position: absolute;
  left: 0;
  bottom: 0;
}
</style>