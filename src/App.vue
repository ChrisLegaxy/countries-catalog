<template>
  <main>
    <div class="container mx-auto sm:px-12">
      <header>
        <div class="px-4">
          <div class="text-4xl font-bold mt-10 inline-block">
            Countries Catalog
          </div>
          <div class="relative mt-8">
            <div>
              <div
                class="flex absolute inset-y-0 left-0 items-center pl-3 pointer-events-none"
              >
                <svg
                  aria-hidden="true"
                  class="w-5 h-5"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
                  ></path>
                </svg>
              </div>
              <input
                id="default-search"
                v-model="searchCriteria"
                type="search"
                class="block p-4 pl-10 w-full rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:focus:ring-blue-500 dark:focus:border-blue-500"
                placeholder="Country Name"
              />
            </div>
          </div>
          <div class="text-red-400 mt-2.5 absolute">
            {{ error?.response?.data.message }}
          </div>
        </div>
      </header>

      <div
        class="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 xl:gap-6 mt-8"
      >
        <!-- Country Card -->
        <div
          v-for="(country, index) in countries"
          :key="index"
          class="rounded-xl"
        >
          <div class="aspect-[1.5] p-4">
            <img :src="country.flags.png" class="w-full h-full" />
          </div>
          <div class="p-4">
            <div class="font-bold truncate" :title="country.name.official">
              {{ country.name.official }}
            </div>
            <div>{{ country.cca2 }}</div>
            <div>{{ country.cca3 }}</div>
            <button
              class="py-2.5 mt-4 text-white font-medium rounded-sm text-sm w-full bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300"
            >
              View Details
            </button>
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="isLoading"
      class="fixed inset-0 w-full h-full grid place-items-center bg-sky-100/60"
    >
      <half-circle-spinner
        :animation-duration="1000"
        :size="60"
        color="#0ea5e9"
      />
    </div>
  </main>
</template>

<script lang="ts" setup>
import { ref, watch } from "vue";
import { useAxios } from "@vueuse/integrations/useAxios";
import { HalfCircleSpinner } from "epic-spinners";

import { axiosInstance } from "./libs/http";
import { Country } from "./interfaces/Country";

let searchTimeout: any;

const {
  data: countries,
  isLoading,
  execute,
  error,
} = useAxios<Country[]>("/all", axiosInstance);

const searchCriteria = ref("");

watch(searchCriteria, async () => {
  if (searchCriteria.value) {
    // avoid excessive calls
    clearTimeout(searchTimeout);
    searchTimeout = await setTimeout(() => {
      execute(`/name/${searchCriteria.value}`);
    }, 1000);
  } else {
    clearTimeout(searchTimeout);
    searchTimeout = await setTimeout(() => {
      execute();
    }, 1000);
  }
});
</script>
