<template>
  <div class="container my-5">
    <b-button variant="secondary" class="mb-4">Previous</b-button>

    <div class="d-flex flex-wrap align-items-start gap-4">
      <b-card class="flex-fill discount-card">
        <b-accordion id="discounts-accordion" flush>
          <b-accordion-item v-for="(accordionList, index) in accordionLists" :key="index" :title="accordionList.name"
            :disabled="accordionList.disabled" :visible="activeAccordionIndex === index">
            <template #title>
              <div class="d-flex justify-content-between align-items-center w-100 p">
                <h6 class="mb-0">{{ accordionList.name }}</h6>
              </div>
            </template>

            <div v-if="accordionList.name === 'Discount'">
              <div class="discount-table">
                <div class="d-flex justify-content-end align-items-center my-3 w-100">
                  <b-button variant="link" class="text-primary p-0" @click="showAddManualModal = true">
                    + Add manual discount
                  </b-button>
                </div>

                <table class="table align-middle">
                  <tbody>
                    <tr v-for="(discount, index) in discounts" :key="index">
                      <td class="text-nowrap">
                        <div class="d-flex justify-content-between align-items-center">
                          <span>{{ discount.name }}</span>
                          <span v-if="discount.editable" class="text-primary" role="button" title="Edit">
                            <i class="fa-solid fa-pen"></i>
                          </span>
                        </div>
                      </td>

                      <td class="text-muted">
                        {{ discount.value }} {{ discount.frequency }}
                      </td>

                      <td class="text-end">
                        <div v-if="!discount.manual" class="form-check form-switch d-flex justify-content-end">
                          <input class="custom-switch" type="checkbox" v-model="discount.active" />
                        </div>

                        <div v-else class="d-flex justify-content-end gap-4 text-primary align-items-center">
                          <i class="fa-solid fa-pen" role="button" title="Edit"
                            @click="editManualDiscount(discount)"></i>
                          <i class="fa-solid fa-trash me-3" role="button" title="Delete"
                            @click="openDeleteModal(discount)"></i>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>

              <div class="d-flex justify-content-between mt-3">
                <b-button variant="link" class="text-primary fw-semibold" :disabled="activeAccordionIndex === 0"
                  @click="goToPreviousAccordion">
                  Previous
                </b-button>

                <b-button variant="primary" class="px-4" :disabled="activeAccordionIndex === accordionLists.length - 1"
                  @click="goToNextAccordion">
                  Next
                </b-button>
              </div>
            </div>

            <div v-else class="d-flex justify-content-between align-items-center">
              <span class="text-muted">{{ accordionList.name }} details</span>
            </div>
          </b-accordion-item>
        </b-accordion>
      </b-card>

      <!-- Overview -->
      <b-card class="overview-card">
        <div class="p-3">
          <div class="text-center">
            <img alt="product" src="../assets/product.png" />
          </div>
          <h6 class="fs-4 fw-bold my-3 text-body-tertiary">Overview</h6>

          <div class="d-flex justify-content-between pb-2">
            <span>Webasto Pure II laadpaal type 2</span>
            <span>€ {{ baseOneTimePrice.toFixed(2) }}</span>
          </div>

          <div class="d-flex justify-content-between mt-2">
            <span>Maandelijkse prijs</span>
            <span>€ {{ baseMonthlyPrice.toFixed(2) }}</span>
          </div>
        </div>

        <div class="d-flex justify-content-between mt-3 p-3 fw-semibold bg-light-subtle">
          <span>Eventually per month excl. btw</span>
          <span>€ {{ finalMonthlyPrice.toFixed(2) }}</span>
        </div>

        <div class="mt-3 bg-light-subtle p-3">
          <div class="d-flex justify-content-between">
            <span>Subtotal onetime costs excl. btw</span>
            <span>€ {{ baseOneTimePrice.toFixed(2) }}</span>
          </div>

          <div v-for="(discount, index) in activeDiscounts" :key="index" class="d-flex justify-content-between pt-2">
            <span><i>{{ discount.name }}</i></span>
            <span>- {{ discount.value }}</span>
          </div>

          <div class="d-flex justify-content-between pt-2 fw-semibold">
            <span>Onetime costs excl. btw</span>
            <span>€ {{ finalOneTimePrice.toFixed(2) }}</span>
          </div>
        </div>
      </b-card>
    </div>

    <!-- Discount Modal -->
    <b-modal v-model="showAddManualModal" title="Add manual discount" hide-footer hide-header-close size="lg">
      <div>
        <p class="mb-3">
          For which price do you calculate the discount?
        </p>
        <div class="d-flex gap-3 mb-4">
          <b-button :variant="priceType === 'one-time' ? 'primary' : 'outline-secondary'"
            @click="priceType = 'one-time'" class="fw-bold calc-btn">
            One time price
            <i :class="['text-light ms-3 fa', priceType === 'one-time' ? 'fa-check-circle' : 'fa-circle']"></i>
          </b-button>
          <b-button :variant="priceType === 'monthly' ? 'primary' : 'outline-secondary'" @click="priceType = 'monthly'"
            class="fw-bold calc-btn">
            Monthly price
            <i :class="['text-light ms-3 fa', priceType === 'monthly' ? 'fa-check-circle' : 'fa-circle']"></i>
          </b-button>
        </div>

        <div class="mb-3">
          <label class="form-label">Discount</label>
          <div class="input-group">
            <select class="form-select" v-model="manualDiscount.type" style="max-width: 100px;">
              <option value="%">%</option>
              <option value="€">€</option>
            </select>
            <input type="number" class="form-control" v-model="manualDiscount.value" />
          </div>
        </div>

        <div class="mb-3">
          <label class="form-label">Duration</label>
          <input type="text" class="form-control" placeholder="Number of months" v-model="manualDiscount.duration" />
        </div>

        <div class="mb-4">
          <label class="form-label">Description</label>
          <input type="text" class="form-control" v-model="manualDiscount.description" />
        </div>

        <div class="d-flex justify-content-between gap-3">
          <b-button variant="link" class="text-primary" @click="cancelManualDiscount">Cancel</b-button>
          <b-button variant="primary" @click="addManualDiscount">Add</b-button>
        </div>
      </div>
    </b-modal>

    <!-- Delete modal -->
    <b-modal v-model="showDeleteModal" title="Delete discount" hide-footer centered>
      <p class="text-muted mb-4">
        Are you sure you want to delete this discount?
      </p>

      <div class="text-end">
        <b-button variant="danger" class="px-4" @click="confirmDelete">
          Delete discount
        </b-button>
      </div>
    </b-modal>
  </div>
