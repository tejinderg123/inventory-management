<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking Planner</h2>
      <p>Review demand forecasts and place batch restock orders for items with increasing demand below their reorder point.</p>
    </div>

    <div v-if="loading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <!-- Budget Allocation Card -->
      <div class="card budget-section">
        <span class="budget-label">Budget Allocation</span>
        <div class="budget-display">{{ formatCurrency(budget) }}</div>
        <input
          type="range"
          class="budget-slider"
          :min="0"
          :max="maxBudget"
          :step="1000"
          v-model.number="budget"
        />
        <div class="budget-range-labels">
          <span>$0</span>
          <span>{{ formatCurrency(maxBudget) }}</span>
        </div>
      </div>

      <!-- Recommended Restocks Card -->
      <div class="card">
        <div class="card-header">
          <div>
            <h3 class="card-title">Recommended Restocks</h3>
            <span class="rec-subtitle">Items with increasing demand that are below reorder point</span>
          </div>
          <button
            class="place-order-btn"
            :disabled="selectedItems.length === 0 || placing"
            @click="placeOrder"
          >
            {{ placing ? 'Placing...' : 'Place Order' }}
          </button>
        </div>

        <div v-if="orderSuccess" class="success-banner">
          Order placed successfully. Check the Orders tab to track delivery.
        </div>

        <div v-if="recommendations.length === 0" class="empty-state">
          No restock recommendations at this time. All items with increasing demand are above their reorder points.
        </div>

        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>Item Name</th>
                <th>SKU</th>
                <th>Current Stock</th>
                <th>Reorder Point</th>
                <th>Qty to Order</th>
                <th>Unit Cost</th>
                <th>Line Total</th>
                <th>Status</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.item_sku">
                <td>{{ item.item_name }}</td>
                <td>{{ item.item_sku }}</td>
                <td>{{ item.quantity_on_hand }}</td>
                <td>{{ item.reorder_point }}</td>
                <td>{{ item.order_quantity }}</td>
                <td>${{ item.unit_cost.toFixed(2) }}</td>
                <td>{{ formatCurrency(item.line_cost) }}</td>
                <td>
                  <span v-if="isSelected(item)" class="badge success">Within budget</span>
                  <span v-else class="badge-over">Over budget</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="summary-row">
          <span>{{ selectedItems.length }} items selected</span>
          <span>Total: <strong>{{ formatCurrency(totalCost) }}</strong></span>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'

export default {
  name: 'Restocking',
  setup() {
    const allForecasts = ref([])
    const allInventory = ref([])
    const loading = ref(true)
    const error = ref(null)
    const placing = ref(false)
    const orderSuccess = ref(false)
    const budget = ref(0)

    const formatCurrency = (value) => {
      return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' })
    }

    // Items with increasing trend that are at or below their reorder point,
    // sorted by largest demand gap (forecasted - current) descending
    const recommendations = computed(() => {
      return allForecasts.value
        .filter(f => f.trend === 'increasing')
        .map(f => {
          const inv = allInventory.value.find(i => i.sku === f.item_sku)
          if (!inv || inv.quantity_on_hand > inv.reorder_point) return null
          const order_quantity = f.forecasted_demand
          const line_cost = inv.unit_cost * order_quantity
          return {
            item_sku: f.item_sku,
            item_name: f.item_name,
            current_demand: f.current_demand,
            forecasted_demand: f.forecasted_demand,
            quantity_on_hand: inv.quantity_on_hand,
            reorder_point: inv.reorder_point,
            unit_cost: inv.unit_cost,
            order_quantity,
            line_cost
          }
        })
        .filter(Boolean)
        .sort((a, b) => (b.forecasted_demand - b.current_demand) - (a.forecasted_demand - a.current_demand))
    })

    // Round up total cost to the nearest $1,000 to give a clean slider ceiling
    const maxBudget = computed(() => {
      return Math.ceil(recommendations.value.reduce((sum, r) => sum + r.line_cost, 0) / 1000) * 1000
    })

    // Greedily include items in priority order until the running total exceeds the budget
    const selectedItems = computed(() => {
      let running = 0
      const result = []
      for (const item of recommendations.value) {
        if (running + item.line_cost <= budget.value) {
          running += item.line_cost
          result.push(item)
        }
      }
      return result
    })

    const totalCost = computed(() => {
      return selectedItems.value.reduce((sum, item) => sum + item.line_cost, 0)
    })

    const isSelected = (item) => {
      return selectedItems.value.some(s => s.item_sku === item.item_sku)
    }

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const [forecasts, inventory] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        allForecasts.value = forecasts
        allInventory.value = inventory
        // Default budget to half of maximum after data is loaded
        budget.value = Math.floor(maxBudget.value / 2)
      } catch (err) {
        error.value = 'Failed to load data'
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const placeOrder = async () => {
      if (selectedItems.value.length === 0 || placing.value) return
      try {
        placing.value = true
        const payload = selectedItems.value.map(item => ({
          item_sku: item.item_sku,
          item_name: item.item_name,
          quantity: item.order_quantity,
          unit_cost: item.unit_cost,
          supplier_name: 'TBD'
        }))
        await api.createRestockOrders(payload)
        orderSuccess.value = true
        budget.value = Math.floor(maxBudget.value / 2)
        setTimeout(() => { orderSuccess.value = false }, 4000)
      } catch (err) {
        error.value = 'Failed to place order'
        console.error(err)
      } finally {
        placing.value = false
      }
    }

    onMounted(loadData)

    return {
      loading,
      error,
      budget,
      placing,
      orderSuccess,
      recommendations,
      maxBudget,
      selectedItems,
      totalCost,
      isSelected,
      formatCurrency,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

.budget-section {
  margin-bottom: 1.25rem;
}

.budget-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.budget-display {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  margin: 0.5rem 0;
}

.budget-slider {
  width: 100%;
  accent-color: #2563eb;
  cursor: pointer;
  margin: 0.5rem 0;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #64748b;
}

.rec-subtitle {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 400;
  display: block;
  margin-top: 0.25rem;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

.badge-over {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  background: #f1f5f9;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.summary-row {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 1rem;
  padding: 0.75rem;
  border-top: 1px solid #e2e8f0;
  font-size: 0.875rem;
  color: #64748b;
}

.summary-row strong {
  color: #0f172a;
}

.success-banner {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
  font-size: 0.875rem;
  font-weight: 500;
}

.place-order-btn {
  padding: 0.5rem 1.25rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #1d4ed8;
}

.place-order-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
</style>
