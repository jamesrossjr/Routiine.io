<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick, watch, computed } from 'vue'
import Chart from 'chart.js/auto'
import type { ChartTypeRegistry } from 'chart.js'

const targetScore = 78
const score = ref(0)
const signals = ref([
  { type: 'positive', name: 'Email response', impact: 5 },
  { type: 'positive', name: 'PageView pattern', impact: 3 },
  { type: 'negative', name: 'Decreased engagement', impact: -4 },
  { type: 'positive', name: 'Form submission', impact: 6 },
  { type: 'negative', name: 'Session time dropped', impact: -2 }
])
// Keep a fixed maximum number of signals to prevent layout shifts
const MAX_SIGNALS_DISPLAYED = 5

const activityTrend = computed(() => {
  return signals.value.filter(s => s.type === 'positive').length > 
         signals.value.filter(s => s.type === 'negative').length ? 12 : -8
})
const conversionTrend = computed(() => {
  return signals.value.reduce((sum, s) => sum + s.impact, 0) > 0 ? 4 : -3
})
const signalCount = computed(() => signals.value.length)

// Get only the most recent signals up to MAX_SIGNALS_DISPLAYED
const visibleSignals = computed(() => {
  return signals.value.slice(-MAX_SIGNALS_DISPLAYED)
})

let scorecardChart: Chart<keyof ChartTypeRegistry, unknown[], unknown> | null = null

// Calculate the dynamic score based on all signals
const calculateScore = () => {
  const signalImpact = signals.value.reduce((sum, s) => sum + s.impact, 0)
  // Base score of 50 plus signal impact multiplied by 2
  return Math.min(100, Math.max(0, 50 + signalImpact * 2))
}

// Update chart and animate score changes
const updateChart = () => {
  const newTarget = calculateScore()
  
  // Animate to new target
  const animateToTarget = setInterval(() => {
    if (score.value < newTarget) {
      score.value++
    } else if (score.value > newTarget) {
      score.value--
    } else {
      clearInterval(animateToTarget)
    }
  }, 20)
  
  // Update chart colors based on score
  if (scorecardChart && scorecardChart.data && scorecardChart.data.datasets) {
    const scoreColor = score.value > 70 ? '#0d9488' : 
                       score.value > 50 ? '#0ea5e9' :
                       score.value > 30 ? '#f59e0b' : '#ef4444'
                       
    if (scorecardChart.data.datasets[0]) {
      scorecardChart.data.datasets[0].backgroundColor = [scoreColor, '#1e293b']
    }
    scorecardChart.update('none')
  }
}

// Simulate backend signal processing with automated changes
const simulateBackendProcessing = () => {
  // Random chance to add or remove signals
  const action = Math.random()
  
  if (action > 0.8 && signals.value.length > 2) {
    // Remove a signal (20% chance if we have more than 2 signals)
    const indexToRemove = Math.floor(Math.random() * signals.value.length)
    signals.value.splice(indexToRemove, 1)
  } else if (action > 0.6 || signals.value.length < 2) {
    // Add a new signal (40% chance or if we have less than 2 signals)
    const posSignals = [
      { type: 'positive', name: 'New page view', impact: 3 },
      { type: 'positive', name: 'Form initiated', impact: 4 },
      { type: 'positive', name: 'Email opened', impact: 2 },
      { type: 'positive', name: 'Video watched', impact: 5 }
    ]
    
    const negSignals = [
      { type: 'negative', name: 'Abandoned cart', impact: -4 },
      { type: 'negative', name: 'Bounce detected', impact: -3 },
      { type: 'negative', name: 'Session timeout', impact: -2 },
      { type: 'negative', name: 'Form abandoned', impact: -5 }
    ]
    
    // 60% chance for positive signal
    const newSignal =
    Math.random() > 0.4
        ? posSignals[Math.floor(Math.random() * posSignals.length)]!
        : negSignals[Math.floor(Math.random() * negSignals.length)]!
      
    signals.value.push(newSignal)
  }
  
  // Update the chart with new data
  updateChart()
}

const renderChart = () => {
  const ctx = document.getElementById('scorecardChart') as HTMLCanvasElement | null
  if (!ctx) return

  // First destroy any existing chart instance
  if (scorecardChart) {
    scorecardChart.destroy()
  }
  
  // Clear any previous chart instance from Chart.js internal registry
  Chart.getChart(ctx)?.destroy()
  
  // Create new chart
  scorecardChart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: ['Score', 'Remaining'],
      datasets: [
        {
          data: [score.value, 100 - score.value],
          backgroundColor: ['#0d9488', '#1e293b'],
          borderWidth: 0,
          circumference: 180,
          rotation: 270
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '80%',
      plugins: {
        legend: { display: false },
        tooltip: { enabled: false }
      },
      animation: {
        duration: 800,
        easing: 'easeOutQuart'
      }
    }
  })
}