</template>

<script>
import {
  BCard,
  BAccordion,
  BAccordionItem,
  BButton,
  BModal
} from 'bootstrap-vue-next'

export default {
  name: 'DiscountMain',
  components: {
    BCard,
    BAccordion,
    BAccordionItem,
    BButton,
    BModal
  },
  data() {
    return {
      baseOneTimePrice: 1000,
      baseMonthlyPrice: 10,
      discounts: [
        { name: 'Discount name', value: '€ 250,00', frequency: 'one time', active: true },
        { name: 'Discount name1', value: '5 %', frequency: 'one time', active: false, },
        { name: 'Discount name2', value: '15 %', frequency: 'one time', active: false, editable: true },
        { name: 'Discount name2', value: '20 %', frequency: 'monthly', active: false, editable: true },
        { name: 'Manual discount', value: '25 %', frequency: 'monthly', active: false, manual: true }
      ],
      showAddManualModal: false,
      priceType: 'monthly',
      manualDiscount: { type: '%', value: '', duration: '', description: '' },
      accordionLists: [
        { name: 'Discount', active: true, disabled: false },
        { name: 'Klantgegevens', active: false, disabled: true },
        { name: 'Productgegevens', active: false, disabled: true },
        { name: 'Checkout', active: false, disabled: true }
      ],
      activeAccordionIndex: 0,
      showDeleteModal: false,
      discountToDelete: null
    }
  },
  computed: {
    activeDiscounts() {
      return this.discounts.filter(d => d.active)
    },

    oneTimeDiscountTotal() {
      let total = 0
      this.activeDiscounts
        .filter(d => d.frequency === 'one time')
        .forEach(d => {
          const numericValue = this.parseDiscountValue(d.value)
          total += d.value.includes('%')
            ? (this.baseOneTimePrice * numericValue / 100)
            : numericValue
        })
      return total
    },


    monthlyDiscountTotal() {
      let total = 0
      this.activeDiscounts
        .filter(d => d.frequency === 'monthly')
        .forEach(d => {
          const numericValue = this.parseDiscountValue(d.value)
          total += d.value.includes('%')
            ? (this.baseMonthlyPrice * numericValue / 100)
            : numericValue
        })
      return total
    },

    finalOneTimePrice() {
      return this.baseOneTimePrice - this.oneTimeDiscountTotal
    },

    finalMonthlyPrice() {
      return this.baseMonthlyPrice - this.monthlyDiscountTotal
    },

    monthlyBreakdown() {
      let first3Months = this.finalMonthlyPrice
      let remainingMonths = this.baseMonthlyPrice

      const monthlyWithDuration = this.activeDiscounts.find(
        d => d.frequency === 'monthly' && d.duration
      )

      if (monthlyWithDuration) {
        const discountValue = this.parseDiscountValue(monthlyWithDuration.value)
        const discountAmount = monthlyWithDuration.value.includes('%')
          ? (this.baseMonthlyPrice * discountValue / 100)
          : discountValue

        first3Months = this.baseMonthlyPrice - discountAmount
        remainingMonths = this.baseMonthlyPrice
      }

      return {
        first3Months,
        remainingMonths
      }
    }
  },
  methods: {
    parseDiscountValue(value) {
      if (!value) return 0
      const normalized = String(value).replace(',', '.')
      const numericOnly = normalized.replace(/[^\d.-]/g, '')
      return parseFloat(numericOnly) || 0
    },
    goToNextAccordion() {
      if (this.activeAccordionIndex < this.accordionLists.length - 1) {
        this.activeAccordionIndex++
      }
    },
    goToPreviousAccordion() {
      if (this.activeAccordionIndex > 0) {
        this.activeAccordionIndex--
      }
    },
    editManualDiscount(discount) {
      alert(`Editing ${discount.name}`)
    },
    openDeleteModal(discount) {
      this.discountToDelete = discount;
      this.showDeleteModal = true;
    },
    confirmDelete() {
      if (this.discountToDelete) {
        this.discounts = this.discounts.filter(
          (d) => d !== this.discountToDelete
        );
      }
      this.showDeleteModal = false;
      this.discountToDelete = null;
    },
    addManualDiscount() {
      this.discounts.push({
        name: this.manualDiscount.description || 'Manual discount',
        value: `${this.manualDiscount.value} ${this.manualDiscount.type}`,
        frequency:
          this.priceType === 'monthly' ? 'monthly' : 'one time',
        active: false,
        manual: true
      })
      this.cancelManualDiscount()
    },
    cancelManualDiscount() {
      this.manualDiscount = { type: '%', value: '', duration: '', description: '' }
      this.showAddManualModal = false
    }
  }
}
</script>

