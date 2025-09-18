<script setup lang="ts">
import tmi from "@tmi.js/chat";
import { streamers } from "./utils/streamers";
import { onMounted, ref } from "vue";

const MESSAGE_SPEED = 2;

const messages = ref<{ username: string; message: string; color: string }[]>(
  []
);
const counter = ref(0);

onMounted(() => {
  const client = new tmi.Client({
    channels: streamers,
  });

  client.connect();

  client.on("message", (e) => {
    const { user, message } = e;

    counter.value++;

    if (counter.value === MESSAGE_SPEED) {
      const emotes = message.emotes;
      let msg = message.text;

      emotes.forEach((emote) => {
        const url = `https://static-cdn.jtvnw.net/emoticons/v2/${emote.id}/default/dark/3.0`;
        emote.indices.forEach((pos) => {
          const [start, end] = pos;
          const toReplace = message.text.slice(start, end + 1);
          msg = msg.replace(
            toReplace,
            `<img class="inline h-6 w-6" src="${url}" />`
          );
        });
      });

      messages.value.push({
        username: user.display,
        message: msg,
        color: user.color,
      });
      if (messages.value.length > 100) {
        messages.value.shift();
      }

      counter.value = 0;
    }
  });
});

// onMounted(() => {
//   const client = new tmi.Client({
//     channels: streamers,
//   });

//   client.connect();

//   client.on("message", (_, tags, message, self) => {
//     if (self) return;

//     counter.value++;

//     if (counter.value === MESSAGE_SPEED) {
//       messages.value.push({
//         username: tags["display-name"] || tags.username || "",
//         message,
//         color: tags.color || "#ff0000",
//       });
//       if (messages.value.length > 100) {
//         messages.value.shift();
//       }

//       counter.value = 0;
//     }
//   });
// });
</script>

<template>
  <div
    class="h-screen w-screen flex flex-col justify-end overflow-hidden gap-1"
  >
    <div
      v-for="message in messages"
      class="font-medium text-white text-shadow-md"
    >
      <span class="font-bold mr-2" :style="{ color: message.color }">{{
        message.username
      }}</span>
      <span v-html="message.message"></span>
    </div>
  </div>
</template>
