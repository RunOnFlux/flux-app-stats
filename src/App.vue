<template>
  <v-app>
    <v-main>
      <v-container fluid>
        <div class="d-flex justify-space-between align-center mb-6">
          <div></div>
          <div class="text-center flex-grow-1">
            <h1>Flux App Analyzer</h1>
            <p>Analyze app registrations on the Flux network from any time period</p>
          </div>
          <v-btn
            icon
            @click="toggleTheme"
            variant="text"
          >
            <v-icon>{{ isDark ? 'mdi-white-balance-sunny' : 'mdi-weather-night' }}</v-icon>
          </v-btn>
        </div>

        <v-row class="mb-6 justify-center">
          <v-col cols="auto">
            <v-text-field
              v-model="startDate"
              label="Start Date & Time (UTC)"
              type="datetime-local"
              variant="outlined"
              density="compact"
              hide-details
              @change="validateDates"
            ></v-text-field>
          </v-col>

          <v-col cols="auto">
            <v-text-field
              v-model="endDate"
              label="End Date & Time (UTC) - Optional"
              type="datetime-local"
              variant="outlined"
              density="compact"
              hide-details
              @change="validateDates"
            ></v-text-field>
          </v-col>

          <v-col cols="auto">
            <v-btn
              color="primary"
              size="large"
              :loading="loading"
              :disabled="!startDate"
              @click="analyze"
            >
              Analyze
            </v-btn>
          </v-col>
        </v-row>

        <v-card v-if="error" class="mb-4 error-card" color="error" variant="tonal" elevation="8" rounded="xl">
          <v-card-text class="d-flex flex-column align-center pa-10">
            <v-icon
              size="100"
              color="error"
              class="mb-6 glow-icon"
            >
              mdi-alert-circle-outline
            </v-icon>
            <div class="text-h5 font-weight-bold text-center mb-2">Oops! Something went wrong</div>
            <div class="text-body-1 text-center text-medium-emphasis">{{ error }}</div>
          </v-card-text>
        </v-card>

    <template v-if="results && !loading">
      <!-- Summary Cards -->
      <v-row class="mb-6">
        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="primary" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-apps</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.summary.newApps }}</div>
                  <div class="text-subtitle-2">New Apps</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ results.rates.appsPerDay }} apps/day</div>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="secondary" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-server-network</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.summary.totalInstances }}</div>
                  <div class="text-subtitle-2">Total Instances</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ results.rates.instancesPerDay }} instances/day</div>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="success" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-package-variant-closed</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.summary.multiComponentApps }}</div>
                  <div class="text-subtitle-2">Multi-Component</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ ((results.summary.multiComponentApps / results.summary.newApps) * 100).toFixed(1) }}% of total</div>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="info" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-calendar-clock</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.timeSinceStart.days }}</div>
                  <div class="text-subtitle-2">Days</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ results.timeSinceStart.hours }} hours total</div>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="warning" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-cube-outline</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.heightRange.difference.toLocaleString() }}</div>
                  <div class="text-subtitle-2">Blocks</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ results.heightRange.min }} → {{ results.heightRange.max }}</div>
              <div class="text-caption">Avg: {{ results.heightRange.avgBlockTime }}s/block</div>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" sm="6" md="4" lg="2">
          <v-card variant="tonal" color="purple" class="h-100">
            <v-card-text class="d-flex flex-column" style="min-height: 150px;">
              <div class="d-flex align-center mb-2">
                <v-icon size="40" class="mr-3">mdi-rocket-launch</v-icon>
                <div>
                  <div class="text-h4 font-weight-bold">{{ results.summary.orbitApps }}</div>
                  <div class="text-subtitle-2">Orbit Apps</div>
                </div>
              </div>
              <v-divider class="my-2"></v-divider>
              <div class="text-caption">{{ ((results.summary.orbitApps / results.summary.newApps) * 100).toFixed(1) }}% of total</div>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- Charts -->
      <v-row class="mb-6">
        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>Apps by Owner (Top 10)</v-card-title>
            <v-card-text style="height: 300px;">
              <Bar :data="ownerChartData" :options="chartOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>Instances Distribution (Top 10)</v-card-title>
            <v-card-text style="height: 300px;">
              <Doughnut :data="instancesChartData" :options="doughnutOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>Apps Over Time</v-card-title>
            <v-card-text style="height: 300px;">
              <Line :data="timelineChartData" :options="timelineOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>Cumulative Apps Growth</v-card-title>
            <v-card-text style="height: 300px;">
              <Line :data="cumulativeChartData" :options="timelineOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>Instance Distribution</v-card-title>
            <v-card-text style="height: 300px;">
              <Bar :data="instanceDistributionChartData" :options="chartOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="6">
          <v-card>
            <v-card-title>App Categories</v-card-title>
            <v-card-text style="height: 300px;">
              <Pie :data="categoryChartData" :options="doughnutOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col v-if="orbitChartData" cols="12" md="6">
          <v-card>
            <v-card-title>Orbit vs Non-Orbit Apps</v-card-title>
            <v-card-text style="height: 300px;">
              <Doughnut :data="orbitChartData" :options="doughnutOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col v-if="multiComponentChartData" cols="12" md="6">
          <v-card>
            <v-card-title>Multi-Component Apps</v-card-title>
            <v-card-text style="height: 300px;">
              <Pie :data="multiComponentChartData" :options="doughnutOptions" />
            </v-card-text>
          </v-card>
        </v-col>

        <v-col v-if="repoVisibilityChartData" cols="12" md="6">
          <v-card>
            <v-card-title>Repository Visibility</v-card-title>
            <v-card-text style="height: 300px;">
              <Doughnut :data="repoVisibilityChartData" :options="doughnutOptions" />
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- High Instance Apps List -->
      <div class="mb-6">
        <div class="d-flex justify-space-between align-center mb-4">
          <div class="d-flex align-center">
            <h2 class="text-h5">High Instance Apps</h2>
            <v-chip variant="tonal" color="primary" prepend-icon="mdi-fire" class="ml-3">{{ highInstanceApps.length }}</v-chip>
          </div>
          <v-text-field
            v-model.number="instanceThreshold"
            type="number"
            label="Minimum Instances"
            prepend-inner-icon="mdi-filter"
            variant="outlined"
            density="compact"
            hide-details
            style="max-width: 200px;"
            min="1"
          ></v-text-field>
        </div>
        <v-alert v-if="highInstanceApps.length === 0" type="info" variant="tonal" icon="mdi-information-outline">
          No apps found with more than {{ instanceThreshold }} instances. Try lowering the threshold.
        </v-alert>
        <v-row v-else>
          <v-col v-for="(app, index) in highInstanceApps" :key="app.name" cols="12" md="6" lg="4">
            <v-card>
              <v-card-title class="d-flex align-center">
                <v-chip variant="tonal" color="purple" size="small" class="mr-2">#{{ index + 1 }}</v-chip>
                <span class="text-truncate">{{ app.name }}</span>
              </v-card-title>
              <v-card-text>
                <div class="d-flex flex-wrap mb-3">
                  <v-chip variant="tonal" color="primary" prepend-icon="mdi-monitor-multiple" size="small" class="mr-2 mb-2">
                    {{ app.instances }} instance{{ app.instances > 1 ? 's' : '' }}
                  </v-chip>
                  <v-chip variant="tonal" color="secondary" prepend-icon="mdi-package-variant" size="small" class="mr-2 mb-2">
                    {{ app.components }} component{{ app.components > 1 ? 's' : '' }}
                  </v-chip>
                  <v-chip v-if="app.isOrbitApp" variant="tonal" color="cyan" prepend-icon="mdi-rocket-launch" size="small" class="mr-2 mb-2">
                    Orbit
                  </v-chip>
                  <v-chip
                    variant="tonal"
                    :color="app.repotags && app.repotags.length > 0 ? 'success' : 'warning'"
                    :prepend-icon="app.repotags && app.repotags.length > 0 ? 'mdi-lock-open-variant' : 'mdi-lock'"
                    size="small"
                    class="mr-2 mb-2"
                  >
                    {{ app.repotags && app.repotags.length > 0 ? 'Public' : 'Private' }}
                  </v-chip>
                </div>
                <div class="d-flex align-center" style="gap: 4px;">
                  <v-icon size="small">mdi-account</v-icon>
                  <span class="text-caption text-truncate" style="font-family: monospace;">{{ app.owner }}</span>
                </div>
                <div v-if="app.repotags && app.repotags.length > 0" class="mt-3">
                  <div class="text-caption font-weight-bold mb-2">
                    <v-icon size="default">mdi-docker</v-icon> Docker Images:
                  </div>
                  <v-chip
                    v-for="tag in app.repotags"
                    :key="tag"
                    size="small"
                    class="mr-1 mb-1"
                    variant="outlined"
                    :href="getRegistryUrl(tag)"
                    target="_blank"
                    link
                  >
                    {{ tag }}
                  </v-chip>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </div>

      <!-- Top Owners -->
      <div class="mb-6">
        <div class="d-flex align-center mb-4">
          <h2 class="text-h5">Top App Owners</h2>
          <v-chip variant="tonal" color="secondary" prepend-icon="mdi-account-star" class="ml-3">10</v-chip>
        </div>
        <v-row>
          <v-col v-for="(owner, index) in results.topOwners.slice(0, 10)" :key="owner.owner" cols="12" md="6" lg="4">
            <v-card>
              <v-card-title class="d-flex align-center">
                <v-chip variant="tonal" color="purple" size="small" class="mr-2">#{{ index + 1 }}</v-chip>
                <span class="text-truncate font-weight-mono text-caption">{{ owner.owner }}</span>
              </v-card-title>
              <v-card-text>
                <div class="d-flex flex-wrap">
                  <v-chip variant="tonal" color="primary" prepend-icon="mdi-application" size="small" class="mr-2 mb-2">
                    {{ owner.apps }} app{{ owner.apps > 1 ? 's' : '' }}
                  </v-chip>
                  <v-chip variant="tonal" color="secondary" prepend-icon="mdi-laptop" size="small" class="mr-2 mb-2">
                    {{ owner.instances }} instance{{ owner.instances > 1 ? 's' : '' }}
                  </v-chip>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </div>

      <!-- Orbit Apps List -->
      <div v-if="results.orbitApps.length > 0" class="mb-6">
        <div class="d-flex align-center mb-4">
          <h2 class="text-h5">Orbit Apps</h2>
          <v-chip variant="tonal" color="cyan" prepend-icon="mdi-rocket-launch" class="ml-3">{{ results.orbitApps.length }}</v-chip>
        </div>
        <RecycleScroller
          class="app-scroller"
          :items="results.orbitApps"
          :item-size="160"
          key-field="name"
          v-slot="{ item, index }"
        >
          <div class="app-item" style="min-height: 160px; padding: 12px; margin-bottom: 8px; border: 1px solid rgba(128,128,128,0.2); border-radius: 4px;">
            <div class="app-header">
              <div class="app-name">
                {{ index + 1 }}. {{ item.name }}
                <v-chip v-if="item.isMultiComponent" variant="tonal" color="secondary" size="x-small" class="ml-2">{{ item.components }} components</v-chip>
                <v-chip
                  variant="tonal"
                  :color="item.repotags && item.repotags.length > 0 ? 'success' : 'warning'"
                  :prepend-icon="item.repotags && item.repotags.length > 0 ? 'mdi-lock-open-variant' : 'mdi-lock'"
                  size="x-small"
                  class="ml-2"
                >
                  {{ item.repotags && item.repotags.length > 0 ? 'Public' : 'Private' }}
                </v-chip>
              </div>
            </div>
            <div class="app-details">
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-calendar</v-icon>Date</span>
                <span>{{ formatDate(item.date) }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-cube-outline</v-icon>Height</span>
                <span>{{ item.height.toLocaleString() }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-monitor-multiple</v-icon>Instances</span>
                <span>{{ item.instances }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-account</v-icon>Owner</span>
                <span style="font-family: monospace; font-size: 0.8rem;">{{ item.owner }}</span>
              </div>
              <div v-if="item.repotags && item.repotags.length > 0" class="app-detail" style="grid-column: 1 / -1;">
                <span class="app-detail-label"><v-icon size="default" class="mr-1">mdi-docker</v-icon>Repotags</span>
                <div class="d-flex flex-wrap" style="gap: 4px;">
                  <v-chip
                    v-for="tag in item.repotags"
                    :key="tag"
                    size="small"
                    variant="outlined"
                    :href="getRegistryUrl(tag)"
                    target="_blank"
                    link
                  >
                    {{ tag }}
                  </v-chip>
                </div>
              </div>
            </div>
          </div>
        </RecycleScroller>
      </div>

      <!-- All Apps List -->
      <div class="mb-6">
        <div class="d-flex align-center mb-4">
          <h2 class="text-h5">All New Apps</h2>
          <v-chip variant="tonal" color="success" prepend-icon="mdi-apps" class="ml-3">{{ results.newApps.length }}</v-chip>
        </div>
        <RecycleScroller
          class="app-scroller"
          :items="results.newApps"
          :item-size="160"
          key-field="name"
          v-slot="{ item, index }"
        >
          <div class="app-item" style="min-height: 160px; padding: 12px; margin-bottom: 8px; border: 1px solid rgba(128,128,128,0.2); border-radius: 4px;">
            <div class="app-header">
              <div class="app-name">
                {{ index + 1 }}. {{ item.name }}
                <v-chip v-if="item.isMultiComponent" variant="tonal" color="secondary" size="x-small" class="ml-2">{{ item.components }} components</v-chip>
                <v-chip v-if="item.isOrbitApp" variant="tonal" color="cyan" size="x-small" class="ml-2">Orbit</v-chip>
                <v-chip
                  variant="tonal"
                  :color="item.repotags && item.repotags.length > 0 ? 'success' : 'warning'"
                  :prepend-icon="item.repotags && item.repotags.length > 0 ? 'mdi-lock-open-variant' : 'mdi-lock'"
                  size="x-small"
                  class="ml-2"
                >
                  {{ item.repotags && item.repotags.length > 0 ? 'Public' : 'Private' }}
                </v-chip>
              </div>
            </div>
            <div class="app-details">
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-calendar</v-icon>Date</span>
                <span>{{ formatDate(item.date) }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-cube-outline</v-icon>Height</span>
                <span>{{ item.height.toLocaleString() }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-monitor-multiple</v-icon>Instances</span>
                <span>{{ item.instances }}</span>
              </div>
              <div class="app-detail">
                <span class="app-detail-label"><v-icon size="small" class="mr-1">mdi-account</v-icon>Owner</span>
                <span style="font-family: monospace; font-size: 0.8rem;">{{ item.owner }}</span>
              </div>
              <div v-if="item.repotags && item.repotags.length > 0" class="app-detail" style="grid-column: 1 / -1;">
                <span class="app-detail-label"><v-icon size="default" class="mr-1">mdi-docker</v-icon>Repotags</span>
                <div class="d-flex flex-wrap" style="gap: 4px;">
                  <v-chip
                    v-for="tag in item.repotags"
                    :key="tag"
                    size="small"
                    variant="outlined"
                    :href="getRegistryUrl(tag)"
                    target="_blank"
                    link
                  >
                    {{ tag }}
                  </v-chip>
                </div>
              </div>
            </div>
          </div>
        </RecycleScroller>
      </div>
    </template>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useTheme } from 'vuetify'
import { Bar, Doughnut, Line, Pie } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'
import ChartDataLabels from 'chartjs-plugin-datalabels'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  ArcElement,
  Title,
  Tooltip,
  Legend,
  ChartDataLabels
)

const API_BASE = 'https://api.runonflux.io'

// Theme management
const theme = useTheme()
const isDark = computed(() => theme.global.current.value.dark)

// Load saved theme preference on mount
const savedTheme = localStorage.getItem('theme')
if (savedTheme) {
  theme.global.name.value = savedTheme
}

const toggleTheme = () => {
  const newTheme = theme.global.current.value.dark ? 'light' : 'dark'
  theme.global.name.value = newTheme
  localStorage.setItem('theme', newTheme)
}

// Fork constants - PON fork where chain became 4x faster
const FORK_BLOCK_HEIGHT = 2020000
const BLOCK_TIME_PRE_FORK = 120 // seconds (2 minutes)
const BLOCK_TIME_POST_FORK = 30 // seconds (0.5 minutes)

// Reactive state
const startDate = ref('')
const endDate = ref('')
const loading = ref(false)
const loadingMessage = ref('')
const error = ref('')
const results = ref(null)
const instanceThreshold = ref(10)

// Set default date (AMA date)
const now = new Date()
startDate.value = '2026-01-28T17:00'

// Validate dates
const validateDates = () => {
  if (startDate.value && endDate.value) {
    const start = new Date(startDate.value)
    const end = new Date(endDate.value)
    if (end <= start) {
      error.value = 'End date must be after start date'
      return false
    }
  }
  error.value = ''
  return true
}

// Format date helper
const formatDate = (date) => {
  return new Date(date).toLocaleString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    timeZone: 'UTC'
  })
}

// Get registry URL from repo tag
const getRegistryUrl = (repotag) => {
  // ghcr.io/owner/repo:tag -> https://github.com/owner/repo/pkgs/container/repo
  if (repotag.startsWith('ghcr.io/')) {
    const parts = repotag.replace('ghcr.io/', '').split(':')[0].split('/')
    if (parts.length >= 2) {
      const owner = parts[0]
      const repo = parts[parts.length - 1]
      return `https://github.com/${owner}/${parts.slice(1).join('/')}/pkgs/container/${repo}`
    }
  }

  // gcr.io/project/image:tag -> https://gcr.io/project/image
  if (repotag.startsWith('gcr.io/')) {
    return `https://${repotag.split(':')[0]}`
  }

  // quay.io/owner/repo:tag -> https://quay.io/repository/owner/repo
  if (repotag.startsWith('quay.io/')) {
    const path = repotag.replace('quay.io/', '').split(':')[0]
    return `https://quay.io/repository/${path}`
  }

  // registry.gitlab.com/owner/repo:tag -> https://gitlab.com/owner/repo/container_registry
  if (repotag.startsWith('registry.gitlab.com/')) {
    const path = repotag.replace('registry.gitlab.com/', '').split(':')[0]
    return `https://gitlab.com/${path}/container_registry`
  }

  // Docker Hub: either "owner/repo:tag" or "repo:tag"
  const dockerHubPath = repotag.split(':')[0]
  if (dockerHubPath.includes('/')) {
    return `https://hub.docker.com/r/${dockerHubPath}`
  } else {
    // Official image (no namespace)
    return `https://hub.docker.com/_/${dockerHubPath}`
  }
}

// Calculate fork-aware average block time
const calculateAverageBlockTime = (minHeight, maxHeight) => {
  if (minHeight === maxHeight) return 0

  const totalBlocks = maxHeight - minHeight

  // Case 1: Both heights are pre-fork
  if (maxHeight <= FORK_BLOCK_HEIGHT) {
    return BLOCK_TIME_PRE_FORK
  }

  // Case 2: Both heights are post-fork
  if (minHeight >= FORK_BLOCK_HEIGHT) {
    return BLOCK_TIME_POST_FORK
  }

  // Case 3: Height range spans the fork
  const preForkBlocks = FORK_BLOCK_HEIGHT - minHeight
  const postForkBlocks = maxHeight - FORK_BLOCK_HEIGHT

  const preForkTime = preForkBlocks * BLOCK_TIME_PRE_FORK
  const postForkTime = postForkBlocks * BLOCK_TIME_POST_FORK
  const totalTime = preForkTime + postForkTime

  return totalTime / totalBlocks
}

// Categorize apps
const categorizeApp = (appName) => {
  const name = appName.toLowerCase()
  if (name.includes('wordpress')) return 'WordPress'
  if (name.includes('conduit')) return 'Conduit'
  if (name.includes('presearch')) return 'Presearch'
  if (name.includes('flux') || name.includes('terraria') || name.includes('valheim') ||
      name.includes('rust') || name.includes('zomboid') || name.includes('enshrouded') ||
      name.includes('soulmask') || name.includes('forest')) return 'Game Servers'
  if (name.includes('node')) return 'Blockchain Nodes'
  return 'Other'
}

// Check if app uses Orbit images
const usesOrbitImage = (compose) => {
  if (!Array.isArray(compose)) return false

  return compose.some(component => {
    const repotag = component.repotag || ''
    return repotag.toLowerCase().includes('orbit')
  })
}

// Fetch and analyze data
const analyze = async () => {
  if (!validateDates()) return

  loading.value = true
  error.value = ''
  results.value = null

  try {
    loadingMessage.value = 'Fetching permanent messages from Flux API...'

    const response = await fetch(`${API_BASE}/apps/permanentmessages`)
    const data = await response.json()

    if (data.status !== 'success') {
      throw new Error('Failed to fetch data from API')
    }

    const messages = data.data

    loadingMessage.value = 'Filtering fluxappregister messages...'

    // Filter for fluxappregister messages
    const registerMessages = messages.filter(msg => msg.type === 'fluxappregister')

    // Convert dates to timestamps
    const startTimestamp = new Date(startDate.value + ':00Z').getTime()
    const endTimestamp = endDate.value
      ? new Date(endDate.value + ':00Z').getTime()
      : Date.now()

    const MAX_REASONABLE_TIMESTAMP = Date.now() + (365 * 24 * 60 * 60 * 1000)

    // Filter by timestamp range
    const newRegistrations = registerMessages.filter(msg =>
      msg.timestamp >= startTimestamp &&
      msg.timestamp <= endTimestamp &&
      msg.timestamp < MAX_REASONABLE_TIMESTAMP
    )

    loadingMessage.value = 'Processing app data...'

    // Process registrations
    const uniqueApps = new Map()
    const allRegistrations = []
    const ownerStats = new Map()

    newRegistrations.forEach(msg => {
      const specs = msg.appSpecifications
      if (!specs || !specs.name) return

      const appName = specs.name
      const owner = specs.owner
      const height = msg.height
      const timestamp = msg.timestamp
      const instances = specs.instances || 0
      const compose = specs.compose
      const isMultiComponent = Array.isArray(compose) && compose.length > 1
      const components = Array.isArray(compose) ? compose.length : 1
      const isOrbitApp = usesOrbitImage(compose)

      // Extract repotags
      const repotags = Array.isArray(compose)
        ? compose.map(c => c.repotag || '').filter(r => r)
        : []

      allRegistrations.push({
        name: appName,
        owner,
        height,
        timestamp,
        date: new Date(timestamp),
        instances,
        isMultiComponent,
        components,
        category: categorizeApp(appName),
        isOrbitApp,
        repotags,
      })

      if (!uniqueApps.has(appName)) {
        uniqueApps.set(appName, {
          name: appName,
          owner,
          height,
          timestamp,
          date: new Date(timestamp),
          instances,
          isMultiComponent,
          components,
          category: categorizeApp(appName),
          isOrbitApp,
          repotags,
        })

        if (!ownerStats.has(owner)) {
          ownerStats.set(owner, { apps: 0, instances: 0 })
        }
        const stats = ownerStats.get(owner)
        stats.apps += 1
        stats.instances += instances
      }
    })

    allRegistrations.sort((a, b) => a.timestamp - b.timestamp)

    // Calculate statistics
    const totalNewApps = uniqueApps.size
    const totalRegistrations = allRegistrations.length
    const totalInstances = Array.from(uniqueApps.values()).reduce((sum, app) => sum + app.instances, 0)
    const multiComponentApps = Array.from(uniqueApps.values()).filter(app => app.isMultiComponent)
    const orbitApps = Array.from(uniqueApps.values()).filter(app => app.isOrbitApp)

    const timeDiff = endTimestamp - startTimestamp
    const daysSinceStart = Math.floor(timeDiff / (1000 * 60 * 60 * 24))
    const hoursSinceStart = Math.floor(timeDiff / (1000 * 60 * 60))

    const minHeight = allRegistrations.length > 0 ? Math.min(...allRegistrations.map(r => r.height)) : 0
    const maxHeight = allRegistrations.length > 0 ? Math.max(...allRegistrations.map(r => r.height)) : 0

    // Calculate fork-aware average block time
    const avgBlockTime = calculateAverageBlockTime(minHeight, maxHeight)

    const topOwners = Array.from(ownerStats.entries())
      .sort((a, b) => b[1].apps - a[1].apps)
      .map(([owner, stats]) => ({ owner, apps: stats.apps, instances: stats.instances }))

    // Find app with highest instances
    const highestInstanceApp = Array.from(uniqueApps.values()).reduce((max, app) =>
      app.instances > max.instances ? app : max
    , { name: 'N/A', instances: 0 })

    results.value = {
      summary: {
        newApps: totalNewApps,
        totalRegistrations,
        totalInstances,
        multiComponentApps: multiComponentApps.length,
        orbitApps: orbitApps.length,
      },
      highestInstanceApp: {
        name: highestInstanceApp.name,
        instances: highestInstanceApp.instances,
        owner: highestInstanceApp.owner,
      },
      timeSinceStart: {
        days: daysSinceStart,
        hours: hoursSinceStart,
      },
      heightRange: {
        min: minHeight,
        max: maxHeight,
        difference: maxHeight - minHeight,
        avgBlockTime: avgBlockTime.toFixed(1),
      },
      rates: {
        appsPerDay: daysSinceStart > 0 ? (totalNewApps / daysSinceStart).toFixed(2) : '0',
        instancesPerDay: daysSinceStart > 0 ? (totalInstances / daysSinceStart).toFixed(2) : '0',
      },
      topOwners,
      newApps: Array.from(uniqueApps.values()).sort((a, b) => a.timestamp - b.timestamp),
      allRegistrations,
      orbitApps,
    }

  } catch (err) {
    error.value = err.message || 'Failed to analyze data'
    console.error(err)
  } finally {
    loading.value = false
    loadingMessage.value = ''
  }
}

// Chart data
const ownerChartData = computed(() => {
  if (!results.value) return null

  const top10 = results.value.topOwners.slice(0, 10)

  return {
    labels: top10.map(o => o.owner.substring(0, 10) + '...'),
    datasets: [{
      label: 'Apps',
      data: top10.map(o => o.apps),
      backgroundColor: [
        '#667eea', '#764ba2', '#f093fb', '#4facfe',
        '#43e97b', '#fa709a', '#fee140', '#30cfd0',
        '#a8edea', '#fed6e3'
      ],
    }]
  }
})

const instancesChartData = computed(() => {
  if (!results.value) return null

  const top10 = results.value.topOwners.slice(0, 10)

  return {
    labels: top10.map(o => o.owner.substring(0, 12) + '...'),
    datasets: [{
      data: top10.map(o => o.instances),
      backgroundColor: [
        '#667eea', '#764ba2', '#f093fb', '#4facfe',
        '#43e97b', '#fa709a', '#fee140', '#30cfd0',
        '#a8edea', '#fed6e3'
      ],
      borderWidth: 0
    }]
  }
})

const timelineChartData = computed(() => {
  if (!results.value || results.value.allRegistrations.length === 0) return null

  // Group by day
  const appsByDay = new Map()

  results.value.allRegistrations.forEach(app => {
    const day = app.date.toISOString().split('T')[0]
    appsByDay.set(day, (appsByDay.get(day) || 0) + 1)
  })

  const sortedDays = Array.from(appsByDay.keys()).sort()

  return {
    labels: sortedDays.map(d => new Date(d).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })),
    datasets: [{
      label: 'New Apps',
      data: sortedDays.map(d => appsByDay.get(d)),
      borderColor: '#667eea',
      backgroundColor: 'rgba(102, 126, 234, 0.1)',
      tension: 0.4,
      fill: true,
    }]
  }
})

