<template>
  <div class="home">
    <!-- <p>
      <button class="btn" @click="changeIndex(1)">星空</button>
      <button class="btn" @click="changeIndex(2)">雪花</button>
      <button class="btn" @click="changeIndex(3)">银河星系</button>
    </p>

    <div class="view">
      <div v-if="index === 1">
        <Star />
      </div>
      <div v-else-if="index === 2">
        <snowFlake />
      </div>
      <div v-else-if="index === 3"> -->
    <!-- <galaxy /> -->
    <!-- <rayCaster /> -->
    <!-- <page /> -->
    <!-- <cannon /> -->
    <cannon2 />
    <!-- </div> -->
    <!-- </div> -->
  </div>
</template>

<script lang="ts">
import { ref, defineComponent } from "vue";
import Star from "../components/star.vue";
import snowFlake from "../components/snowflake.vue";
import galaxy from "../components/galaxy.vue";
import rayCaster from "../components/rayCaster.vue";
import page from "../components/3dPage.vue";
import cannon from "../components/cannon.vue";
import cannon2 from "../components/cannon2.vue";

export default defineComponent({
  components: {
    Star,
    snowFlake,
    galaxy,
    rayCaster,
    page,
    cannon,
    cannon2,
  },
  setup() {
    let index = ref(1);
    let changeIndex = (num: number) => {
      index.value = num;
      let canvasTags = document.getElementsByTagName("canvas");
      if (canvasTags.length > 1) {
        document.body.removeChild(canvasTags[0]);
      }
    };
    return {
      index,
      changeIndex,
    };
  },
});
</script>

<style scoped>
.btn {
  width: 100px;
  height: 45px;
  background: rgb(223, 180, 180);
  border: 0;
  margin: 5px;
  color: white;
  font-size: 18px;
  cursor: pointer;
  border-radius: 5px;
}
</style>