onMounted(async () => {
  await nextTick()
  
  // Set initial score target based on current signals
  score.value = 0
  
  // Render initial chart
  renderChart()
  
  // Start with initial calculation and animation
  updateChart()
  
  // Set up automatic signal processing simulation to show automation
  const automationInterval = setInterval(() => {
    simulateBackendProcessing()
  }, 3000) // Update every 3 seconds
  
  // Store interval for cleanup
  onBeforeUnmount(() => {
    clearInterval(automationInterval)
  })
})

// Add reactivity for score updates
watch(score, () => {
  if (scorecardChart && scorecardChart.data && scorecardChart.data.datasets && scorecardChart.data.datasets[0]) {
    scorecardChart.data.datasets[0].data = [score.value, 100 - score.value]
    scorecardChart.update('none') // Use 'none' for smoother updates
  }
})

onBeforeUnmount(() => {
  if (scorecardChart) {
    scorecardChart.destroy()
    scorecardChart = null
  }
  
  // Clear any running animations or intervals
  const allIntervals = window.setInterval(() => {}, 9999);
  for (let i = 1; i <= allIntervals; i++) {
    window.clearInterval(i);
  }
})
</script>

<template>
  <UContainer class="py-12 flex justify-center items-center border-0">
    <UPageCard variant="ghost" class="shadow-lg shadow-none text-center">
      <div class="w-[320px] sm:w-[360px] md:w-[400px] aspect-[9/18] rounded-[2.5rem] bg-gradient-to-b from-slate-900 via-slate-800 to-slate-900 text-white p-6 shadow-xl border-0 flex flex-col justify-between">
        <!-- Header -->
        <h2 class="text-center text-lg font-bold tracking-tight text-white">Momentum Scorecard</h2>

        <!-- Score Circle -->
        <div class="relative h-40 mt-2">
          <canvas id="scorecardChart" class="w-full h-full" />
          <div class="absolute inset-0 flex items-center justify-center text-5xl font-bold text-white">
            {{ score }}
          </div>
        </div>

        <!-- Stats -->
        <div class="text-sm mt-4 space-y-3">
          <div class="flex justify-between items-center">
            <p :class="[activityTrend >= 0 ? 'text-teal-400' : 'text-red-400']">
              Activities: <span class="font-semibold">{{ activityTrend >= 0 ? '+' : '' }}{{ activityTrend }}%</span>
            </p>
            <div class="w-2 h-2 rounded-full" :class="[activityTrend >= 0 ? 'bg-teal-400' : 'bg-red-400']"></div>
          </div>
          
          <div class="flex justify-between items-center">
            <p :class="[conversionTrend >= 0 ? 'text-teal-400' : 'text-red-400']">
              Conversion: <span class="font-semibold">{{ conversionTrend >= 0 ? '+' : '' }}{{ conversionTrend }}%</span>
            </p>
            <div class="w-2 h-2 rounded-full" :class="[conversionTrend >= 0 ? 'bg-teal-400' : 'bg-red-400']"></div>
          </div>
          
          <!-- Signal List - Fixed height container -->
          <div class="mt-4">
            <p class="text-sky-300 mb-1">Signals detected: <span class="font-semibold">{{ signalCount }}</span></p>
            <!-- Fixed height container to prevent layout shifts -->
            <div class="h-32 overflow-y-auto pr-2 signals-container">
              <div class="space-y-1">
                <div v-for="(signal, index) in visibleSignals" :key="index" 
                    class="flex justify-between items-center text-xs py-1 px-2 rounded"
                    :class="[signal.type === 'positive' ? 'bg-emerald-900/40' : 'bg-red-900/40']">
                  <span>{{ signal.name }}</span>
                  <span :class="[signal.type === 'positive' ? 'text-emerald-400' : 'text-red-400']">
                    {{ signal.type === 'positive' ? '+' : '' }}{{ signal.impact }}
                  </span>
                </div>
                <!-- Placeholder empty items to maintain consistent height when fewer signals -->
                <div v-for="i in Math.max(0, MAX_SIGNALS_DISPLAYED - visibleSignals.length)" :key="`empty-${i}`"
                    class="flex justify-between items-center text-xs py-1 px-2 rounded opacity-0 pointer-events-none">
                  <span>Placeholder</span>
                  <span>+0</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </UPageCard>
  </UContainer>
</template>

<style scoped>
canvas {
  max-width: 100%;
  max-height: 100%;
}

/* Hide scrollbar but maintain scroll functionality */
.signals-container {
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE and Edge */
}

.signals-container::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}
</style>