const cumulativeChartData = computed(() => {
  if (!results.value || results.value.allRegistrations.length === 0) return null

  // Group by day
  const appsByDay = new Map()

  results.value.allRegistrations.forEach(app => {
    const day = app.date.toISOString().split('T')[0]
    appsByDay.set(day, (appsByDay.get(day) || 0) + 1)
  })

  const sortedDays = Array.from(appsByDay.keys()).sort()

  // Calculate cumulative totals
  let cumulative = 0
  const cumulativeData = sortedDays.map(d => {
    cumulative += appsByDay.get(d)
    return cumulative
  })

  return {
    labels: sortedDays.map(d => new Date(d).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })),
    datasets: [{
      label: 'Total Apps',
      data: cumulativeData,
      borderColor: '#43e97b',
      backgroundColor: 'rgba(67, 233, 123, 0.1)',
      tension: 0.4,
      fill: true,
    }]
  }
})

const instanceDistributionChartData = computed(() => {
  if (!results.value) return null

  // Generate highly distinct colors using golden angle for maximum separation
  const generateColor = (index) => {
    // Use golden angle (~137.5°) to maximize color difference between adjacent bars
    const goldenAngle = 137.5
    const hue = (index * goldenAngle) % 360

    // Avoid red spectrum (0-30 and 330-360)
    let adjustedHue = hue
    if (hue < 30) adjustedHue = hue + 60
    else if (hue > 330) adjustedHue = hue - 60

    // High saturation and moderate-high lightness for vibrant, bright colors
    const saturation = 85 + (index % 4) * 5 // Very saturated (85-100%)
    const lightness = 55 + ((index % 5) - 2) * 5 // Brighter (50-65%)

    return `hsl(${adjustedHue}, ${saturation}%, ${lightness}%)`
  }

  // Count instances for each value from 1 to 100
  const instanceCounts = {}
  results.value.newApps.forEach(app => {
    const instances = app.instances || 0
    if (instances >= 1 && instances <= 100) {
      instanceCounts[instances] = (instanceCounts[instances] || 0) + 1
    }
  })

  // Filter out values with no data and sort
  const filteredLabels = []
  const filteredData = []
  const filteredColors = []

  Object.keys(instanceCounts)
    .map(Number)
    .sort((a, b) => a - b)
    .forEach(value => {
      filteredLabels.push(value.toString())
      filteredData.push(instanceCounts[value])
      filteredColors.push(generateColor(value - 1))
    })

  return {
    labels: filteredLabels,
    datasets: [{
      label: 'Number of Apps',
      data: filteredData,
      backgroundColor: filteredColors,
      borderWidth: 0
    }]
  }
})

