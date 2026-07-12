<script setup lang="ts">
import { ref } from "vue";
import { supabase } from "../../utils/supabase";
import { Trash } from "@lucide/vue";

type jobtype = {
  id: number;
  title: string;
  company: string;
  url: string;
  status: string;
  created_at: Date;
};

defineProps<{
  jobs: jobtype[];
}>();

const emit = defineEmits(["job-deleted", "status-updated"]);

const Jobs = ref<jobtype[]>([]);

const statusColors: Record<string, string> = {
  Sökt: "bg-blue-300 border-blue-300 text-blue-800",
  Intervju: "bg-yellow-300 border-yellow-300 text-yellow-800",
  Erbjudande: "bg-green-300 border-green-300 text-green-800",
  Avslag: "bg-red-300 border-red-300 text-red-800",
};

/* async function getJobs() {
  const { data, error } = await supabase.from<"jobs", jobtype>("jobs").select();

  if (data) {
    Jobs.value = data;
  }
  console.log(data);
  if (error) {
    throw new Error();
  }
}

getJobs(); */

function onStatusChange(id: number, event: Event) {
  const target = event.target as HTMLSelectElement;

  emit("status-updated", id, target.value);
}

async function deleteJob(id: number) {
  const { error } = await supabase.from("jobs").delete().eq("id", id);

  if (error) {
    console.error("Kunde inte ta bort jobb", error.message);
  }
  const response = confirm("Vill du verkligen ta bort detta jobb?");

  if (response === false) {
    return;
  }

  if (!error) {
    emit("job-deleted", id);
  }

  Jobs.value = Jobs.value.filter((job) => job.id !== id);
}

/* onMounted(() => {
  getJobs();
}); */
</script>

<template>
  <div class="p-2.5 md:grid md:grid-cols-3">
    <div
      v-for="job in jobs"
      :key="job.id"
      class="border pb-15 pt-1 md:w-150 md:mx-auto md:mt-5 flex flex-col mb-3 rounded-xl bg-gray-100 border-gray-300"
    >
      <div class="flex items-center gap-2 px-2">
        <h1 class="text-sm font-medium">
          {{ job.company }}
        </h1>

        <div
          class="border w-fit p-1 rounded-2xl bg-blue-300 border-blue-300"
          :class="statusColors[job.status]"
        >
          <h2 class="text-sm">{{ job.status }}</h2>
        </div>
        <button
          @click="deleteJob(job.id)"
          class="ml-auto text-gray-500 hover:bg-gray-200 p-1.5 rounded-full"
        >
          <Trash :size="18" />
        </button>
      </div>
      <h2 class="ml-2 text-xs">{{ job.title }}</h2>

      <h3 class="mt-2 text-xs font-mono pl-2">
        {{
          new Date(job.created_at).toLocaleDateString("sv-SE", {
            day: "numeric",
            month: "short",
            year: "numeric",
          })
        }}
      </h3>

      <div class="border border-gray-400 w-fit mt-5 ml-1 text-sm rounded-lg">
        <select
          :value="job.status"
          class="text-center outline-none appearance-none"
          name="status"
          @change="onStatusChange(job.id, $event)"
          id=""
        >
          <option value="Sökt">Sökt</option>
          <option value="Intervju">Intervju</option>
          <option value="Avslag">Avslag</option>
          <option value="Erbjudande">Erbjudande</option>
        </select>
      </div>
    </div>
  </div>
</template>
