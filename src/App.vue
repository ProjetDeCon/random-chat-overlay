<script setup lang="ts">
import tmi from "tmi.js";
import { streamers } from "./utils/streamers";
import { onMounted, ref } from "vue";

const MESSAGE_SPEED = 7;

const messages = ref<{ username: string; message: string; color: string }[]>(
  []
);
const counter = ref(0);

onMounted(() => {
  const client = new tmi.Client({
    channels: streamers,
  });

  client.connect();

  client.on("message", (_, tags, message, self) => {
    if (self) return;

    counter.value++;

    if (counter.value === MESSAGE_SPEED) {
      messages.value.push({
        username: tags["display-name"] || tags.username || "",
        message,
        color: tags.color || "#ff0000",
      });
      if (messages.value.length > 100) {
        messages.value.shift();
      }

      counter.value = 0;
    }
  });
});
</script>

<template>
  <div
    class="h-screen w-screen flex flex-col justify-end overflow-hidden gap-1"
  >
    <div
      v-for="message in messages"
      class="font-medium text-white text-shadow-md"
    >
      <span class="font-bold" :style="{ color: message.color }">{{
        message.username
      }}</span>
      {{ message.message }}
    </div>
  </div>
</template>
