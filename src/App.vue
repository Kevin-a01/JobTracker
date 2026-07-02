<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { supabase } from "../utils/supabase";
import { BriefcaseBusinessIcon, Plus, X } from "@lucide/vue";

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

const form = ref({
  company: "",
  title: "",
  created_at: new Date().toISOString().split("T")[0],
  status: "",
  url: "",
});

const resetForm = () => {
  form.value = {
    company: "",
    title: "",
    created_at: new Date().toISOString().split("T")[0],
    status: "",
    url: "",
  };
};

const InsertJob = async () => {
  try {
    const { data, error } = await supabase
      .from("jobs")
      .insert([
        {
          company: form.value.company,
          title: form.value.title,
          created_at: form.value.created_at,
          status: form.value.status,
          url: form.value.url,
        },
      ])
      .select();
    if (error) {
      throw new Error();
    }
    console.log("Ansökan sparad", data);

    await getJobs();

    resetForm();
    dialogRef.value?.close();
  } catch (error) {
    console.error("Kunde inte spara datan", error);
  }
};

onMounted(() => {
  getJobs();
});

const dialogRef = ref<HTMLDialogElement | null>(null);

const closeDialog = () => {
  dialogRef.value?.close();
};

const soktCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "sökt").length;
});

const intervjuCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "intervju").length;
});

const ejVidareCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "ej_vidare").length;
});

const erbjudandeCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "erbjudande").length;
});
</script>

<template>
  <header>
    <div class="p-3 gap-3 flex items-center">
      <div class="bg-black p-1.5 w-fit rounded-xl">
        <BriefcaseBusinessIcon :size="20" class="text-white" />
      </div>
      <div class="flex flex-col items-center">
        <h1 class="font-medium text-xl">Jobb Sökaren!</h1>
        <h2 class="text-sm font-mono">{{ Jobs.length }} ansökningar</h2>
      </div>
      <button
        command="show-modal"
        commandFor="job-input"
        class="bg-black text-white flex items-center gap-1 w-16 h-9 justify-center rounded-2xl ml-auto"
      >
        <Plus :size="20" /> Ny
      </button>
    </div>

    <!-- <p class="text-blue-500" v-for="job in Jobs" :key="job.id">
      {{ job.title }}
    </p> -->
  </header>
  <section class="flex justify-center items-center gap-2 overflow-hidden mt-5">
    <div
      class="border border-gray-300 py-6 px-6.5 rounded-2xl bg-gray-50 w-min-[90px] flex flex-col items-center"
    >
      <span class="w-2 h-2 rounded-full bg-blue-500 block"></span>
      <span class="font-medium mt-1">{{ soktCount }}</span>
      <span class="text-xs text-gray-500 font-medium">Sökt</span>
    </div>

    <div
      class="border border-gray-300 py-6 px-4 rounded-2xl bg-gray-50 flex flex-col items-center"
    >
      <span class="w-2 h-2 rounded-full bg-yellow-500 block"></span>
      <span class="font-medium mt-1">{{ intervjuCount }}</span>
      <span class="text-xs text-gray-500 font-medium">Intervju</span>
    </div>

    <div
      class="border border-gray-300 py-6 px-3 rounded-2xl bg-gray-50 flex flex-col items-center"
    >
      <span class="w-2 h-2 rounded-full bg-red-500 block"></span>
      <span class="font-medium mt-1">{{ ejVidareCount }}</span>
      <span class="text-xs text-gray-500 font-medium">Ej Vidare</span>
    </div>

    <div
      class="border border-gray-300 py-6 px-1.5 rounded-2xl bg-gray-50 flex flex-col items-center"
    >
      <span class="w-2 h-2 rounded-full bg-green-500 block"></span>
      <span class="font-medium mt-1">{{ erbjudandeCount }}</span>
      <span class="text-xs text-gray-500 font-medium">Erbjudande</span>
    </div>
  </section>
  <div>
    <h3 v-for="job in Jobs" :key="job.id">{{ job.title }}</h3>
  </div>

  <section class="">
    <!-- Mobile Modal -->
    <dialog
      ref="dialogRef"
      id="job-input"
      class="fixed top-auto md:mx-auto md:top-60 rounded-t-xl md:rounded-xl p-6 max-w-md w-full backdrop:backdrop-blur-xs"
    >
      <div class="flex justify-between items-center mb-6">
        <h3 class="font-medium">Lägg till ansökan</h3>
        <button
          class="text-gray-400 hover:bg-gray-200 p-1 rounded-lg transition-all duration-300 ease-in-out"
          command="close"
          commandFor="job-input"
        >
          <X :size="15" />
        </button>
      </div>
      <form @submit.prevent="InsertJob">
        <div class="grid grid-cols-2">
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-500 mb-1"
              for="company"
              >Företag *</label
            >
            <input
              class="border w-35 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
              type="text"
              v-model="form.company"
              name="company"
              id="company"
              required
              placeholder="Spotify"
            />
          </div>
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-400 mb-1"
              for="title"
              >Roll *</label
            >
            <input
              class="border w-40 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
              type="text"
              v-model="form.title"
              name="title"
              id="title"
              required
              placeholder="Frontend Developer"
            />
          </div>
        </div>

        <div class="grid grid-cols-2 mt-5">
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-500 mb-1"
              for="date"
              >Datum</label
            >
            <input
              class="border w-35 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
              type="date"
              v-model="form.created_at"
              name="created_at"
              id="date"
            />
          </div>
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-400 mb-1"
              for="status"
              >Status</label
            >
            <select
              name="status"
              v-model="form.status"
              id="status"
              class="border w-35 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
            >
              <option value="sökt">Sökt</option>
              <option value="intervju">Intervju</option>
              <option value="ej_vidare">Ej Vidare</option>
              <option value="erbjudande">Erbjudande</option>
            </select>
          </div>
        </div>

        <div class="mt-6">
          <label
            class="block text-xs uppercase font-semibold text-gray-400 mb-1"
            for="url"
            >Länk</label
          >
          <input
            class="border w-full outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
            type="text"
            v-model="form.url"
            name="url"
            id="url"
            placeholder="https://..."
          />
        </div>

        <div class="mt-7 flex justify-evenly gap-3 items-center">
          <button
            v-on:click="closeDialog"
            type="button"
            class="border text-gray-500 border-gray-200 p-1 w-40 rounded-xl"
          >
            Avbryt
          </button>

          <button type="submit" class="bg-black text-white p-1 rounded-xl w-40">
            Lägg till
          </button>
        </div>
      </form>
    </dialog>
  </section>

  <main></main>
</template>
