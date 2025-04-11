<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from 'vue'
import Chart from 'chart.js/auto'
import type { ChartTypeRegistry, Point, BubbleDataPoint } from 'chart.js'

// Signal type definition
interface Signal {
  type: 'positive' | 'negative'
  name: string
  impact: number
  timestamp: number
}

// Constants
const MAX_SIGNALS_DISPLAYED = 5
const ANIMATION_SPEED = 20 // ms between animation frames
const SIMULATION_INTERVAL = 3000 // ms between signal updates
const BATCH_INTERVAL = 80 // ms between chart updates

// State management
const score = ref(0)
const targetScore = ref(0)
const signals = ref<Signal[]>([
  { type: 'positive', name: 'Email response', impact: 5, timestamp: Date.now() - 5000 },
  { type: 'positive', name: 'PageView pattern', impact: 3, timestamp: Date.now() - 4000 },
  { type: 'negative', name: 'Decreased engagement', impact: -4, timestamp: Date.now() - 3000 },
  { type: 'positive', name: 'Form submission', impact: 6, timestamp: Date.now() - 2000 },
  { type: 'negative', name: 'Session time dropped', impact: -2, timestamp: Date.now() - 1000 }
])

// Chart reference with proper type definition
let chartInstance: Chart<keyof ChartTypeRegistry, (number | [number, number] | Point | BubbleDataPoint | null)[], unknown> | null = null

// Interval references for cleanup
let animationTimeoutId: number | null = null
let automationIntervalId: number | null = null
let chartUpdateTimeoutId: number | null = null

// Computed properties
const activityTrend = computed(() => {
  const positiveCount = signals.value.filter(s => s.type === 'positive').length
  const negativeCount = signals.value.filter(s => s.type === 'negative').length
  return positiveCount > negativeCount ? 12 : -8
})

const conversionTrend = computed(() => {
  const netImpact = signals.value.reduce((sum, s) => sum + s.impact, 0)
  return netImpact > 0 ? 4 : -3
})

const signalCount = computed(() => signals.value.length)

// Get only the most recent signals up to MAX_SIGNALS_DISPLAYED
const visibleSignals = computed(() => {
  return [...signals.value]
    .sort((a, b) => b.timestamp - a.timestamp)
    .slice(0, MAX_SIGNALS_DISPLAYED)
})

// Generate placeholders to maintain consistent layout
const placeholderCount = computed(() =>
  Math.max(0, MAX_SIGNALS_DISPLAYED - visibleSignals.value.length)
)

// Calculate score color based on value
const scoreColor = computed(() => {
  if (score.value > 70) return '#0d9488' // teal-600
  if (score.value > 50) return '#0ea5e9' // sky-500
  if (score.value > 30) return '#f59e0b' // amber-500
  return '#ef4444' // red-500
})

// Calculate the dynamic score based on all signals
const calculateScore = () => {
  const signalImpact = signals.value.reduce((sum, s) => sum + s.impact, 0)
  // Base score of 50 plus signal impact multiplied by 2
  return Math.min(100, Math.max(0, 50 + signalImpact * 2))
}

// Create or update chart with error handling and proper cleanup
const renderChart = () => {
  const ctx = document.getElementById('scorecardChart') as HTMLCanvasElement | null
  if (!ctx) {
    console.error('Canvas element not found')
    return
  }

  try {
    // Clean up existing chart instance if it exists
    if (chartInstance) {
      chartInstance.destroy()
      chartInstance = null
    }

    // Create new chart with configuration
    chartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['Score', 'Remaining'],
        datasets: [{
          data: [score.value, 100 - score.value],
          backgroundColor: [scoreColor.value, '#1e293b'],
          borderWidth: 0,
          circumference: 180,
          rotation: 270
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        cutout: '80%',
        plugins: {
          legend: { display: false },
          tooltip: { enabled: false }
        },
        layout: {
          padding: {
            bottom: 30 // Add padding for the score text
          }
        }
      }
    })
  } catch (error) {
    console.error('Error creating chart:', error)
  }
}

