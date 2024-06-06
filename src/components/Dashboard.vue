<template>
  <div>
    <div>
      Data Goes Here
    </div>
    <div class="summary-dashboard">
      <div style="display: flex; justify-content: space-around;">
        <div class="summary-dashboard-item">
          <div>
            {{ transactions.length }}
          </div>
          <div>Total Transactions</div>
        </div>
        <div class="summary-dashboard-item">
          <div>
            {{ summaryData.transactionsByState }}
          </div>
          <div>Transactions By State</div>
        </div>
        <div class="summary-dashboard-item">
          <div>
            {{ summaryData.totalItems }}
          </div>
          <div>Total Items Sold</div>
        </div>
        <div class="summary-dashboard-item">
          <div>
            {{ summaryData.totalRevenue }}
          </div>
          <div>Total Revenue</div>
        </div>
        <div class="summary-dashboard-item">
          <div>
            {{ summaryData.totalTaxLiability }}
          </div>
          <div>Total Tax Liability</div>
        </div>
        <div class="summary-dashboard-item">
          <div>
            {{ summaryData.totalTaxLiabilityByState }}
          </div>
          <div>Tax Liability By State</div>
        </div>
      </div>
    </div>
    <div class="dashboard-filters" style="display: flex; justify-content: space-around;">
      <div>
        <div>Transaction Type</div>
        <div v-for="transactionType of filtersBase.transactionType">
          <input type="checkbox" :id="transactionType" :value="transactionType" v-model="filters.transactionType">
          <label :for="transactionType">{{ transactionType }}</label>
        </div>
      </div>
      <div>
        <div>State</div>
        <div v-for="state of filtersBase.state">
          <input type="checkbox" :id="state" :value="state" v-model="filters.state">
          <label :for="state">{{ state }}</label>
        </div>
      </div>
      <div>
        <div>Is Taxed</div>
        <input type="checkbox" id="True" :value="true" v-model="filters.isTaxed">
        <label for="True">True</label>
        <input type="checkbox" id="False" :value="false" v-model="filters.isTaxed">
        <label for="False">False</label>
      </div>
      <div>
        <div>Has Discount</div>
        <input type="checkbox" id="True" :value="true" v-model="filters.hasDiscount">
        <label for="True">True</label>
        <input type="checkbox" id="False" :value="false" v-model="filters.hasDiscount">
        <label for="False">False</label>
      </div>
    </div>
    <div v-for="transaction of transactions">
      <div style="margin-top: 20px;" v-if="passesTransactionLevelFilter(transaction)">
        <div style="display: flex; justify-content: left; border: solid black 1px;">
          <div style="margin-right: 10px;">
            {{transaction.type}}: {{transaction.address.country}},
          </div>
          <div style="margin-right: 10px;">
            {{transaction.address.street}}, {{transaction.address.city}}, {{transaction.address.state}}, {{transaction.address.postal_code}}
          </div>
        </div>
        <div class="transaction-table transaction-table-header" style="border: solid black 1px;">
            <div>Name</div>
            <div>Price</div>
            <div>Quantity</div>
            <div>Currency</div>
            <div>Discount</div>
            <div>Discount Type</div>
            <div>Final Price</div>
            <div>Is Taxed</div>
            <div>Tax Liability</div>
          </div>
        <div v-for="item of transaction.line_items">
          <div class="transaction-table" style="border: solid black 1px;" v-if="passesItemLevelFilter(item)">
            <div>
              {{item.name}}
            </div>
            <div>
              {{item.price}}
            </div>
            <div>
              {{item.quantity}}
            </div>
            <div>
              {{item.currency}}
            </div>
            <div>
              {{item.discount}}
            </div>
            <div>
              {{item.discount_type}}
            </div>
            <div>
              {{item.finalPrice}}
            </div>
            <div>
              {{item.isTaxed}}
            </div>
            <div>
              {{item.taxAmount}}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  export default {
    name: 'Dashboard',
    mounted() {
      this.getTransactions()
    },
    data() {
      return {
        codeToStatemapping: {
          'ny_us': 'NY',
          'ca_us': 'CA',
          'wa_us': 'WA'
        },
        transactions: [],
        summaryData: {
          totalTransactions: 0,
          totalItems: 0,
          totalRevenue: 0,
          transactionsByState: {},
          transactionsByDebitOrCredit: {},
          totalTaxLiability: 0,
          totalTaxLiabilityByState: {}
        },
        filtersBase: {
          transactionType: [],
          state: [],
          isTaxed: [true, false], // Always either true or false
          hasDiscount: [true, false],
        },
        filters: {
          transactionType: [],
          state: [],
          isTaxed: [true, false],
          hasDiscount: [true, false],
        }
      }
    },
    methods: {
      resetAllData () {
        this.summaryData = {
          totalTransactions: 0,
          totalItems: 0,
          totalRevenue: 0,
          transactionsByState: {},
          transactionsByDebitOrCredit: {},
          totalTaxLiability: 0,
          totalTaxLiabilityByState: {},
        }
        this.transactions = []
      },
      passesTransactionLevelFilter (transaction) {
        if (!(this.filters.transactionType.includes(transaction.type))){
          return false
        }
        if (!(this.filters.state.includes(transaction.address.state))){
          return false
        }
        return true
      },
      passesItemLevelFilter (item) {
        if (!(this.filters.isTaxed.includes(item.isTaxed))){
          return false
        }
        // Special Case for discount
        if (!item.discount && !this.filters.hasDiscount.includes(false)) {
          return false
        }
        if (item.discount && !this.filters.hasDiscount.includes(true)){
          return false
        }
        return true
      },
      addTransactionToSummaryData (transaction) {
        this.summaryData.totalItems += transaction.line_items.length
        const state = transaction.address.state
        if (!(state in this.summaryData.transactionsByState)) {
          this.summaryData.transactionsByState[state] = 0
        }
        this.summaryData.transactionsByState[state] += 1
      },
      addTaxAmountToSummaryData (state, taxAmount) {
        this.summaryData.totalTaxLiability += taxAmount
        if (!(state in this.summaryData.totalTaxLiabilityByState)) {
          this.summaryData.totalTaxLiabilityByState[state] = 0
        }
        this.summaryData.totalTaxLiabilityByState[state] += taxAmount
      },
      addRevenueToSummaryData (finalPrice) {
        this.summaryData.totalRevenue += finalPrice
      },
      addStateToFilters (state) {
        if (!(this.filtersBase.state.includes(state))) {
          this.filtersBase.state.push(state)
        }
      },
      addTransactionTypeToFilters (transactionType) {
        if (!(this.filtersBase.transactionType.includes(transactionType))) {
          this.filtersBase.transactionType.push(transactionType)
        }
      },
      async getTransactions () {
        const url = 'https://server.getsphere.com/tax_api/test_transactions'
        const response = await axios.get(url, { 'headers': { 'Authorization': 'Bearer rX8HOSkFCXR0Jy6DwA7Y9iSJL3a7gP65m2eazcW75AE7XGKS3LlqVXlFocloUgKc' } })
        const transactions = response.data.transactions
        const taxAuthorities = response.data.tax_authority

        const taxMapping = {}
        for (const taxAuthority of taxAuthorities) {
          if (taxAuthority.name in this.codeToStatemapping) {
            const state = this.codeToStatemapping[taxAuthority.name]
            taxMapping[state] = taxAuthority
          }
        }

        this.resetAllData()
        for (const transaction of transactions) {
          const state = transaction.address.state
          this.addStateToFilters(state)
          this.addTransactionTypeToFilters(transaction.type)
          this.addTransactionToSummaryData(transaction)
          for (const lineItem of transaction.line_items) {
            const isTaxed = taxMapping[state].taxable_items.includes(lineItem.name)
            const totalPrice = lineItem.price * lineItem.quantity
            let discountAmount = 0
            if (lineItem.discount) {
              discountAmount = lineItem.discount_type == 'percentage' ? totalPrice*(lineItem.discount/100) : lineItem.discount
            }
            const finalPrice = totalPrice - discountAmount
            this.addRevenueToSummaryData(finalPrice)
            // Set new values
            lineItem.finalPrice = finalPrice
            lineItem.isTaxed = isTaxed
            lineItem.taxAmount = isTaxed ? finalPrice*taxMapping[state].rate/100 : 0;
            this.addTaxAmountToSummaryData(state, lineItem.taxAmount)
          }
          this.transactions.push(transaction)
        }
        // Now all transactions have their updated price
        console.log(transactions)
        // Set all filters to include everything we get from the incoming response
        this.filters = JSON.parse(JSON.stringify(this.filtersBase))
      }
    }
  }
</script>

<style scoped>

.transaction-table-header {
  color: white;
  background-color: cadetblue;
}

.transaction-table {
  width: 100%;
  display: grid;
  grid-template-columns: 11% 11% 11% 11% 11% 11% 11% 11% 11%;
}
</style>
