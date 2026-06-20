<script setup lang="ts">
import { ref } from "vue";
import { supabase } from "../utils/supabase";

type jobtype = {
  id: number;
  title: string;
  company: string;
  url: string;
  status: string;
};

const Jobs = ref<jobtype[]>([]);

async function getJobs() {
  const { data, error } = await supabase.from<"jobs", jobtype>("jobs").select();
  if (data) {
    Jobs.value = data;
  }
  console.log(data);
  console.log("Error", error);
}

getJobs();
</script>

<template>
  <header>
    <h1 class="text-red-500">Hello World</h1>
    <p class="text-blue-500" v-for="job in Jobs" :key="job.id">
      {{ job.title }}
    </p>
    <p v-for="job in Jobs" :key="job.id">{{ job.company }}</p>
  </header>
</template>
