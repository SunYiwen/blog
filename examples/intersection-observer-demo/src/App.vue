<template>
  <link
    href="https://fonts.font.im/css?family=Kirang+Haerang"
    rel="stylesheet"
  />
  <link href="https://fonts.font.im/css?family=Comfortaa" rel="stylesheet" />

  <view class="demo-body">
    <header class="demo-header">Intersection Observer Demo</header>
    <view class="demo-tags">
      <view
        v-for="(tag, index) in tagList"
        :key="index"
        :class="['demo-tag', index == selecter.activeKey ? 'active' : '']"
      >
        {{ tag }}
      </view>
    </view>

    <view class="demo-scroll" id="demo-scroll">
      <view
        v-for="(height, index) in heightList"
        :key="index"
        class="demo-box"
        :style="{ height: `${height}px` }"
        :id="`box-${index}`"
      >
        {{ index }}
      </view>
    </view>
  </view>
</template>

<script lang="ts">
import { Vue, setup } from "vue-class-component";
import { onMounted, ref, reactive } from "vue";

function useSelected() {
  let activeKey = ref(0);

  const tagSelected = (index: number) => {
    activeKey.value = index;
  };

  const queue: Array<number> = reactive([]);
  const queueSet: Set<number> = new Set();
  const addObserver = () => {
    const root = document.getElementById("#demo-scroll");
    let options = {
      root,
      threshold: [0],
    };

    const handleIntersect = (entries: any) => {
      entries.forEach((entry: any) => {
        
        // 解析id
        const id = entry?.target?.id.split("-")[1] || 0;

        // 相交
        if (entry.isIntersecting) {
          if (!queueSet.has(id)) {
            if (queue.length > 0 && Math.abs(id - queue[queue.length - 1]) !== 1) {
              // console.log("reverse:", queue);
              queue.reverse();
            }

            queue.push(id);
            queueSet.add(id);
          }
        } else {
          if (queue.length > 0 && (activeKey.value == 0 || activeKey.value == 4) && queue[queue.length - 1] == activeKey.value) {
            // console.log("reverse:", queue);
            queue.reverse();
          }

          if (id == queue[0]) {
            queueSet.delete(queue[0]);
            queue.shift();
          }
        }

        console.log("queue:", queue);

        if (queue.length > 0) {
          if (queue[queue.length - 1] == 0 || queue[queue.length - 1] == 4) {
            activeKey.value = queue[queue.length - 1];
          } else {
            activeKey.value = queue[0];
          }
        }
      });
    };

    const observer = new IntersectionObserver(handleIntersect, options);
    const boxes = document.getElementsByClassName("demo-box");
    for (let i = 0; i < boxes.length; i++) {
      observer.observe(boxes[i]);
    }
  };
  
  // 元素挂载完成的时候注册监听
  onMounted(addObserver);

  return {
    activeKey,
    tagSelected,
  };
}

export default class App extends Vue {
  tagList = ["tag0", "tag1", "tag2", "tag3", "tag4"];
  heightList = [100, 200, 400, 700, 300];
  selecter = setup(() => useSelected());
}
</script>

<style lang="scss" scoped>
.demo {
  &-box {
    width: 100%;
    padding: 10px 24px;
    background: rgb(94, 217, 226);
    display: flex;
    box-sizing: border-box;
    margin-bottom: 20px;
    font-size: 50px;
  }

  &-scroll {
    background: rgba(224, 193, 236, 0.8);
    height: 600px;
    width: 100%;
    margin-top: 20px;
    overflow-y: scroll;
  }

  &-header {
    display: flex;
    flex-direction: row;
    justify-content: center;
    font-family: "Kirang Haerang", cursive;
    font-size: 50px;
    margin-top: 20px;
  }

  &-body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-content: center;
    width: 600px;
    margin: 0 auto;
  }

  &-tags {
    display: flex;
    flex-direction: row;
    margin-top: 50px;
  }

  &-tag {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-content: center;
    padding: 8px 14px;
    background: rgb(147, 177, 231);
    width: 80px;
    margin-right: 10px;
    border-radius: 20px;
    font-size: 20px;
    font-family: "Comfortaa", cursive;

    &.active {
      background: rgb(174, 148, 245);
      font-weight: bold;
    }
  }
}
</style>
