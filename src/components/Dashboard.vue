<template>
<div class="container">
    <div class="heading">Tax Liability Dashboard</div>
    <div class="summary-dashboard">
        <div class="top_box">

            <div class="summary-dashboard-item">
                <div class="title">Total Transactions</div>
                <div>
                    {{ filteredTransactions.length }}
                </div>
            </div>

            <div class="summary-dashboard-item">
                <div class="title">Transactions By State</div>
                <div class="record">
                    <div v-for="(transactionsState, state) in filteredSummaryData.transactionsByState" :key="state">
                        <span> {{ state }} </span> {{ transactionsState }}
                    </div>
                </div>
            </div>

            <div class="summary-dashboard-item">
                <div class="title">Total Items Sold</div>
                <div>
                    {{ filteredSummaryData.totalItems }}
                </div>
            </div>

            <div class="summary-dashboard-item">
                <div class="title">Total Revenue</div>
                <div>
                    {{ formatCurrency(filteredSummaryData.totalRevenue) }}
                </div>
            </div>

            <div class="summary-dashboard-item">
                <div class="title">Total Tax Liability</div>
                <div>
                    {{ formatCurrency(filteredSummaryData.totalTaxLiability) }}
                </div>
            </div>

            <div class="summary-dashboard-item">
                <div class="title">Tax Liability By State</div>
                <div class="record">
                    <div v-for="(taxLiability, state) in filteredSummaryData.totalTaxLiabilityByState" :key="state">
                        <span> {{ state }} </span> ${{ taxLiability.toFixed(2) }}
                    </div>
                </div>
            </div>

        </div>
    </div>
    <div class="dashboard-filters" style="display: flex; justify-content: space-around">
        <div>
            <div class="label">Transaction Type</div>
            <div v-for="transactionType of filtersBase.transactionType">
                <input type="checkbox" :id="transactionType" :value="transactionType" v-model="filters.transactionType" />
                <label :for="transactionType">{{ transactionType }}</label>
            </div>
        </div>
        <div>
            <div class="label">State</div>
            <div v-for="state of filtersBase.state">
                <input type="checkbox" :id="state" :value="state" v-model="filters.state" />
                <label :for="state">{{ state }}</label>
            </div>
        </div>
        <div>
            <div class="label">Is Taxed</div>
            <div>
                <input type="checkbox" id="True" :value="true" v-model="filters.isTaxed" />
                <label for="True">True</label>
            </div>

            <div>
                <input type="checkbox" id="False" :value="false" v-model="filters.isTaxed" />
                <label for="False">False</label>
            </div>
        </div>
        <div>
            <div class="label">Has Discount</div>
            <div>
                <input type="checkbox" id="True" :value="true" v-model="filters.hasDiscount" />
                <label for="True">True</label>
            </div>
            <div>
                <input type="checkbox" id="False" :value="false" v-model="filters.hasDiscount" />
                <label for="False">False</label>
            </div>
        </div>
    </div>
    <div v-for="transaction of transactions">
        <div v-if="passesTransactionLevelFilter(transaction)">
            <div class="data_table">
                <div style="display: flex; justify-content: space-between;" class="table_heading">
                    <div style="margin-right: 10px">
                        {{ transaction.type }}
                    </div>
                    <div style="margin-right: 10px">
                        {{ transaction.address.street }}, {{ transaction.address.city }},
                        {{ transaction.address.state }},
                        {{ transaction.address.postal_code }}
                    </div>
                </div>

                <div class="transaction-table transaction-table-header">
                    <div>Name</div>
                    <div>Price</div>
                    <div>Quantity</div>
                    <div>Currency</div>
                    <div>Discount</div>
                    <div>Discount Type</div>
                    <div>Final Price</div>
                    <div>Tax Rate</div>
                    <div>Tax Liability</div>
                </div>
                <div>
                  <div v-for="item of transaction.line_items" class="table-nth">
                    <div class="transaction-table" v-if="passesItemLevelFilter(item)">
                      <div>
                        {{ item.name }}
                      </div>
                      <div>
                        {{ item.price }}
                      </div>
                      <div>
                        {{ item.quantity }}
                      </div>
                      <div>
                        {{ item.currency }}
                      </div>
                      <div>
                        {{ item.discount ? item.discount : 'N/A' }}
                      </div>
                      <div>
                        {{ item.discount_type ? item.discount_type : 'N/A' }}
                      </div>
                      <div>
                        {{ item.finalPrice }}
                      </div>
                      <div>
                        {{ item.isTaxed ? `${item.taxRate}%` : 'N/A' }}
                      </div>
                      <div>
                        {{ item.taxAmount }}
                      </div>
                    </div>
                  </div>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import axios from "axios";
