<script setup lang="ts">
import InputSwitch from "primevue/inputswitch";
import MultiSelect from "primevue/multiselect";
import Panel from "@/components/misc/Panel.vue";
import ProgressSpinner from "primevue/progressspinner";
import ReportItem from "@/components/report/ReportItem.vue";
import ViewPanel from "@/components/misc/ViewPanel.vue";

import { computed, onMounted, reactive } from "vue";
import { useMapStore } from "@/stores/map_store";
import { useReportStore } from "@/stores/report_store";

import { TaiwanBounds } from "@/helpers/constant";
import Global from "@/global";

const FilterIntensityOptions = [
  { label: "1", value: 1 },
  { label: "2", value: 2 },
  { label: "3", value: 3 },
  { label: "4", value: 4 },
  { label: "5⁻", value: 5 },
  { label: "5⁺", value: 6 },
  { label: "6⁻", value: 7 },
  { label: "6⁺", value: 8 },
  { label: "7", value: 9 },
];

const FilterMagnitudeOptions = [
  { label: "1", value: 1 },
  { label: "2", value: 2 },
  { label: "3", value: 3 },
  { label: "4", value: 4 },
  { label: "5", value: 5 },
  { label: "6", value: 6 },
  { label: "7", value: 7 },
  { label: "8", value: 8 },
];

const mapStore = useMapStore();
const reportStore = useReportStore();
const filters = reactive<{
  numbered: boolean;
  intensity: number[];
  magnitude: number[];
}>({
  numbered: false,
  intensity: [],
  magnitude: [],
});

const filteredList = computed(() =>
  reportStore.list
    .filter((v) => !filters.numbered || (filters.numbered && v.no % 1000))
    .filter(
      (v) => !filters.intensity.length || filters.intensity.includes(v.int)
    )
    .filter(
      (v) => !filters.magnitude.length || filters.magnitude.includes(~~v.mag)
    )
);

onMounted(() => {
  Global.api.getReportList().then((v) => {
    reportStore.$patch({ partial: new Map(v.map((v) => [v.id, v])) });
  });

  window.setTimeout(() => mapStore.map?.fitBounds(TaiwanBounds));
});
</script>

<template>
  <ViewPanel class="report-wrapper" title="地震報告">
    <Panel class="filter-panel" title="篩選器" titleIcon="filter_alt" collapsed>
      <div class="filter-list">
        <label class="filter-item" for="filter-numbered">
          <div class="filter-label">只顯示編號報告</div>
          <InputSwitch v-model="filters.numbered" input-id="filter-numbered" />
        </label>
        <div class="filter-item">
          <div class="filter-label">最大震度</div>
          <MultiSelect
            v-model="filters.intensity"
            :options="FilterIntensityOptions"
            option-label="label"
            option-value="value"
            :style="{ width: '120px' }"
          />
        </div>
        <div class="filter-item">
          <div class="filter-label">規模</div>
          <MultiSelect
            v-model="filters.magnitude"
            :options="FilterMagnitudeOptions"
            option-label="label"
            option-value="value"
            :style="{ width: '120px' }"
          />
        </div>
      </div>
    </Panel>
    <div class="report-list">
      <ReportItem
        v-if="reportStore.list.length"
        v-for="(report, i) in filteredList"
        :key="`report-item-${report.id}`"
        :report="report"
        :markerOpacity="0.25 + 0.75 * (1 - i / filteredList.length)"
        :markerZIndex="filteredList.length - i"
      />
      <ProgressSpinner
        v-else
        style="height: 32px; width: 32px"
        strokeWidth="4"
      />
    </div>
  </ViewPanel>
</template>

<style scoped>
.report-wrapper {
  width: 280px;
  overflow-y: auto;
}

.report-list {
  display: flex;
  flex-direction: column;
}

.filter-panel {
  margin: 8px;
}

.filter-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.filter-item {
  display: flex;
  align-items: center;
}

.filter-label {
  flex: 1;
  font-size: 14px;
}
</style>
