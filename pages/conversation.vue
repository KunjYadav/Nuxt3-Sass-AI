<template>
  <div>
    <!-- Heading -->
    <Heading
      title="Conversation"
      description="Our most advanced conversation model."
      icon="lucide:message-square"
      iconColor="text-violet-500"
      bgColor="bg-violet-500/10"
    />
    <div class="px-4 lg:px-8">
      <div>
        <form
          @submit.prevent="submitForm"
          class="rounded-lg border w-full p-4 px-3 md:px-6 focus-within:shadow-sm grid grid-cols-12 gap-2"
        >
          <div class="col-span-12 lg:col-span-10 flex flex-col justify-center">
            <div class="m-0 p-0">
              <input
                type="text"
                v-model="prompt"
                placeholder="How do I calculate the radius of a circle?"
                class="border-0 outline-none focus-visible:ring-0 focus-visible:ring-transparent w-full"
              />
            </div>
          </div>
          <Button
            class="col-span-12 lg:col-span-2"
            type="submit"
            :disabled="!prompt || isLoading"
          >
            Generate
          </Button>
        </form>
      </div>
      <div class="space-y-4 mt-4">
        <div
          v-if="isLoading"
          class="p-8 rounded-lg w-full flex items-center justify-center bg-muted"
        >
          <Loader />
        </div>
        <Empty v-if="!messages.length && !isLoading" label="No conversation started." />

        <div class="flex flex-col-reverse gap-y-4">
          <div
            v-for="message in messages"
            :key="message.content"
            :class="`p-8 w-full flex items-center gap-x-3 rounded-lg ${
              message.role === 'user'
                ? 'bg-white border border-black/10'
                : 'bg-slate-200'
            }`"
          >
            <UserAvatar v-if="message.role === 'user'" />
            <BotAvatar v-else />
            <p class="text-sm">{{ message.content }}</p>
          </div>
        </div>
      </div>
    </div>
    <!-- Loader -->
    <!-- Empty -->
    <!-- BotAvatar -->
    <!-- UserAvatar -->
  </div>
</template>

<script setup lang="ts">
import type { ChatCompletionRequestMessage } from "~/types";

import { useProModal } from "~/store/useProModal";
const store = useProModal();

const prompt = ref("");
const isLoading = ref(false);
const messages = ref<ChatCompletionRequestMessage[]>([]);

const submitForm = async () => {
  isLoading.value = true;
  const userMessage: ChatCompletionRequestMessage = {
    role: "user",
    content: prompt.value,
  };
  const newMessages = [...messages.value, userMessage];
  const { data, error } = await useFetch("/api/conversation", {
    method: "POST",
    body: {
      messages: newMessages,
    },
  });

  if (data.value) {
    messages.value = [
      ...messages.value,
      userMessage,
      {
        role: "assistant",
        content: data.value.content as string,
      },
    ];
    await refreshNuxtData("userData");
  }

  if (error.value) {
    console.log("[Conversation_Error]", error.value.statusMessage);
    if (error.value.statusCode === 403) {
      store.onOpen();
    }
  }
  isLoading.value = false;
};
</script>

<style scoped></style>
