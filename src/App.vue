<script setup lang="ts">
import { ref } from "vue";
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

getJobs();
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
    <h1 class="text-red-500">Hello World</h1>
    <!-- <p class="text-blue-500" v-for="job in Jobs" :key="job.id">
      {{ job.title }}
    </p> -->
  </header>

  <main class="">
    <!-- Mobile Modal -->
    <dialog
      ref="dialogRef"
      id="job-input"
      class="fixed top-auto rounded-t-xl md:rounded-xl p-6 max-w-md w-full backdrop:backdrop-blur-xs"
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
      <form action="">
        <div class="grid grid-cols-2">
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-500 mb-1"
              for="title"
              >Företag *</label
            >
            <input
              class="border w-35 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
              type="text"
              name="title"
              id="title"
              required
              placeholder="Spotify"
            />
          </div>
          <div class="">
            <label
              class="block text-xs uppercase font-semibold text-gray-400 mb-1"
              for="role"
              >Roll *</label
            >
            <input
              class="border w-40 outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
              type="text"
              name="role"
              id="role"
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
              name="date"
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
            for="link"
            >Länk</label
          >
          <input
            class="border w-full outline-none border-gray-400 bg-gray-100 rounded-lg pl-1 h-7"
            type="text"
            name="link"
            id="link"
            placeholder="https://..."
          />
        </div>

        <div class="mt-7 flex justify-evenly gap-3 items-center">
          <button
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
  </main>
</template>
