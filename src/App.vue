<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { supabase } from "../utils/supabase";
import { BriefcaseBusinessIcon, Plus, Search, X } from "@lucide/vue";
import JobCard from "./components/jobCard.vue";

type jobtype = {
  id: number;
  title: string;
  company: string;
  url: string;
  status: string;
  created_at: Date;
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
  return Jobs.value.filter((job) => job.status === "Sökt").length;
});

const intervjuCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "Intervju").length;
});

const avslagCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "Avslag").length;
});

const erbjudandeCount = computed(() => {
  return Jobs.value.filter((job) => job.status === "Erbjudande").length;
});

const activeStatus = ref<string>("");

async function getJobsByStatus(valfriStatus: string) {
  try {
    let query = supabase.from("jobs").select();

    if (activeStatus.value === valfriStatus) {
      activeStatus.value = "";
    } else {
      activeStatus.value = valfriStatus;
      query = query.eq("status", valfriStatus);
    }

    const { data, error } = await query;

    if (error) throw new Error();

    if (data) {
      Jobs.value = data;
    }
  } catch (error) {
    console.error("Det gick ej att filtrera", error);
  }
}

function handleJobDeleted(id: number) {
  Jobs.value = Jobs.value.filter((job) => job.id !== id);
}

async function updateStatus(id: number, newStatus: string) {
  const { error } = await supabase
    .from("jobs")
    .update({ status: newStatus })
    .eq("id", id);

  if (error) {
    console.error("Misslyckades att uppdatera jobb", error.message);
  }

  Jobs.value = Jobs.value.map((job) =>
    job.id === id ? { ...job, status: newStatus } : job,
  );
}
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
        class="bg-black text-white flex items-center gap-1 w-16 h-9 justify-center rounded-2xl ml-auto cursor-pointer hover:text-gray-400 transform duration-200 ease-in"
      >
        <Plus :size="20" /> Ny
      </button>
    </div>

    <!-- <p class="text-blue-500" v-for="job in Jobs" :key="job.id">
      {{ job.title }}
    </p> -->
  </header>
  <section
    class="flex justify-evenly items-center gap-2 md:justify-center md:gap-7 overflow-hidden mt-3"
  >
    <button
      @click="getJobsByStatus('Sökt')"
      :class="{ 'border-gray-500': activeStatus === 'Sökt' }"
      class="border border-gray-300 py-6 px-6.5 md:w-100 rounded-2xl bg-gray-50 w-min-[90px] flex flex-col items-center"
    >
      <span class="w-2 h-2 md:w-4 md:h-4 rounded-full bg-blue-500 block"></span>
      <span class="font-medium mt-1">{{ soktCount }}</span>
      <span class="md:text-lg text-xs text-gray-500 font-medium">Sökt</span>
    </button>

    <button
      @click="getJobsByStatus('Intervju')"
      :class="{ 'border-gray-500': activeStatus === 'Intervju' }"
      class="border border-gray-300 py-6 px-4 rounded-2xl md:w-100 bg-gray-50 flex flex-col items-center"
    >
      <span
        class="w-2 h-2 md:w-4 md:h-4 rounded-full bg-yellow-500 block"
      ></span>
      <span class="font-medium mt-1 md:text-lg">{{ intervjuCount }}</span>
      <span class="md:text-lg text-xs text-gray-500 font-medium">Intervju</span>
    </button>

    <button
      @click="getJobsByStatus('Avslag')"
      :class="{ 'border-gray-500': activeStatus === 'Avslag' }"
      class="border border-gray-300 py-6 px-5 md:w-100 rounded-2xl bg-gray-50 flex flex-col items-center"
    >
      <span class="w-2 h-2 md:w-4 md:h-4 rounded-full bg-red-500 block"></span>
      <span class="font-medium mt-1 md:text-lg">{{ avslagCount }}</span>
      <span class="md:text-lg text-xs text-gray-500 font-medium">Avslag</span>
    </button>

    <button
      @click="getJobsByStatus('Erbjudande')"
      :class="{ 'border-gray-500': activeStatus === 'Erbjudande' }"
      class="border border-gray-300 md:w-100 py-6 px-1.5 rounded-2xl bg-gray-50 flex flex-col items-center"
    >
      <span
        class="w-2 h-2 md:w-4 md:h-4 rounded-full bg-green-500 block"
      ></span>
      <span class="font-medium md:text-lg mt-1">{{ erbjudandeCount }}</span>
      <span class="md:text-lg text-xs text-gray-500 font-medium"
        >Erbjudande</span
      >
    </button>
  </section>
  <div class="mt-5 mb-1 relative w-fit mx-auto">
    <Search :size="18" class="absolute left-1.5 top-2.5 text-gray-500" />
    <input
      type="text"
      name=""
      id=""
      placeholder="Sök företag eller roll..."
      class="border p-1.5 w-88 pl-7 outline-none border-gray-400 rounded-xl"
    />

    <!-- <h3 v-for="job in Jobs" :key="job.id">{{ job.title }}</h3> -->
  </div>

  <section>
    <JobCard
      :jobs="Jobs"
      @job-deleted="handleJobDeleted"
      @status-updated="updateStatus"
    />
  </section>

  <section class="">
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
              <option value="Sökt">Sökt</option>
              <option value="Intervju">Intervju</option>
              <option value="Ej Vidare">Ej Vidare</option>
              <option value="Erbjudande">Erbjudande</option>
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