const categoryChartData = computed(() => {
  if (!results.value) return null

  const categories = new Map()

  results.value.newApps.forEach(app => {
    const cat = app.category
    categories.set(cat, (categories.get(cat) || 0) + 1)
  })

  return {
    labels: Array.from(categories.keys()),
    datasets: [{
      data: Array.from(categories.values()),
      backgroundColor: [
        '#667eea', '#764ba2', '#f093fb', '#4facfe',
        '#43e97b', '#fa709a'
      ],
      borderWidth: 0
    }]
  }
})

const orbitChartData = computed(() => {
  if (!results.value || !results.value.summary) return null

  const orbitCount = results.value.summary.orbitApps || 0
  const nonOrbitCount = results.value.summary.newApps - orbitCount

  return {
    labels: ['Orbit Apps', 'Non-Orbit Apps'],
    datasets: [{
      data: [orbitCount, nonOrbitCount],
      backgroundColor: [
        '#f093fb',
        '#667eea'
      ],
      borderWidth: 0
    }]
  }
})

const multiComponentChartData = computed(() => {
  if (!results.value || !results.value.summary) return null

  const multiCount = results.value.summary.multiComponentApps || 0
  const singleCount = results.value.summary.newApps - multiCount

  return {
    labels: ['Multi-Component', 'Single Component'],
    datasets: [{
      data: [multiCount, singleCount],
      backgroundColor: [
        '#43e97b',
        '#667eea'
      ],
      borderWidth: 0
    }]
  }
})

