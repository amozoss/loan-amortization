<template>
  <div class="container">
    <div class="row">
      <label class="label">Principal : </label>
      <input class="input" type="text" placeholder="Loan amount plus fee(if any)" v-model="loanAmount">
      <label class="label">Interest rate : </label>
      <input class="input" type="text" placeholder="% per payment" v-model="interestRate">
      <label class="label">Number Of Payments(Months)</label>
      <input class="input" type="text" placeholder="Number of payments" v-model="paymentNumber">
    </div>
    <button class="button" @click="calc_payments" >
      Calculate
    </button>
    <div v-if="payments.length">
      <p>Monthly principal & interest: {{ monthlyPayment.toFixed(2) }}</p>
      <p>Total Interest: {{ totalInterest.toFixed(2) }}</p>
      <p>Total of all monthly payments: {{ totalPayment.toFixed(2) }}</p>
    </div>
    <table class="table" cellspacing="10" v-if="payments.length">
      <thead>
        <tr>
          <th>Number</th>
          <th>Payment</th>
          <th>Interest</th>
          <th>Principal</th>
          <th>Balance</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="payment in payments" :key="payment.count">
          <td>{{ payment.count }}</td>
          <td>{{ payment.payment.toFixed(2) }}</td>
          <td>{{ payment.interest.toFixed(2) }}</td>
          <td>{{ payment.principal.toFixed(2) }}</td>
          <td>{{ payment.balance.toFixed(2) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script type="text/javascript">
export default {
  mounted(){

  },
  data(){
    return {
      loanAmount: 300000,
      paymentNumber:360,
      interestRate:4.625,
      payments:[],
    }
  },
  methods:{
    reset(){
      this.loanAmount = '';
      this.paymentNumber = '';
      this.interestRate = '';
      this.monthlyPayment = '';
      this.payments=[];
    },
    calc_payment(p = this.loanAmount, n = this.paymentNumber, int = this.interestRate){
      // https://www.vertex42.com/ExcelArticles/amortization-calculation.html
      //            r(1 + r)^n
      // pmt = p * --------------
      //            (1 + r)^n - 1

      let r = int / 100 / 12
      let pmt    = p * (r * Math.pow((1 + r), n) / (Math.pow((1 + r), n) - 1));
      this.monthlyPayment  = pmt

      return pmt
    },

    calc_payments(e, balance = parseFloat(this.loanAmount), rate = this.interestRate){
      var count = 0;
      var payments = [];
      var totalPayment = 0;
      var totalInterest = 0;
      let payment = Number(this.calc_payment())
      do {
        count++
        var interest = (balance * (rate/12/100))
        var principal = (payment - interest)

        if (balance < payment) {
          principal = balance;
          payment   = (Number(interest) + Number(principal))
        }

        balance = balance - principal

        if (balance < 0) {
          principal = principal + balance;
          interest  = Number(interest) - balance;
          balance   = 0;
        }

        payments.push({
          balance: balance, payment: payment,
          interest: interest,
          principal: principal,
          count
        });

        totalPayment   = totalPayment + payment;
        totalInterest  = totalInterest + interest;

      } while (balance > 0);
      this.payments = payments;
      this.totalInterest = totalInterest;
      this.totalPayment = totalPayment;
    }
  }
}
</script>

<style type="text/css" scoped>
table{
  margin-top: 5%;
  border-top: 1px solid #ccc;
}
input {
  max-width: 100px;
}
.row {
  display: flex;
  flex-direction: column;
}
</style>