import { transactionsResponseJSON } from '../data/transaction'
export default {
    name: "Dashboard",
    mounted() {
        this.getTransactions();
    },
    data() {
        return {
            codeToStatemapping: {
                ny_us: "NY",
                ca_us: "CA",
                wa_us: "WA",
            },
            transactions: [],
            summaryData: {
                totalTransactions: 0,
                totalItems: 0,
                totalRevenue: 0,
                transactionsByState: {},
                transactionsByDebitOrCredit: {},
                totalTaxLiability: 0,
                totalTaxLiabilityByState: {},
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
            },
        };
    },
    computed: {
        filteredTransactions() {
            return this.transactions.filter((transaction) =>
                this.passesTransactionLevelFilter(transaction)
            );
        },
        filteredSummaryData() {
            const summary = {
                totalTransactions: 0,
                totalItems: 0,
                totalRevenue: 0,
                totalTaxLiability: 0,
                transactionsByState: {},
                totalTaxLiabilityByState: {},
            };

            this.filteredTransactions.forEach((transaction) => {
                summary.totalTransactions++;
                transaction.line_items.forEach((item) => {
                    if (this.passesItemLevelFilter(item)) {
                        summary.totalItems++;
                        summary.totalRevenue += item.finalPrice;
                        summary.totalTaxLiability += item.taxAmount;

                        const state = transaction.address.state;
                        if (!(state in summary.transactionsByState)) {
                            summary.transactionsByState;
                            summary.transactionsByState[state] = 0;
                        }
                        summary.transactionsByState[state]++;

                        if (!(state in summary.totalTaxLiabilityByState)) {
                            summary.totalTaxLiabilityByState[state] = 0;
                        }
                        summary.totalTaxLiabilityByState[state] += item.taxAmount;
                    }
                });
            });

            return summary;
        },
        formatCurrency() {
            return (value) => {
                return value.toLocaleString('en-US', {
                    style: 'currency',
                    currency: 'USD'
                });
            };
        }
    },
    methods: {
        resetAllData() {
            this.summaryData = {
                totalTransactions: 0,
                totalItems: 0,
                totalRevenue: 0,
                transactionsByState: {},
                transactionsByDebitOrCredit: {},
                totalTaxLiability: 0,
                totalTaxLiabilityByState: {},
            };
            this.transactions = [];
        },
        passesTransactionLevelFilter(transaction) {
            if (!this.filters.transactionType.includes(transaction.type)) {
                return false;
            }
            if (!this.filters.state.includes(transaction.address.state)) {
                return false;
            }
            return true;
        },
        passesItemLevelFilter(item) {
            if (!this.filters.isTaxed.includes(item.isTaxed)) {
                return false;
            }
            // Special Case for discount
            if (!item.discount && !this.filters.hasDiscount.includes(false)) {
                return false;
            }
            if (item.discount && !this.filters.hasDiscount.includes(true)) {
                return false;
            }
            return true;
        },
        addTransactionToSummaryData(transaction) {
            this.summaryData.totalItems += transaction.line_items.length;
            const state = transaction.address.state;
            if (!(state in this.summaryData.transactionsByState)) {
                this.summaryData.transactionsByState[state] = 0;
            }
            this.summaryData.transactionsByState[state] += 1;
        },
        addTaxAmountToSummaryData(state, taxAmount) {
            this.summaryData.totalTaxLiability += taxAmount;
            if (!(state in this.summaryData.totalTaxLiabilityByState)) {
                this.summaryData.totalTaxLiabilityByState[state] = 0;
            }
            this.summaryData.totalTaxLiabilityByState[state] += taxAmount;
        },
        addRevenueToSummaryData(finalPrice) {
            this.summaryData.totalRevenue += finalPrice;
        },
        addStateToFilters(state) {
            if (!this.filtersBase.state.includes(state)) {
                this.filtersBase.state.push(state);
            }
        },
        addTransactionTypeToFilters(transactionType) {
            if (!this.filtersBase.transactionType.includes(transactionType)) {
                this.filtersBase.transactionType.push(transactionType);
            }
        },
        enrichLineItemWithTaxInfo(state, taxMapping, lineItem) {
            const isTaxed = taxMapping[state].taxable_items.includes(lineItem.name);
            const totalPrice = lineItem.price * lineItem.quantity;
            let discountAmount = 0;
            if (lineItem.discount) {
                discountAmount = lineItem.discount_type == "percentage" ? totalPrice * (lineItem.discount / 100) : lineItem.discount;
            }
            const finalPrice = totalPrice - discountAmount;
            this.addRevenueToSummaryData(finalPrice);
            // Set new values
            lineItem.finalPrice = finalPrice;
            lineItem.isTaxed = isTaxed;
            lineItem.taxRate = taxMapping[state].rate;
            lineItem.taxAmount = isTaxed ? (finalPrice * taxMapping[state].rate) / 100 : 0;
            this.addTaxAmountToSummaryData(state, lineItem.taxAmount);
        },
        async getTransactions() {
            // Removed network request so we can deploy this as a github page
            // const url = "https://server.getsphere.com/tax_api/test_transactions";
            // const response = await axios.get(url, {
            //     headers: {
            //         Authorization: "Bearer <bearer>",
            //     },
            // });
            const transactions = transactionsResponseJSON.transactions;
            const taxAuthorities = transactionsResponseJSON.tax_authority;

            const taxMapping = {};
            for (const taxAuthority of taxAuthorities) {
              if (taxAuthority.name in this.codeToStatemapping) {
                const state = this.codeToStatemapping[taxAuthority.name];
                taxMapping[state] = taxAuthority;
              }
            }

            this.resetAllData();
            for (const transaction of transactions) {
                const state = transaction.address.state;
                this.addStateToFilters(state);
                this.addTransactionTypeToFilters(transaction.type);
                this.addTransactionToSummaryData(transaction);
                for (const lineItem of transaction.line_items) {
                    this.enrichLineItemWithTaxInfo(state, taxMapping, lineItem);
                }
                this.transactions.push(transaction);
            }
            // Now all transactions have their updated price
            console.log(transactions);
            // Set all filters to include everything we get from the incoming response
            this.filters = JSON.parse(JSON.stringify(this.filtersBase));
        },
    },
};
</script>