const repoVisibilityChartData = computed(() => {
  if (!results.value) return null

  let publicCount = 0
  let privateCount = 0

  results.value.newApps.forEach(app => {
    if (app.repotags && app.repotags.length > 0) {
      publicCount++
    } else {
      privateCount++
    }
  })

  if (publicCount === 0 && privateCount === 0) return null

  return {
    labels: ['Public Repositories', 'Private Repositories'],
    datasets: [{
      data: [publicCount, privateCount],
      backgroundColor: [
        '#00bcd4',  // Cyan for public
        '#ff9800'   // Orange for private
      ],
      borderWidth: 0
    }]
  }
})

const highInstanceApps = computed(() => {
  if (!results.value || !results.value.newApps) return []

  return results.value.newApps
    .filter(app => app.instances > instanceThreshold.value)
    .sort((a, b) => b.instances - a.instances)
})

const chartOptions = computed(() => {
  const textColor = isDark.value ? '#ffffff' : '#000000'
  const gridColor = isDark.value ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)'

  return {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        display: false
      },
      datalabels: {
        display: false
      }
    },
    scales: {
      x: {
        ticks: {
          color: textColor
        },
        grid: {
          color: gridColor
        },
        border: {
          color: gridColor
        }
      },
      y: {
        ticks: {
          color: textColor
        },
        grid: {
          color: gridColor
        },
        border: {
          color: gridColor
        }
      }
    }
  }
})

