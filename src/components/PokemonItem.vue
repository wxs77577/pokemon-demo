<script setup lang="ts">
import { useRequest } from "vue-request";

const props = defineProps({
  url: { type: String, required: true },
});
const { data, loading } = useRequest(() => {
  return fetch(props.url).then((res) => res.json());
});
</script>
<template>
  <div
    class="card card-side bg-base-100 shadow-xl cursor-pointer hover:scale-105 transition hover:z-10 hover:shadow-2xl items-center"
    v-if="data"
  >
    <span class="loading loading-infinity" v-if="loading"></span>
    <div class="flex-shrink-0 ml-5">
      <figure class="avatar">
        <div class="h-24 rounded-full bg-base-300 p-3 mask mask-hexagon">
          <img
            :src="data.sprites.back_default"
            :alt="data.name"
            v-if="data.sprites.back_default"
          />
        </div>
      </figure>
      <div class="text-center capitalize font-bold mt-3 text-slate-600">
        {{ data.name }}
      </div>
    </div>
    <div class="card-body" v-if="data">
      <!-- <h2 class="card-title capitalize text-slate-600">{{ data.name }}</h2> -->
      <div class="text-slate-500">
        <div class="table table-xs">
          <tr class="" v-for="item in data.stats">
            <th class="capitalize">{{ item.stat.name }}</th>
            <td class="text-right text-sm text-slate-800">
              <div class=" font-mono font-bold">{{ item.base_stat }}</div>
            </td>
          </tr>
        </div>
      </div>
      <div class="card-actions justify-end">
        <div
          class="badge badge-primary"
          v-for="item in data.abilities"
          :key="item.slot"
        >
          {{ item.ability.name }}
        </div>
      </div>
    </div>
  </div>
</template>
