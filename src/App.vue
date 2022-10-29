<template>
  <main>
    <div class="container mx-auto sm:px-12">
      <header>
        <div class="px-4">
          <div class="text-4xl font-bold mt-10 inline-block">
            Countries Catalog
          </div>
          <!-- Search -->
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
              <div class="text-red-400 mt-2.5 absolute">
                {{ error?.response?.data.message }}
              </div>
            </div>
          </div>
          <!-- Sorting -->
          <div class="mt-12">Sort by name</div>
          <div class="flex">
            <div>
              <button
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.ASC,
                }"
                class="px-5 py-2.5 mt-4 font-medium rounded-sm text-sm border-2 border-sky-700 hover:bg-sky-800 hover:text-white focus:ring-2 focus:ring-blue-300"
                @click="orderCountryNameBy(OrderBy.ASC)"
              >
                Ascending
              </button>
              <button
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.DESC,
                }"
                class="ml-2.5 px-5 py-2.5 mt-4 font-medium rounded-sm text-sm border-2 border-sky-700 hover:bg-sky-800 hover:text-white focus:ring-2 focus:ring-blue-300"
                @click="orderCountryNameBy(OrderBy.DESC)"
              >
                Descending
              </button>
              <button
                class="ml-2.5 px-5 py-2.5 mt-4 text-white font-medium rounded-sm text-sm bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300"
                @click="reset"
              >
                Reset
              </button>
            </div>

            <div class="ml-auto">
              <button
                :disabled="isFirstPage"
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.ASC,
                  'cursor-not-allowed bg-gray-500 hover:bg-gray-500':
                    isFirstPage,
                }"
                class="px-5 py-2.5 mt-4 text-white font-medium rounded-sm text-sm border-2 bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300"
                @click="prev"
              >
                Previous
              </button>
              <button
                :disabled="isLastPage"
                :class="{
                  'bg-sky-700 text-white': orderNameBy === OrderBy.DESC,
                  'cursor-not-allowed bg-gray-500 hover:bg-gray-500':
                    isLastPage,
                }"
                class="ml-2.5 px-5 py-2.5 mt-4 text-white font-medium rounded-sm text-sm border-2 bg-sky-700 hover:bg-sky-800 focus:ring-2 focus:ring-blue-300"
                @click="next"
              >
                Next
              </button>
            </div>
          </div>
        </div>
      </header>

      <div
        class="grid grid-cols-2 md:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 xl:gap-6 mt-4 mb-12"
      >
        <!-- Country Card -->
        <div
          v-for="(country, index) in displayCountries"
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
      v-if="isLoading || loading"
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
import { ref, watch, watchEffect, computed } from "vue";
import { useAxios } from "@vueuse/integrations/useAxios";
import { useOffsetPagination } from "@vueuse/core";
import { HalfCircleSpinner } from "epic-spinners";
import _ from "lodash";

import { axiosInstance } from "./libs/http";
import { Country } from "./interfaces/Country";

const loading = ref(false);

const {
  data: countries,
  isLoading,
  execute,
  error,
} = useAxios<Country[]>("/all", axiosInstance);

/**
 * * Search
 */
let searchTimeout: any;
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

/**
 * * Order By
 */
enum OrderBy {
  ASC = "ASC",
  DESC = "DESC",
}

const orderNameBy = ref();

const reset = () => {
  orderNameBy.value = "";
  execute();
};

const orderCountryNameBy = (order: OrderBy) => {
  orderNameBy.value = order;
  loading.value = true;

  if (order === OrderBy.ASC) {
    countries.value = _.orderBy(countries.value, ["name.official"], "asc");
  } else if (order === OrderBy.DESC) {
    countries.value = _.orderBy(countries.value, ["name.official"], "desc");
  }

  // just additional UX
  setTimeout(() => {
    loading.value = false;
  }, 1000);
};

/**
 * * Pagination
 */
const displayCountries = ref();
const pageSize = ref(25);
const pageCount = ref();

const fetchData = ({
  currentPage,
  currentPageSize,
}: {
  currentPage: number;
  currentPageSize: number;
}) => {
  const start = (currentPage - 1) * currentPageSize;
  const end = start + currentPageSize;
  displayCountries.value = countries.value?.slice(start, end);
};

const { currentPage, currentPageSize, isFirstPage, prev, next } =
  useOffsetPagination({
    total: countries.value?.length,
    page: 1,
    pageSize,
    onPageChange: fetchData,
  });

/**
 * * Custom Last Page Logic
 *
 * Due to problem with useOffsetPagination package has bugs that doesn't
 * support async/await to get the actual pageCount
 */
watchEffect(() => {
  // initial load
  displayCountries.value = countries.value?.slice(0, pageSize.value);

  if (countries.value) {
    pageCount.value = Math.max(
      1,
      Math.ceil(countries.value.length / currentPageSize.value)
    );
  }
});

const isLastPage = computed(() => currentPage.value === pageCount.value);
</script>