// Update chart with new data without re-rendering
const updateChartDisplay = () => {
  if (!chartInstance) {
    renderChart()
    return
  }

  try {
    // Use type-safe access to the datasets with null checks
    if (chartInstance.data && chartInstance.data.datasets && chartInstance.data.datasets[0]) {
      chartInstance.data.datasets[0].data = [score.value, 100 - score.value]
      chartInstance.data.datasets[0].backgroundColor = [scoreColor.value, '#1e293b']

      // Update with proper animation for smooth transitions
      chartInstance.update('active')
    }
  } catch (error) {
    console.error('Error updating chart:', error)
    // If there's an error updating, try to re-render the chart
    renderChart()
  }
}

// Improved animation with setTimeout
const animateToTargetScore = () => {
  // Clear any existing animation
  if (animationTimeoutId !== null) {
    window.clearTimeout(animationTimeoutId)
    animationTimeoutId = null
  }

  // Calculate the difference to determine animation speed
  const difference = Math.abs(targetScore.value - score.value)
  const speedAdjustment = difference > 10 ? 2 : 1 // Speed up larger changes
  const animationDelay = Math.max(10, ANIMATION_SPEED / speedAdjustment)

  let lastUpdateTime = 0

  // Define the animation function
  const animate = () => {
    const currentTime = Date.now()
    let needsUpdate = false

    if (score.value < targetScore.value) {
      score.value++
      needsUpdate = true
    } else if (score.value > targetScore.value) {
      score.value--
      needsUpdate = true
    } else {
      // We've reached the target, no need to continue
      return
    }

    // Only update chart at certain intervals to avoid too many redraws
    if (needsUpdate && (currentTime - lastUpdateTime > BATCH_INTERVAL)) {
      updateChartDisplay()
      lastUpdateTime = currentTime
    }

    // Continue animation with setTimeout for better control
    animationTimeoutId = window.setTimeout(animate, animationDelay)
  }

  // Start the animation
  animate()
}

// Set target score and start animation
const setTargetScore = (newTarget: number) => {
  targetScore.value = Math.min(100, Math.max(0, newTarget))
  animateToTargetScore()
}

// Signal generation with more variety
const getRandomSignal = (positive: boolean): Signal => {
  const posSignals = [
    { type: 'positive' as const, name: 'New page view', impact: 3 },
    { type: 'positive' as const, name: 'Form initiated', impact: 4 },
    { type: 'positive' as const, name: 'Email opened', impact: 2 },
    { type: 'positive' as const, name: 'Video watched', impact: 5 },
    { type: 'positive' as const, name: 'Live chat started', impact: 3 },
    { type: 'positive' as const, name: 'Product clicked', impact: 2 },
    { type: 'positive' as const, name: 'Download initiated', impact: 4 }
  ]

  const negSignals = [
    { type: 'negative' as const, name: 'Abandoned cart', impact: -4 },
    { type: 'negative' as const, name: 'Bounce detected', impact: -3 },
    { type: 'negative' as const, name: 'Session timeout', impact: -2 },
    { type: 'negative' as const, name: 'Form abandoned', impact: -5 },
    { type: 'negative' as const, name: 'Error occurred', impact: -3 },
    { type: 'negative' as const, name: 'Page load slow', impact: -2 },
    { type: 'negative' as const, name: 'Exit from funnel', impact: -4 }
  ]

  const sourceArray = positive ? posSignals : negSignals
  const randomIndex = Math.floor(Math.random() * sourceArray.length)
  const signal = sourceArray[randomIndex] || {
    type: positive ? 'positive' as const : 'negative' as const,
    name: positive ? 'Default positive signal' : 'Default negative signal',
    impact: positive ? 3 : -3
  }

  return {
    type: signal.type,
    name: signal.name,
    impact: signal.impact,
    timestamp: Date.now()
  }
}

