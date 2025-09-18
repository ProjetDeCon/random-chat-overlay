<script setup lang="ts">
import tmi from "tmi.js";
import { streamers } from "./utils/streamers";
import { onMounted, ref } from "vue";

const messages = ref<{ username: string; message: string }[]>([]);

onMounted(() => {
  const client = new tmi.Client({
    channels: streamers,
  });

  client.connect();

  client.on("message", (channels, tags, message, self) => {
    if (self) return;

    messages.value.push({
      username: tags["display-name"] || tags.username || "",
      message,
    });
    if (messages.value.length > 50) {
      messages.value.shift();
    }
  });
});
</script>

<template>
  <div>
    <div v-for="message in messages">
      {{ message.username }} - {{ message.message }}
    </div>
  </div>
</template>