const doughnutOptions = computed(() => {
  const textColor = isDark.value ? '#ffffff' : '#000000'

  return {
    responsive: true,
    maintainAspectRatio: false,
    elements: {
      arc: {
        borderRadius: 0
      }
    },
    plugins: {
      legend: {
        position: 'bottom',
        labels: {
          color: textColor
        }
      },
      tooltip: {
        callbacks: {
          label: function(context) {
            const label = context.label || ''
            const value = context.parsed || 0
            const total = context.dataset.data.reduce((acc, val) => acc + val, 0)
            const percentage = ((value / total) * 100).toFixed(1)
            return `${label}: ${value} (${percentage}%)`
          }
        }
      },
      datalabels: {
        display: true,
        color: '#fff',
        font: {
          weight: 'bold',
          size: 14
        },
        formatter: function(value, context) {
          const total = context.dataset.data.reduce((acc, val) => acc + val, 0)
          const percentage = ((value / total) * 100).toFixed(1)
          return percentage > 5 ? `${percentage}%` : ''
        }
      }
    }
  }
})

const timelineOptions = computed(() => {
  const textColor = isDark.value ? '#ffffff' : '#000000'
  const gridColor = isDark.value ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)'
  const bgColor = isDark.value ? 'rgba(30, 30, 30, 0.95)' : 'rgba(255, 255, 255, 0.95)'

  return {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        display: true,
        position: 'top',
        labels: {
          color: textColor
        }
      },
      datalabels: {
        display: false
      },
      tooltip: {
        enabled: false,
        external: function(context) {
          // Tooltip Element
          let tooltipEl = document.getElementById('chartjs-tooltip')

          // Create element on first render
          if (!tooltipEl) {
            tooltipEl = document.createElement('div')
            tooltipEl.id = 'chartjs-tooltip'
            tooltipEl.innerHTML = '<div></div>'
            document.body.appendChild(tooltipEl)
          }

          // Hide if no tooltip
          const tooltipModel = context.tooltip
          if (tooltipModel.opacity === 0) {
            tooltipEl.style.opacity = 0
            return
          }

          // Set Text
          if (tooltipModel.body) {
            const dataPoints = tooltipModel.dataPoints
            const dataPoint = dataPoints[0]
            const dataIndex = dataPoint.dataIndex
            const datasetIndex = dataPoint.datasetIndex
            const dataset = context.chart.data.datasets[datasetIndex]

            const label = dataset.label || ''
            const value = dataPoint.parsed.y

            let innerHtml = `<div style="padding: 6px 10px; background: ${bgColor}; border: 1px solid #666; border-radius: 4px; box-shadow: 0 2px 8px rgba(0,0,0,0.3); font-size: 12px;">`

            // First line - label and value
            innerHtml += `<div style="color: ${textColor}; margin-bottom: 3px;">${label}: ${value}</div>`

            // Second line - trend with colored text
            if (dataIndex > 0) {
              const currentValue = dataset.data[dataIndex]
              const previousValue = dataset.data[dataIndex - 1]
              const change = currentValue - previousValue
              const percentChange = previousValue !== 0 ? ((change / previousValue) * 100).toFixed(1) : 0

              let trendText = ''
              let trendColor = textColor

              if (change > 0) {
                trendText = `▲ +${change} (+${percentChange}%)`
                trendColor = '#4caf50'
              } else if (change < 0) {
                trendText = `▼ ${change} (${percentChange}%)`
                trendColor = '#f44336'
              } else {
                trendText = '━ No change (0%)'
              }

              innerHtml += `<div style="color: ${trendColor}; font-weight: bold; font-size: 11px;">${trendText}</div>`
            }

            innerHtml += '</div>'

            tooltipEl.innerHTML = innerHtml
          }

          const position = context.chart.canvas.getBoundingClientRect()

          // Display, position, and set styles for font
          tooltipEl.style.opacity = 1
          tooltipEl.style.position = 'absolute'
          tooltipEl.style.left = position.left + window.pageXOffset + tooltipModel.caretX + 'px'
          tooltipEl.style.top = position.top + window.pageYOffset + tooltipModel.caretY + 'px'
          tooltipEl.style.pointerEvents = 'none'
          tooltipEl.style.transition = 'all 0.1s ease'
        }
      }
    },
    scales: {
      x: {
        ticks: {
          color: textColor
        },
        grid: {
          color: gridColor
        },
        border: {
          color: gridColor
        }
      },
      y: {
        beginAtZero: true,
        ticks: {
          stepSize: 1,
          color: textColor
        },
        grid: {
          color: gridColor
        },
        border: {
          color: gridColor
        }
      }
    }
  }
})
</script>

<style scoped>
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-15px);
  }
}

.animate-bounce {
  animation: bounce 1.5s ease-in-out infinite;
}

@keyframes glow {
  0%, 100% {
    filter: drop-shadow(0 0 5px rgba(var(--v-theme-error), 0.3));
    opacity: 1;
  }
  50% {
    filter: drop-shadow(0 0 12px rgba(var(--v-theme-error), 0.5));
    opacity: 1;
  }
}

.glow-icon {
  animation: glow 2s ease-in-out infinite;
}

</style>