// Simulate backend signal processing with debounced updates - improved debouncing
const simulateBackendProcessing = () => {
  // Random chance to add or remove signals
  const action = Math.random()

  if (action > 0.8 && signals.value.length > 3) {
    // Remove a signal (20% chance if we have more than 3 signals)
    const indexToRemove = Math.floor(Math.random() * signals.value.length)
    signals.value.splice(indexToRemove, 1)
  } else if (action > 0.6 || signals.value.length < 3) {
    // Add a new signal (40% chance or if we have less than 3 signals)
    const isPositive = Math.random() > 0.4 // 60% chance for positive signals
    const newSignal = getRandomSignal(isPositive)
    signals.value.push(newSignal)
  }

  // Debounce score updates to prevent excessive chart redraws
  if (chartUpdateTimeoutId !== null) {
    window.clearTimeout(chartUpdateTimeoutId)
  }

  chartUpdateTimeoutId = window.setTimeout(() => {
    // Update target score based on new signals
    setTargetScore(calculateScore())
    chartUpdateTimeoutId = null
  }, 50) // Small delay to batch multiple signal changes
}

// Improved cleanup function with null checks
const cleanup = () => {
  if (chartInstance) {
    chartInstance.destroy()
    chartInstance = null
  }

  if (animationTimeoutId !== null) {
    window.clearTimeout(animationTimeoutId)
    animationTimeoutId = null
  }

  if (automationIntervalId !== null) {
    window.clearInterval(automationIntervalId)
    automationIntervalId = null
  }

  if (chartUpdateTimeoutId !== null) {
    window.clearTimeout(chartUpdateTimeoutId)
    chartUpdateTimeoutId = null
  }
}

// Manual signal addition - useful for external triggers
const _addSignal = (type: 'positive' | 'negative', name: string, impact: number) => {
  signals.value.push({
    type,
    name,
    impact,
    timestamp: Date.now()
  })
  setTargetScore(calculateScore())
}

// Setup mounted with improved initialization
onMounted(async () => {
  await nextTick()

  // Set initial values
  score.value = 0

  // Initialize chart after DOM is fully rendered
  renderChart()

  // Set initial score target based on current signals and update directly
  const initialScore = calculateScore()
  targetScore.value = initialScore
  score.value = initialScore

  // Update the chart once with initial values
  updateChartDisplay()

  // Set up automatic signal processing simulation
  automationIntervalId = window.setInterval(() => {
    simulateBackendProcessing()
  }, SIMULATION_INTERVAL)
})

// Clean up resources on unmount
onBeforeUnmount(cleanup)
</script>

