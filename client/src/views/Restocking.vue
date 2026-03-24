<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <!-- Success State -->
    <div v-if="orderResult" class="success-card">
      <div class="success-icon-wrap">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="success-icon">
          <path fill-rule="evenodd" d="M2.25 12c0-5.385 4.365-9.75 9.75-9.75s9.75 4.365 9.75 9.75-4.365 9.75-9.75 9.75S2.25 17.385 2.25 12zm13.36-1.814a.75.75 0 10-1.22-.872l-3.236 4.53L9.53 12.22a.75.75 0 00-1.06 1.06l2.25 2.25a.75.75 0 001.14-.094l3.75-5.25z" clip-rule="evenodd" />
        </svg>
      </div>
      <div class="success-body">
        <h3>{{ t('restocking.orderPlaced') }}</h3>
        <p>{{ t('restocking.orderNumber') }}: <strong>{{ orderResult.order_number }}</strong></p>
        <p>{{ t('restocking.expectedDelivery') }}: <strong>{{ orderResult.expected_delivery }}</strong></p>
        <p>{{ t('restocking.totalValue') }}: <strong>{{ currencySymbol }}{{ orderResult.total_value.toLocaleString() }}</strong></p>
        <button class="btn-secondary" @click="resetOrder">{{ t('restocking.placeAnother') }}</button>
      </div>
    </div>

    <!-- Main Content -->
    <div v-else>
      <!-- Budget Slider Card -->
      <div class="card budget-card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetTitle') }}</h3>
          <span class="budget-display">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
        </div>
        <div class="slider-wrap">
          <span class="slider-label">{{ currencySymbol }}1,000</span>
          <input
            type="range"
            class="budget-slider"
            :min="1000"
            :max="50000"
            :step="500"
            :value="budget"
            @input="onSliderInput"
          />
          <span class="slider-label">{{ currencySymbol }}50,000</span>
        </div>
        <p class="slider-hint">{{ t('restocking.sliderHint') }}</p>
      </div>

      <!-- Stats Cards -->
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">{{ t('restocking.totalBudget') }}</div>
          <div class="stat-value">{{ currencySymbol }}{{ budget.toLocaleString() }}</div>
        </div>
        <div class="stat-card" :class="{ warning: estimatedCost > budget }">
          <div class="stat-label">{{ t('restocking.estimatedCost') }}</div>
          <div class="stat-value">
            <span v-if="recLoading">--</span>
            <span v-else>{{ currencySymbol }}{{ (recommendations.total_cost || 0).toLocaleString() }}</span>
          </div>
        </div>
        <div class="stat-card" :class="budgetRemainingClass">
          <div class="stat-label">{{ t('restocking.budgetRemaining') }}</div>
          <div class="stat-value">
            <span v-if="recLoading">--</span>
            <span v-else>{{ currencySymbol }}{{ (recommendations.budget_remaining || 0).toLocaleString() }}</span>
          </div>
        </div>
        <div class="stat-card info">
          <div class="stat-label">{{ t('restocking.itemsToRestock') }}</div>
          <div class="stat-value">
            <span v-if="recLoading">--</span>
            <span v-else>{{ recommendedItems.length }}</span>
          </div>
        </div>
      </div>

      <!-- Recommendations Table -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendationsTitle') }}</h3>
          <div v-if="recLoading" class="inline-loading">{{ t('common.loading') }}</div>
        </div>

        <div v-if="recError" class="error">{{ recError }}</div>
        <div v-else-if="!recLoading && recommendedItems.length === 0" class="empty-state">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="table-container">
          <table class="restock-table">
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.trend') }}</th>
                <th class="col-num">{{ t('restocking.table.currentDemand') }}</th>
                <th class="col-num">{{ t('restocking.table.forecastedDemand') }}</th>
                <th class="col-num">{{ t('restocking.table.demandGap') }}</th>
                <th class="col-num">{{ t('restocking.table.unitCost') }}</th>
                <th class="col-num">{{ t('restocking.table.restockQty') }}</th>
                <th class="col-num">{{ t('restocking.table.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendedItems" :key="item.id">
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ translateProductName(item.item_name) }}</td>
                <td>
                  <span :class="['badge', item.trend]">
                    {{ t(`trends.${item.trend}`) }}
                  </span>
                </td>
                <td class="col-num">{{ item.current_demand }}</td>
                <td class="col-num"><strong>{{ item.forecasted_demand }}</strong></td>
                <td class="col-num">
                  <span :style="{ color: item.demand_gap > 0 ? '#16a34a' : '#ef4444' }">
                    {{ item.demand_gap > 0 ? '+' : '' }}{{ item.demand_gap }}
                  </span>
                </td>
                <td class="col-num">{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
                <td class="col-num"><strong>{{ item.restock_quantity }}</strong></td>
                <td class="col-num">
                  <strong>{{ currencySymbol }}{{ (item.unit_cost * item.restock_quantity).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Place Order Footer -->
        <div class="card-footer">
          <div class="footer-summary" v-if="!recLoading && recommendedItems.length > 0">
            <span>{{ t('restocking.totalItems', { count: recommendedItems.length }) }}</span>
            <span class="footer-total">{{ t('restocking.totalCost') }}: <strong>{{ currencySymbol }}{{ (recommendations.total_cost || 0).toLocaleString() }}</strong></span>
          </div>
          <button
            class="btn-primary"
            :disabled="submitting || recommendedItems.length === 0 || recLoading"
            @click="placeOrder"
          >
            <span v-if="submitting" class="spinner"></span>
            <span v-if="submitting">{{ t('restocking.submitting') }}</span>
            <span v-else>{{ t('restocking.placeOrder') }}</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency, translateProductName } = useI18n()

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    // Filters imported for consistency but not applied to this page
    const { getCurrentFilters } = useFilters()

    const budget = ref(10000)
    const recommendations = ref({})
    const recLoading = ref(false)
    const recError = ref(null)
    const submitting = ref(false)
    const orderResult = ref(null)

    let debounceTimer = null

    const recommendedItems = computed(() => {
      return recommendations.value.items || []
    })

    const estimatedCost = computed(() => recommendations.value.total_cost || 0)

    const budgetRemainingClass = computed(() => {
      const remaining = recommendations.value.budget_remaining
      if (remaining === undefined || remaining === null) return ''
      if (remaining < 0) return 'danger'
      if (remaining < budget.value * 0.1) return 'warning'
      return 'success'
    })

    const loadRecommendations = async () => {
      recLoading.value = true
      recError.value = null
      try {
        recommendations.value = await api.getRestockingRecommendations(budget.value)
      } catch (err) {
        recError.value = 'Failed to load recommendations: ' + err.message
        recommendations.value = {}
      } finally {
        recLoading.value = false
      }
    }

    const onSliderInput = (event) => {
      budget.value = Number(event.target.value)
      clearTimeout(debounceTimer)
      debounceTimer = setTimeout(() => {
        loadRecommendations()
      }, 300)
    }

    const placeOrder = async () => {
      if (submitting.value || recommendedItems.value.length === 0) return

      submitting.value = true
      try {
        const orderData = {
          items: recommendedItems.value.map(item => ({
            item_sku: item.item_sku,
            item_name: item.item_name,
            quantity: item.restock_quantity,
            unit_cost: item.unit_cost
          })),
          total_cost: recommendations.value.total_cost
        }
        orderResult.value = await api.submitRestockingOrder(orderData)
      } catch (err) {
        recError.value = 'Failed to place order: ' + err.message
      } finally {
        submitting.value = false
      }
    }

    const resetOrder = () => {
      orderResult.value = null
      loadRecommendations()
    }

    onMounted(loadRecommendations)

    return {
      t,
      currencySymbol,
      translateProductName,
      budget,
      recommendations,
      recommendedItems,
      estimatedCost,
      budgetRemainingClass,
      recLoading,
      recError,
      submitting,
      orderResult,
      onSliderInput,
      placeOrder,
      resetOrder
    }
  }
}
</script>

<style scoped>
.budget-card .card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.budget-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #2563eb;
  letter-spacing: -0.025em;
}