<style scoped lang="scss">
.table {
  border-collapse: separate;
  border-spacing: 0 0.5rem;
}

.table td {
  padding: 0.75rem 1rem;
  vertical-align: middle;
}

.form-check-input {
  width: 2.5em;
  height: 1.3em;
}

.card {
  border-radius: 0;
  border: none !important;
  box-shadow: none !important;
}

::v-deep {
  .accordion-button {
    padding: 12px 16px;

    &.collapsed {
      background-color: #CACACA;
      color: #767676;
    }

    &:not(.collapsed) {
      background-color: #26B7CD;
      color: #ffffff;
    }

    &::after {
      display: none !important;
    }

    &[disabled],
    &:disabled {
      opacity: 0.6;
      cursor: not-allowed;
      background-color: #f5f5f5 !important;
      color: #999 !important;
    }
  }

  .card-body {
    padding: 0;
  }

}

.text-primary {
  color: #26B7CD !important;
}

.btn {
  height: 40px !important;
  padding: 9px 16px !important;
  border-radius: 0 !important;
}

.btn-primary {
  --bs-btn-bg: #26B7CD;
  --bs-btn-border-color: #26B7CD;
  --bs-btn-hover-bg: #177e8e;
  --bs-btn-hover-border-color: #177e8e;
  --bs-btn-active-bg: #177e8e;
  --bs-btn-active-border-color: #177e8e;
  --bs-btn-disabled-bg: #7eb6bf;
  --bs-btn-disabled-border-color: #7eb6bf;
}

.btn-secondary {
  --bs-btn-bg: #767676;
}

.btn-link {
  --bs-btn-color: #26B7CD;
  text-decoration: none !important;
}

.btn-danger {
  --bs-btn-bg: #CC4B37;
}

.btn-outline-secondary {
  background-color: #EAEAEA;
  --bs-btn-color: #767676;
  --bs-btn-border-color: #EAEAEA;
  --bs-btn-hover-color: #767676;
  --bs-btn-hover-bg: #EAEAEA;
  --bs-btn-active-bg: #EAEAEA;
  --bs-btn-hover-border-color: #EAEAEA;
  --bs-btn-active-border-color: #EAEAEA;
  --bs-btn-disabled-color: #767676;
  --bs-btn-disabled-bg: #EAEAEA;
  --bs-btn-disabled-border-color: #EAEAEA;
}

.calc-btn {
  border-radius: 10px !important;
  height: 66px !important;
}

.bg-light-subtle {
  background-color: #EDF6FB !important;
}

.discount-card {
  min-width: 770px;
}

.overview-card {
  max-width: 350px;
  min-width: 300px;
}

@media (max-width: 1024px) {

  .discount-card {
    width: 100%;
    min-width: 100%;
  }

  .overview-card {
    width: 100%;
    max-width: 100%;
  }
}

.input-group:not(.has-validation)> :not(:last-child):not(.dropdown-toggle):not(.dropdown-menu):not(.form-floating),
.input-group:not(.has-validation)>.dropdown-toggle:nth-last-child(n+3),
.input-group:not(.has-validation)>.form-floating:not(:last-child)>.form-control,
.input-group:not(.has-validation)>.form-floating:not(:last-child)>.form-select {
  border-radius: 0 !important;
}

.form-control,
.input-group> :not(:first-child):not(.dropdown-menu):not(.valid-tooltip):not(.valid-feedback):not(.invalid-tooltip):not(.invalid-feedback) {
  border-radius: 0 !important;
}

.custom-switch {
  appearance: none;
  -webkit-appearance: none;
  width: 70px;
  height: 35px;
  background-color: #CACACA;
  position: relative;
  cursor: pointer;
  outline: none;
  transition: background-color 0.3s ease;
  border: 2px solid transparent;
}

.custom-switch::before {
  content: "";
  position: absolute;
  top: 3px;
  left: 3px;
  width: 27px;
  height: 27px;
  background-color: #FFFFFF;
  transition: transform 0.3s ease;
}

.custom-switch:checked {
  background-color: #26B7CD;
}

.custom-switch:checked::before {
  transform: translateX(34px);
}
</style>