<template>
  <UContainer class="py-12 flex justify-center items-center">
    <UCard
      :ui="{
        base: 'shadow-none text-center',
        rounded: 'rounded-lg',
        shadow: '',
        padding: 'p-4',
        header: {
          base: '',
          padding: 'px-4 py-1.5 sm:px-6 sm:py-2',
          rounded: '-mx-4 -mt-4 sm:-mx-6 sm:-mt-6',
          shadow: 'shadow-sm',
          border: 'border-b',
          // eslint-disable-next-line @typescript-eslint/no-unused-vars
          paddingIcon: 'pe-2',
          // eslint-disable-next-line @typescript-eslint/no-unused-vars
          sizeIcon: 'h-4 w-4'
        },
        footer: {
          base: '',
          padding: 'px-4 py-3 sm:px-6 sm:py-4',
          rounded: '-mx-4 -mb-4 sm:-mx-6 sm:-mb-6',
          shadow: 'shadow-sm',
          border: 'border-t'
        }
      }"
      class="bg-transparent border-0"
    >
      <div
        class="w-[320px] sm:w-[360px] md:w-[400px] aspect-[9/18] rounded-[2.5rem] bg-gradient-to-b from-slate-900 via-slate-800 to-slate-900 text-white p-6 flex flex-col justify-between"
        aria-label="Momentum Scorecard"
        role="region"
      >
        <!-- Header -->
        <UHeading
          tag="h2"
          size="lg"
          class="text-center tracking-tight text-white"
        >
          Momentum Scorecard
        </UHeading>

        <!-- Score Circle -->
        <div
          class="relative h-40 mt-2"
          aria-label="Score Meter"
        >
          <div class="scorecard-chart-container w-full h-full">
            <canvas
              id="scorecardChart"
              role="img"
              aria-label="Score Visualization"
            />
          </div>
          <div
            class="absolute inset-0 flex flex-col items-center justify-end pb-2"
          >
            <span
              class="text-6xl font-bold"
              :style="`color: ${scoreColor}`"
              aria-live="polite"
            >
              {{ score }}
            </span>
            <span class="text-xs text-slate-400 mt-1">momentum score</span>
          </div>
        </div>

        <!-- Stats -->
        <div class="text-sm mt-6 space-y-3">
          <div class="flex justify-between items-center">
            <p
              :class="[activityTrend >= 0 ? 'text-teal-400' : 'text-red-400']"
              class="flex-grow"
            >
              Activities:
              <span class="font-semibold min-w-[32px] inline-block text-right">
                {{ activityTrend >= 0 ? '+' : '' }}{{ activityTrend }}%
              </span>
            </p>
            <div class="flex items-center flex-shrink-0 ml-2">
              <span
                class="text-xs mr-1"
                :class="[activityTrend >= 0 ? 'text-teal-400' : 'text-red-400']"
              >
                {{ activityTrend >= 0 ? '↑' : '↓' }}
              </span>
              <UBadge
                size="xs"
                :color="activityTrend >= 0 ? 'teal' : 'red'"
                variant="subtle"
                class="w-2 h-2 p-0 rounded-full"
              />
            </div>
          </div>

          <div class="flex justify-between items-center">
            <p
              :class="[conversionTrend >= 0 ? 'text-teal-400' : 'text-red-400']"
              class="flex-grow"
            >
              Conversion:
              <span class="font-semibold min-w-[32px] inline-block text-right">
                {{ conversionTrend >= 0 ? '+' : '' }}{{ conversionTrend }}%
              </span>
            </p>
            <div class="flex items-center flex-shrink-0 ml-2">
              <span
                class="text-xs mr-1"
                :class="[conversionTrend >= 0 ? 'text-teal-400' : 'text-red-400']"
              >
                {{ conversionTrend >= 0 ? '↑' : '↓' }}
              </span>
              <UBadge
                size="xs"
                :color="conversionTrend >= 0 ? 'teal' : 'red'"
                variant="subtle"
                class="w-2 h-2 p-0 rounded-full"
              />
            </div>
          </div>

          <!-- Signal List -->
          <div class="mt-5">
            <div class="flex justify-between items-center mb-2">
              <p class="text-sky-300">
                Signals detected
              </p>
              <UBadge
                size="xs"
                color="sky"
                variant="subtle"
                class="font-semibold ml-1"
                style="margin-right: 9px;"
              >
                {{ signalCount }}
              </UBadge>
            </div>

            <div class="h-32 overflow-y-auto pr-2 signals-container scrollbars-hidden">
              <div class="space-y-1">
                <div
                  v-for="(signal, index) in visibleSignals"
                  :key="`signal-${index}`"
                  class="flex justify-between items-center text-xs py-1.5 px-2.5 rounded transition-all duration-200"
                  :class="[
                    signal.type === 'positive'
                      ? 'bg-emerald-900/40 border-l-2 border-emerald-500'
                      : 'bg-red-900/40 border-l-2 border-red-500'
                  ]"
                  role="listitem"
                >
                  <span class="flex-grow">{{ signal.name }}</span>
                  <span
                    class="font-medium flex-shrink-0 min-w-[32px] text-right"
                    :class="[
                      signal.type === 'positive'
                        ? 'text-emerald-400'
                        : 'text-red-400'
                    ]"
                  >
                    {{ signal.type === 'positive' ? '+' : '' }}{{ signal.impact }}
                  </span>
                </div>

                <!-- Placeholders to maintain consistent layout -->
                <div
                  v-for="i in placeholderCount"
                  :key="`empty-${i}`"
                  class="flex justify-between items-center text-xs py-1.5 px-2.5 rounded opacity-0 pointer-events-none h-6"
                  aria-hidden="true"
                >
                  <span class="flex-grow">Placeholder</span>
                  <span class="flex-shrink-0 min-w-[32px] text-right">+0</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </UCard>
  </UContainer>
</template>

<style scoped>
canvas {
  max-width: 100%;
  max-height: 100%;
}

.scorecard-chart-container {
  position: relative;
  width: 100%;
  height: 100%;
}

/* Hide scrollbar but maintain scroll functionality */
.scrollbars-hidden {
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE and Edge */
}

.scrollbars-hidden::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}
</style>