.slider-wrap {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.5rem 0;
}

.budget-slider {
  flex: 1;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
}

.slider-label {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 500;
  white-space: nowrap;
}

.slider-hint {
  font-size: 0.813rem;
  color: #94a3b8;
  margin-top: 0.5rem;
}

.inline-loading {
  font-size: 0.813rem;
  color: #64748b;
}

.empty-state {
  padding: 3rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

.restock-table {
  width: 100%;
  table-layout: auto;
}

.col-num {
  text-align: right;
  white-space: nowrap;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0 0;
  margin-top: 1rem;
  border-top: 1px solid #e2e8f0;
  gap: 1rem;
}

.footer-summary {
  display: flex;
  gap: 2rem;
  font-size: 0.875rem;
  color: #64748b;
}

.footer-total {
  color: #0f172a;
}

.btn-primary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.625rem 1.5rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.3);
}

.btn-primary:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.btn-secondary {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1.25rem;
  background: white;
  color: #2563eb;
  border: 1px solid #2563eb;
  border-radius: 8px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-top: 1rem;
}

.btn-secondary:hover {
  background: #eff6ff;
}

.spinner {
  display: inline-block;
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.4);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
  flex-shrink: 0;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.success-card {
  background: white;
  border: 1px solid #bbf7d0;
  border-radius: 10px;
  padding: 2rem;
  display: flex;
  align-items: flex-start;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.success-icon-wrap {
  flex-shrink: 0;
}

.success-icon {
  width: 48px;
  height: 48px;
  color: #16a34a;
}

.success-body h3 {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.75rem;
}

.success-body p {
  font-size: 0.938rem;
  color: #334155;
  margin-bottom: 0.375rem;
}
</style>
