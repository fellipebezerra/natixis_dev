<template>
  <v-container fluid class="price-band-container pa-4">
    <v-row no-gutters>

```
  <!-- ========== PAINEL ESQUERDO: Lista de Price Bands ========== -->
  <v-col cols="2" class="panel-left pr-2">
    <v-card flat outlined height="100%">
      <v-card-title class="panel-title py-2 px-3">
        Price Bands
        <v-spacer />
        <v-btn color="primary" small @click="onNewPriceBand">
          New
        </v-btn>
      </v-card-title>

      <v-divider />

      <v-card-text class="pa-0">
        <v-text-field
          v-model="searchBand"
          placeholder="Filter Price Band"
          dense
          outlined
          hide-details
          clearable
          class="ma-2"
          prepend-inner-icon="mdi-magnify"
        />

        <v-list dense class="band-list pa-0">
          <v-list-item
            v-for="band in filteredBands"
            :key="band.id"
            :class="{ 'selected-item': selectedBand && selectedBand.id === band.id }"
            @click="onSelectBand(band)"
          >
            <v-list-item-content>
              <v-list-item-title>{{ band.name }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>

          <v-list-item v-if="filteredBands.length === 0">
            <v-list-item-content>
              <v-list-item-title class="grey--text text-center">
                No records found
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card-text>
    </v-card>
  </v-col>

  <!-- ========== PAINEL CENTRAL: Formulário ========== -->
  <v-col cols="4" class="panel-center px-2">
    <v-card flat outlined height="100%">
      <v-card-title class="panel-title py-2 px-3">
        {{ isNew ? 'New Price Band' : 'Price Band Details' }}
      </v-card-title>

      <v-divider />

      <v-card-text>
        <v-form ref="form" v-model="formValid" lazy-validation>

          <v-text-field
            v-model="form.name"
            label="Name"
            outlined
            dense
            :rules="[rules.required]"
            class="mb-3"
          />

          <v-text-field
            v-model="form.description"
            label="Description"
            outlined
            dense
            class="mb-3"
          />

          <v-row>
            <v-col cols="6">
              <v-text-field
                v-model="form.minSpread"
                label="Min Spread"
                outlined
                dense
                type="number"
                :rules="[rules.required, rules.numeric]"
              />
            </v-col>
            <v-col cols="6">
              <v-text-field
                v-model="form.maxSpread"
                label="Max Spread"
                outlined
                dense
                type="number"
                :rules="[rules.required, rules.numeric]"
              />
            </v-col>
          </v-row>

          <v-checkbox
            v-model="form.isActive"
            label="Is Active"
            dense
            class="mt-0"
          />

        </v-form>
      </v-card-text>
    </v-card>
  </v-col>

  <!-- ========== PAINEL DIREITO: Dual Lists ========== -->
  <v-col cols="6" class="panel-right pl-2">
    <v-card flat outlined height="100%">
      <v-card-title class="panel-title py-2 px-3">
        Assignments
      </v-card-title>

      <v-divider />

      <v-card-text>

        <!-- ---- Currency Pairs ---- -->
        <div class="dual-list-label mb-1">Currency Pairs</div>
        <v-row no-gutters align="center" class="mb-4">
          <!-- Available -->
          <v-col cols="5">
            <div class="dual-list-header">Available</div>
            <v-text-field
              v-model="searchCurrencyPair"
              placeholder="Filter"
              dense
              outlined
              hide-details
              clearable
              class="mb-1"
              prepend-inner-icon="mdi-magnify"
            />
            <v-card outlined class="dual-list-box">
              <v-list dense class="pa-0">
                <v-list-item
                  v-for="item in filteredAvailableCurrencyPairs"
                  :key="item.id"
                  :class="{ 'selected-item': selectedAvailableCurrencyPairs.includes(item.id) }"
                  @click="toggleAvailableCurrencyPair(item.id)"
                >
                  <v-list-item-title>{{ item.name }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>

          <!-- Transfer Buttons -->
          <v-col cols="2" class="text-center">
            <v-btn icon small @click="assignCurrencyPairs" :disabled="selectedAvailableCurrencyPairs.length === 0">
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
            <br />
            <v-btn icon small @click="unassignCurrencyPairs" :disabled="selectedAssignedCurrencyPairs.length === 0">
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
          </v-col>

          <!-- Assigned -->
          <v-col cols="5">
            <div class="dual-list-header">Assigned</div>
            <v-card outlined class="dual-list-box mt-7">
              <v-list dense class="pa-0">
                <v-list-item
                  v-for="item in assignedCurrencyPairs"
                  :key="item.id"
                  :class="{ 'selected-item': selectedAssignedCurrencyPairs.includes(item.id) }"
                  @click="toggleAssignedCurrencyPair(item.id)"
                >
                  <v-list-item-title>{{ item.name }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>
        </v-row>

        <v-divider class="mb-4" />

        <!-- ---- Trading Groups ---- -->
        <div class="dual-list-label mb-1">Trading Groups</div>
        <v-row no-gutters align="center">
          <!-- Available -->
          <v-col cols="5">
            <div class="dual-list-header">Available</div>
            <v-text-field
              v-model="searchTradingGroup"
              placeholder="Filter"
              dense
              outlined
              hide-details
              clearable
              class="mb-1"
              prepend-inner-icon="mdi-magnify"
            />
            <v-card outlined class="dual-list-box">
              <v-list dense class="pa-0">
                <v-list-item
                  v-for="item in filteredAvailableTradingGroups"
                  :key="item.id"
                  :class="{ 'selected-item': selectedAvailableTradingGroups.includes(item.id) }"
                  @click="toggleAvailableTradingGroup(item.id)"
                >
                  <v-list-item-title>{{ item.name }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>

          <!-- Transfer Buttons -->
          <v-col cols="2" class="text-center">
            <v-btn icon small @click="assignTradingGroups" :disabled="selectedAvailableTradingGroups.length === 0">
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
            <br />
            <v-btn icon small @click="unassignTradingGroups" :disabled="selectedAssignedTradingGroups.length === 0">
              <v-icon>mdi-chevron-left</v-icon>
            </v-btn>
          </v-col>

          <!-- Assigned -->
          <v-col cols="5">
            <div class="dual-list-header">Assigned</div>
            <v-card outlined class="dual-list-box mt-7">
              <v-list dense class="pa-0">
                <v-list-item
                  v-for="item in assignedTradingGroups"
                  :key="item.id"
                  :class="{ 'selected-item': selectedAssignedTradingGroups.includes(item.id) }"
                  @click="toggleAssignedTradingGroup(item.id)"
                >
                  <v-list-item-title>{{ item.name }}</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>
        </v-row>

      </v-card-text>
    </v-card>
  </v-col>

</v-row>

<!-- ========== BOTÕES DE AÇÃO ========== -->
<v-row class="mt-3" justify="end">
  <v-col cols="auto">
    <v-btn outlined @click="onCancel" class="mr-2">Cancel</v-btn>
    <v-btn color="primary" @click="onSave" :loading="saving" :disabled="!formValid">
      Save
    </v-btn>
  </v-col>
</v-row>

<!-- ========== SNACKBAR ========== -->
<v-snackbar v-model="snackbar.show" :color="snackbar.color" top right timeout="3000">
  {{ snackbar.message }}
  <template v-slot:action="{ attrs }">
    <v-btn text v-bind="attrs" @click="snackbar.show = false">Close</v-btn>
  </template>
</v-snackbar>
```

  </v-container>
