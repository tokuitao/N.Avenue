<template>
  <div class="min-h-screen bg-orange-50 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="absolute inset-0 bg-gradient-to-r from-orange-400 to-orange-600 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-md mx-auto">
          <h1 class="text-3xl font-bold mb-6 text-center text-orange-600">営業効率測定アプリ</h1>
          
          <form @submit.prevent="calculateEfficiency" class="space-y-4">
            <div>
              <label for="sentEmails" class="block text-sm font-medium text-gray-700">送信メール数</label>
              <input v-model.number="sentEmails" type="number" id="sentEmails" required min="0" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-orange-500 focus:border-orange-500">
            </div>
            
            <div>
              <label for="replies" class="block text-sm font-medium text-gray-700">返信数</label>
              <input v-model.number="replies" type="number" id="replies" required min="0" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-orange-500 focus:border-orange-500">
            </div>
            
            <div>
              <label for="appointments" class="block text-sm font-medium text-gray-700">アポイントメント設定数</label>
              <input v-model.number="appointments" type="number" id="appointments" required min="0" class="mt-1 block w-full border-gray-300 rounded-md shadow-sm focus:ring-orange-500 focus:border-orange-500">
            </div>
            
            <button type="submit" class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-orange-600 hover:bg-orange-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-orange-500">
              効率を計算
            </button>
          </form>
          
          <div v-if="showResults" class="mt-8">
            <h2 class="text-2xl font-semibold mb-4 text-orange-600">結果</h2>
            <div class="grid grid-cols-2 gap-4 mb-6">
              <div class="bg-orange-100 p-4 rounded-lg text-center">
                <p class="text-sm text-orange-600 font-medium">返信率</p>
                <p class="text-3xl font-bold text-orange-700">{{ replyRate.toFixed(1) }}%</p>
              </div>
              <div class="bg-orange-100 p-4 rounded-lg text-center">
                <p class="text-sm text-orange-600 font-medium">アポイントメント獲得率</p>
                <p class="text-3xl font-bold text-orange-700">{{ appointmentRate.toFixed(1) }}%</p>
              </div>
            </div>
            
            <div class="mt-6">
              <canvas ref="chartCanvas" role="img" aria-label="返信率とアポイントメント獲得率のグラフ"></canvas>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

const sentEmails = ref(0);
const replies = ref(0);
const appointments = ref(0);
const replyRate = ref(0);
const appointmentRate = ref(0);
const showResults = ref(false);
const chartCanvas = ref(null);
let chart = null;

const calculateEfficiency = () => {
  replyRate.value = (replies.value / sentEmails.value) * 100;
  appointmentRate.value = (appointments.value / replies.value) * 100;
  showResults.value = true;
};

const updateChart = async () => {
  if (typeof window !== 'undefined') {
    const Chart = (await import('chart.js/auto')).default;
    
    if (chart) {
      chart.destroy();
    }

    const ctx = chartCanvas.value.getContext('2d');
    chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['返信率', 'アポイントメント獲得率'],
        datasets: [{
          label: '効率 (%)',
          data: [replyRate.value, appointmentRate.value],
          backgroundColor: [
            'rgba(255, 159, 64, 0.6)',
            'rgba(255, 99, 132, 0.6)'
          ],
          borderColor: [
            'rgba(255, 159, 64, 1)',
            'rgba(255, 99, 132, 1)'
          ],
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        plugins: {
          legend: {
            display: false
          },
          tooltip: {
            callbacks: {
              label: function(context) {
                return context.parsed.y.toFixed(1) + '%';
              }
            }
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            max: 100,
            ticks: {
              callback: function(value) {
                return value + '%';
              }
            }
          }
        }
      }
    });
  }
};

watch(showResults, (newValue) => {
  if (newValue) {
    updateChart();
  }
});

onMounted(() => {
  // コンポーネントがマウントされたときの処理（必要に応じて）
});
</script>
