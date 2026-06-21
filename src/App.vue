<script setup lang="ts">
import { ref } from "vue";
import { supabase } from "../utils/supabase";
import { BriefcaseBusinessIcon, type LucideProps } from "@lucide/vue";
import { BriefcaseIcon } from "@lucide/vue";

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
    <div class="p-3 gap-3 flex items-center">
      <div class="bg-black p-1.5 w-fit rounded-xl">
        <BriefcaseBusinessIcon :size="20" class="text-white" />
      </div>
      <div class="flex items-center flex-col">
        <h1 class="text-sm font-medium">Jobb Sökaren!</h1>
        <h2 class="text-sm font-mono">{{ Jobs.length }} ansökningar</h2>
      </div>
    </div>

    <h1 class="text-red-500">Hello World</h1>
    <!-- <p class="text-blue-500" v-for="job in Jobs" :key="job.id">
      {{ job.title }}
    </p> -->
  </header>
</template>
