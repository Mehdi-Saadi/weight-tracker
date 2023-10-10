<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightChartEl = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref(60.0);
const currentWeight = computed(() => {
    return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});
const addWeight = () => {
    weights.value.push({
        weight: weightInput.value,
        date: new Date().getTime()
    })
}

watch(weights, newWeights => {
    const ws = [...newWeights];

    if (weightChart.value) {
        weightChart.value.data.labels = ws
            .sort((a, b) => a.date - b.date)
            .map(w => new Date(w.date).toLocaleDateString())
            .slice(-7);

        weightChart.value.data.datasets[0].data = ws
            .sort((a, b) => a.date - b.date)
            .map(w => w.weight)
            .slice(-7);

        weightChart.value.update();

        return
    }

    nextTick(() => {
        weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
            type: 'line',
            data: {
                labels: ws
                    .sort((a, b) => a.date - b.date)
                    .map(w => new Date(w.date).toLocaleDateString()),
                datasets: [
                    {
                        label: 'Weight',
                        data: ws
                            .sort((a, b) => a.date - b.date)
                            .map(w => w.weight),
                        backgroundColor: 'rgba(255, 105, 180, 0.2)',
                        borderColor: 'rgb(255, 105, 180)',
                        borderWidth: 1,
                        fill: true
                    }
                ]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false
            }
        });
    });
}, { deep: true });
</script>

<template>
    <div class="max-w-lg mx-auto pt-10 flex flex-col items-center justify-center space-y-5">
        <h1 class="text-xl font-bold">Weight Tracker</h1>

        <div class="w-36 h-36 rounded-full border-4 border-pink-400 flex flex-col items-center justify-center space-y-1 bg-white">
            <span class="font-bold text-2xl">{{ currentWeight.weight }}</span>
            <span class="text-xs text-gray-500 italic">Current weight (kg)</span>
        </div>

        <form @submit.prevent="addWeight" class="w-full flex">
            <input type="number" step="0.1" v-model="weightInput" class="w-full rounded-l-lg px-4 py-2 border outline-pink-400 focus:outline border-pink-400">
            <button type="submit" class="text-gray-50 bg-pink-400 border border-pink-400 rounded-r-lg px-6 text-sm hover:text-pink-400 hover:bg-white transition duration-300">Add</button>
        </form>

        <div v-if="weights && weights.length > 0" class="w-full">

            <h2 class="text-sm text-gray-500">Last 7 days</h2>

            <div class="mt-2">
                <canvas ref="weightChartEl"></canvas>
            </div>

            <div class="mt-10 text-sm">
                <h2 class="text-gray-500">Weight History</h2>
                <ul class="mt-2">
                    <li v-for="weight in weights" class="flex justify-between">
                        <span class="font-bold">{{ weight.weight }}kg</span>
                        <span class="text-gray-500">{{ new Date(weight.date).toLocaleDateString() }}</span>
                    </li>
                </ul>
            </div>

        </div>
    </div>
</template>