</template>

<script>
export default {
  name: 'PriceBand',

  data () {
    return {
      // Lista principal
      priceBands: [],
      searchBand: '',
      selectedBand: null,
      isNew: false,
      saving: false,
      formValid: true,

      // Formulário
      form: {
        name: '',
        description: '',
        minSpread: null,
        maxSpread: null,
        isActive: true
      },

      // Regras de validação
      rules: {
        required: v => !!v || 'Field is required',
        numeric: v => !isNaN(v) || 'Must be a number'
      },

      // Currency Pairs - dual list
      allCurrencyPairs: [],
      assignedCurrencyPairs: [],
      searchCurrencyPair: '',
      selectedAvailableCurrencyPairs: [],
      selectedAssignedCurrencyPairs: [],

      // Trading Groups - dual list
      allTradingGroups: [],
      assignedTradingGroups: [],
      searchTradingGroup: '',
      selectedAvailableTradingGroups: [],
      selectedAssignedTradingGroups: [],

      // Snackbar
      snackbar: {
        show: false,
        message: '',
        color: 'success'
      }
    }
  },

  computed: {
    filteredBands () {
      if (!this.searchBand) return this.priceBands
      return this.priceBands.filter(b =>
        b.name.toLowerCase().includes(this.searchBand.toLowerCase())
      )
    },

    assignedCurrencyPairIds () {
      return this.assignedCurrencyPairs.map(x => x.id)
    },

    availableCurrencyPairs () {
      return this.allCurrencyPairs.filter(x => !this.assignedCurrencyPairIds.includes(x.id))
    },

    filteredAvailableCurrencyPairs () {
      if (!this.searchCurrencyPair) return this.availableCurrencyPairs
      return this.availableCurrencyPairs.filter(x =>
        x.name.toLowerCase().includes(this.searchCurrencyPair.toLowerCase())
      )
    },

    assignedTradingGroupIds () {
      return this.assignedTradingGroups.map(x => x.id)
    },

    availableTradingGroups () {
      return this.allTradingGroups.filter(x => !this.assignedTradingGroupIds.includes(x.id))
    },

    filteredAvailableTradingGroups () {
      if (!this.searchTradingGroup) return this.availableTradingGroups
      return this.availableTradingGroups.filter(x =>
        x.name.toLowerCase().includes(this.searchTradingGroup.toLowerCase())
      )
    }
  },

  mounted () {
    this.loadPriceBands()
    this.loadCurrencyPairs()
    this.loadTradingGroups()
  },

  methods: {

    // ---- Carregamento de dados ----

    async loadPriceBands () {
      try {
        const res = await this.$http.get('/api/price-bands')
        this.priceBands = res.body
      } catch (e) {
        this.showSnackbar('Error loading price bands', 'error')
      }
    },

    async loadCurrencyPairs () {
      try {
        const res = await this.$http.get('/api/currency-pairs')
        this.allCurrencyPairs = res.body
      } catch (e) {
        this.showSnackbar('Error loading currency pairs', 'error')
      }
    },

    async loadTradingGroups () {
      try {
        const res = await this.$http.get('/api/trading-groups')
        this.allTradingGroups = res.body
      } catch (e) {
        this.showSnackbar('Error loading trading groups', 'error')
      }
    },

    async loadAssignments (bandId) {
      try {
        const res = await this.$http.get(`/api/price-bands/${bandId}/assignments`)
        this.assignedCurrencyPairs = res.body.currencyPairs || []
        this.assignedTradingGroups = res.body.tradingGroups || []
      } catch (e) {
        this.showSnackbar('Error loading assignments', 'error')
      }
    },

    // ---- Seleção de Price Band ----

    onSelectBand (band) {
      this.selectedBand = band
      this.isNew = false
      this.form = { ...band }
      this.resetSelections()
      this.loadAssignments(band.id)
    },

    onNewPriceBand () {
      this.selectedBand = null
      this.isNew = true
      this.form = { name: '', description: '', minSpread: null, maxSpread: null, isActive: true }
      this.assignedCurrencyPairs = []
      this.assignedTradingGroups = []
      this.resetSelections()
      if (this.$refs.form) this.$refs.form.resetValidation()
    },

    // ---- Dual List: Currency Pairs ----

    toggleAvailableCurrencyPair (id) {
      const idx = this.selectedAvailableCurrencyPairs.indexOf(id)
      if (idx === -1) this.selectedAvailableCurrencyPairs.push(id)
      else this.selectedAvailableCurrencyPairs.splice(idx, 1)
    },

    toggleAssignedCurrencyPair (id) {
      const idx = this.selectedAssignedCurrencyPairs.indexOf(id)
      if (idx === -1) this.selectedAssignedCurrencyPairs.push(id)
      else this.selectedAssignedCurrencyPairs.splice(idx, 1)
    },

    assignCurrencyPairs () {
      const toMove = this.availableCurrencyPairs.filter(x =>
        this.selectedAvailableCurrencyPairs.includes(x.id)
      )
      this.assignedCurrencyPairs = [...this.assignedCurrencyPairs, ...toMove]
      this.selectedAvailableCurrencyPairs = []
    },

    unassignCurrencyPairs () {
      this.assignedCurrencyPairs = this.assignedCurrencyPairs.filter(x =>
        !this.selectedAssignedCurrencyPairs.includes(x.id)
      )
      this.selectedAssignedCurrencyPairs = []
    },

    // ---- Dual List: Trading Groups ----

    toggleAvailableTradingGroup (id) {
      const idx = this.selectedAvailableTradingGroups.indexOf(id)
      if (idx === -1) this.selectedAvailableTradingGroups.push(id)
      else this.selectedAvailableTradingGroups.splice(idx, 1)
    },

    toggleAssignedTradingGroup (id) {
      const idx = this.selectedAssignedTradingGroups.indexOf(id)
      if (idx === -1) this.selectedAssignedTradingGroups.push(id)
      else this.selectedAssignedTradingGroups.splice(idx, 1)
    },

    assignTradingGroups () {
      const toMove = this.availableTradingGroups.filter(x =>
        this.selectedAvailableTradingGroups.includes(x.id)
      )
      this.assignedTradingGroups = [...this.assignedTradingGroups, ...toMove]
      this.selectedAvailableTradingGroups = []
    },

    unassignTradingGroups () {
      this.assignedTradingGroups = this.assignedTradingGroups.filter(x =>
        !this.selectedAssignedTradingGroups.includes(x.id)
      )
      this.selectedAssignedTradingGroups = []
    },

    // ---- Guardar ----

    async onSave () {
      if (!this.$refs.form.validate()) return

      this.saving = true
      try {
        const payload = {
          ...this.form,
          currencyPairIds: this.assignedCurrencyPairs.map(x => x.id),
          tradingGroupIds: this.assignedTradingGroups.map(x => x.id)
        }

        if (this.isNew) {
          const res = await this.$http.post('/api/price-bands', payload)
          this.priceBands.push(res.body)
          this.selectedBand = res.body
          this.isNew = false
        } else {
          await this.$http.put(`/api/price-bands/${this.selectedBand.id}`, payload)
          const idx = this.priceBands.findIndex(b => b.id === this.selectedBand.id)
          if (idx !== -1) this.$set(this.priceBands, idx, { ...this.selectedBand, ...this.form })
        }

        this.showSnackbar('Saved successfully', 'success')
      } catch (e) {
        this.showSnackbar('Error saving price band', 'error')
      } finally {
        this.saving = false
      }
    },

    onCancel () {
      if (this.selectedBand) {
        this.onSelectBand(this.selectedBand)
      } else {
        this.onNewPriceBand()
      }
    },

    // ---- Helpers ----

    resetSelections () {
      this.selectedAvailableCurrencyPairs = []
      this.selectedAssignedCurrencyPairs = []
      this.selectedAvailableTradingGroups = []
      this.selectedAssignedTradingGroups = []
    },

    showSnackbar (message, color = 'success') {
      this.snackbar = { show: true, message, color }
    }
  }
}
</script>

<style scoped>
.price-band-container {
  height: calc(100vh - 64px);
}

.panel-title {
  font-size: 14px !important;
  font-weight: 600;
}

.band-list {
  max-height: calc(100vh - 220px);
  overflow-y: auto;
}

.selected-item {
  background-color: #1976D2 !important;
  color: white !important;
}

.selected-item .v-list-item__title {
  color: white !important;
}

.dual-list-box {
  height: 200px;
  overflow-y: auto;
}

.dual-list-label {
  font-weight: 600;
  font-size: 13px;
  color: #555;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.dual-list-header {
  font-size: 12px;
  color: #888;
  margin-bottom: 4px;
}
</style>
