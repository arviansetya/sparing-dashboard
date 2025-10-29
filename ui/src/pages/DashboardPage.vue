<template>
  <q-page
    class="q-pa-md bg-grey-8 text-white"
    style="font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif"
  >
    <!-- Header -->
    <div class="text-center text-h3 q-mb-md">INDEKS STANDAR PENCEMAR UDARA</div>

    <!-- Row: Legend + Chart Sejajar -->
    <div class="row q-col-gutter-md q-mb-md">
      <!-- Kategori Legend (Kiri) -->
      <div class="col-12 col-sm-3">
        <q-card class="bg-grey-9 q-pa-md" style="height: 65vh">
          <q-card-section>
            <div v-for="(cat, index) in categories" :key="index" class="q-mt-xs">
              <div
                class="q-pa-xs"
                :style="{
                  backgroundColor: cat.color,
                  borderRadius: '2px',
                  height: '100px',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  textAlign: 'center',
                }"
              >
                <span
                  style="
                    font-weight: bold;
                    color: white;
                    line-height: 1.2;
                    word-wrap: break-word;
                    white-space: normal;
                    padding: 0 8px;
                    font-size: clamp(10px, 1.5vw, 30px);
                  "
                >
                  {{ cat.label }}
                </span>
              </div>
            </div>

            <div
              class="text-bold q-pt-sm"
              style="font-size: clamp(12px, 1.8vw, 28px); text-align: center; color: white"
            >
              KATEGORI
            </div>
          </q-card-section>
        </q-card>
      </div>

      <!-- Chart (Kanan) -->
      <div class="col-12 col-sm-9">
        <q-card class="bg-grey-9 q-pa-sm" style="height: 65vh">
          <q-card-section class="q-pa-none" style="height: 100%">
            <v-chart :option="chartOption" style="height: 100%; width: 100%" autoresize />
          </q-card-section>
        </q-card>
      </div>
    </div>

    <!-- Info Bawah (Parameter Kritis, ISPU, Waktu, Kualitas Udara) -->
    <q-card class="bg-grey-9 q-mt-md q-pa-none">
      <q-card-section class="q-pa-none">
        <table
          class="q-table full-width"
          style="border-collapse: collapse; width: 100%; text-align: center; color: white"
        >
          <thead>
            <tr style="border: 1px solid #777">
              <th style="border: 1px solid #777; font-size: 1.5rem; padding: 10px">
                PARAMETER KRITIS
              </th>
              <th style="border: 1px solid #777; font-size: 1.5rem; padding: 10px">ISPU</th>
              <th style="border: 1px solid #777; font-size: 1.5rem; padding: 10px">WAKTU</th>
              <th style="border: 1px solid #777; font-size: 1.5rem; padding: 10px">
                KUALITAS UDARA
              </th>
            </tr>
          </thead>
          <tbody>
            <tr style="border: 1px solid #777">
              <td
                style="border: 1px solid #777; padding: 16px; font-size: 3.8rem; font-weight: bold"
              >
                PM₂.₅
              </td>
              <td
                style="border: 1px solid #777; padding: 16px; font-size: 3.8rem; font-weight: bold"
              >
                62
              </td>
              <td style="border: 1px solid #777; padding: 16px; font-size: 2rem">
                03-September-2023 17:00:00
              </td>
              <td style="border: 1px solid #777; padding: 12px">
                <q-btn
                  unelevated
                  class="text-weight-bold"
                  style="
                    background-color: #0000ff;
                    color: white;
                    font-size: 2rem;
                    padding: 8px 16px;
                    border-radius: 4px;
                    min-width: 100px;
                  "
                >
                  SEDANG
                </q-btn>
              </td>
            </tr>
          </tbody>
        </table>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import VChart from 'vue-echarts';
import { use } from 'echarts/core';
import { GridComponent, TooltipComponent } from 'echarts/components';
import { BarChart } from 'echarts/charts';
import { CanvasRenderer } from 'echarts/renderers';
import type { CallbackDataParams } from 'echarts/types/dist/shared';

// Daftarkan modul ECharts ke vue-echarts
use([
  BarChart,
  GridComponent,
  TooltipComponent,
  CanvasRenderer, // atau SVGRenderer
]);

export default defineComponent({
  name: 'AirQualityDashboard',
  components: {
    VChart,
  },
  setup() {
    const categories = [
      { label: 'BERBAHAYA', color: '#000000' },
      { label: 'SANGAT TIDAK SEHAT', color: '#ff0000' },
      { label: 'TIDAK SEHAT', color: '#ffd700' },
      { label: 'SEDANG', color: '#0000ff' },
      { label: 'BAIK', color: '#00ff00' },
    ];

    const getColorByValue = (value: number) => {
      if (value >= 301) return '#000000'; // Berbahaya
      if (value >= 201) return '#ff0000'; // Sangat Tidak Sehat
      if (value >= 101) return '#ffd700'; // Tidak Sehat
      if (value >= 51) return '#0000ff'; // Sedang
      return '#00ff00'; // Baik
    };

    // const getColorLabel = (value: number) => {
    //   if (value >= 301) return 'Berbahaya';
    //   if (value >= 201) return 'Sangat Tidak Sehat';
    //   if (value >= 101) return 'Tidak Sehat';
    //   if (value >= 51) return 'Sedang';
    //   return 'Baik';
    // };

    // Data polutan dan nilai
    const pollutants = ['PM₁₀', 'PM₂.₅', 'SO₂', 'CO', 'O₃', 'NO₂', 'HC'];
    const values = [28, 100, 41, 17, 6, 122, 32];

    // Opsi chart
    const chartOption = ref({
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'shadow',
        },
        formatter: (params: { name: string; value: number }[]) => {
          const p = params?.[0];
          return p ? `${p.name}<br/>${p.value}` : '';
        },
      },
      grid: {
        left: '3%',
        right: '3%',
        top: '15%',
        bottom: '3%',
        containLabel: true,
      },
      xAxis: {
        type: 'category',
        data: pollutants,
        axisLine: { show: false },
        axisTick: { show: false },
        axisLabel: {
          color: '#ffffff',
          fontSize: 25,
          fontWeight: 'bold',
          padding: [10, 0, 0, 0],
        },
      },
      yAxis: {
        show: false,
        min: 0,
        max: 120,
      },
      series: [
        {
          name: 'Indeks',
          type: 'bar',
          barWidth: '50%',
          itemStyle: {
            borderRadius: [5, 5, 0, 0],
            color: (params: CallbackDataParams) => getColorByValue(params.value as number),
          },
          label: {
            show: true,
            position: 'top',
            color: '#ffffff',
            fontWeight: 'bold',
            fontSize: 20,
          },
          emphasis: {
            focus: 'series',
          },
          data: values,
        },
      ],
    });

    return {
      categories,
      chartOption,
    };
  },
});
</script>

<style scoped>
/* Pastikan card legend dan chart memiliki tinggi sama */
.q-card {
  border-radius: 8px;
}

.text-h5,
.text-h4 {
  margin: 0;
  line-height: 1.2;
}

/* Untuk mobile: pastikan tidak terlalu sempit */
@media (max-width: 600px) {
  .q-card-section {
    padding: 8px;
  }
}
</style>
