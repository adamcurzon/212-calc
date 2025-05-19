<template>
  <main>
    <h1>212 Calculator</h1>
    <div class="holder">
      <div class="input">
        <label for="buyPrice">Buy Price</label>
        <div class="input-row">
          <div class="input-icon">$</div>
          <input type="number" step="0.01" v-model="form.buyPrice" id="buyPrice" placeholder="Buy Price" />
        </div>
        <label for="sellPrice">Sell Price</label>
        <div class="input-row">
          <div class="input-icon">$</div>
          <input type="number" step="0.01" v-model="form.sellPrice" id="sellPrice" placeholder="Sell Price" />
        </div>
        <label for="stopLoss">Stop Loss</label>
        <div class="input-row">
          <div class="input-icon">$</div>
          <input type="number" step="0.01" v-model="form.stopLoss" id="stopLoss" placeholder="Stop Loss" />
        </div>
        <label for="balance">Balance</label>
        <div class="input-row">
          <div class="input-icon">£</div>
          <input type="number" step="0.01" v-model="form.balance" id="balance" placeholder="Balance" />
        </div>
        <div class="buttons">
          <button @click="openPopup">
            Edit Fees
          </button>
          <button @click="resetForm">
            Reset
          </button>
        </div>
      </div>

      <div class="output">
        <div class="profit-calc">
          <label>Profit Calculation</label>
          <table>
            <tbody>
              <tr>
                <th>Fee to Buy</th>
                <td class="red">-{{ feeToBuy }} USD</td>
              </tr>
              <tr>
                <th>Shares Bought</th>
                <td>{{ numSharesBought }} Shares</td>
              </tr>
              <tr>
                <th>Fee to Sell</th>
                <td class="red">-{{ feeToSell }} USD</td>
              </tr>
              <tr>
                <th>Balance</th>
                <td>{{ gbpBalanceAfterSell }} GBP</td>
              </tr>
              <tr>
                <th>Profit</th>
                <td :class="{ 'red': this.profit <= 0, 'green': this.profit > 0 }">
                  {{ profit }} GBP
                  ({{ profitPercentage }}%)
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="loss-calc">
          <label>Stop Loss Calculation</label>
          <table>
            <tbody>
              <tr>
                <th>Fee to Buy</th>
                <td class="red">-{{ feeToBuy }} USD</td>
              </tr>
              <tr>
                <th>Shares Bought</th>
                <td>{{ numSharesBought }} Shares</td>
              </tr>
              <tr>
                <th>Fee to Sell</th>
                <td class="red">-{{ feeToSellLoss }} USD</td>
              </tr>
              <tr>
                <th>Balance</th>
                <td>{{ gbpBalanceAfterSellLoss }} GBP</td>
              </tr>
              <tr>
                <th>Loss</th>
                <td :class="{ 'red': this.loss <= 0, 'green': this.loss > 0 }">
                  {{ loss }} GBP
                  ({{ lossPercentage }}%)
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div id="popup" :class="{ 'active': popupOpen }">
      <div class="modal">
        <div class="input">
          <button @click="closePopup">Close Fees</button>
          <label for="exchangeRate">Exchange Rate</label>
          <div class="input-row">
            <div class="input-icon">%</div>
            <input type="number" step="0.01" v-model="form.exchangeRate" id="exchangeRate"
              placeholder="Exchange Rate" />
          </div>
          <label for="fxFee">FX Fee</label>
          <div class="input-row">
            <div class="input-icon">%</div>
            <input type="number" step="0.01" v-model="form.fxFee" id="fxFee" placeholder="FX Fee" />
          </div>
          <label for="finraFee">Finra Fee</label>
          <div class="input-row">
            <div class="input-icon">%</div>
            <input type="number" step="0.01" v-model="form.finraFee" id="finraFee" placeholder="Finra Fee" />
          </div>
          <label for="secFee">SEC Fee</label>
          <div class="input-row">
            <div class="input-icon">%</div>
            <input type="number" step="0.01" v-model="form.secFee" id="secFee" placeholder="SEC Fee" />
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      form: {
        buyPrice: '10.00',
        sellPrice: '11.00',
        stopLoss: '9.00',
        balance: '10000.00',
        exchangeRate: '1.34',
        fxFee: '0.15',
        finraFee: '0.000166',
        secFee: '0.00278',
      },
      popupOpen: false,
    }
  },
  methods: {
    convertGBPtoUSD(amount) {
      const exchangeRate = parseFloat(this.form.exchangeRate);
      return (amount * exchangeRate).toFixed(2);
    },
    convertUSDtoGBP(amount) {
      const exchangeRate = parseFloat(this.form.exchangeRate);
      return (amount / exchangeRate).toFixed(2);
    },
    calcFXFee(amount) {
      const fxFee = parseFloat(this.form.fxFee) / 100;
      return (amount * fxFee).toFixed(2);
    },
    finraFee(numOfShares) {
      return (numOfShares * this.form.finraFee).toFixed(4);
    },
    secFee(numOfShares, sellPrice) {
      return (numOfShares * sellPrice * (this.form.secFee / 100)).toFixed(4);
    },
    closePopup() {
      this.popupOpen = false;
    },
    openPopup() {
      this.popupOpen = true;
    },
    resetForm() {
      this.form = {
        buyPrice: '10.00',
        sellPrice: '11.00',
        stopLoss: '9.00',
        balance: '10000.00',
        exchangeRate: '1.34',
        fxFee: '0.15',
        finraFee: '0.000166',
        secFee: '0.00278',
      }
      localStorage.removeItem('form');
    }
  },
  computed: {
    usdBalance() {
      return parseFloat(this.convertGBPtoUSD(this.form.balance));
    },
    feeToBuy() {
      return parseFloat(this.calcFXFee(this.usdBalance));
    },
    numSharesBought() {
      return parseFloat(parseFloat(this.usdBalance - this.feeToBuy) / parseFloat(this.form.buyPrice)).toFixed(2);
    },
    feeToSell() {
      const fxFee = parseFloat(this.calcFXFee(this.numSharesBought * this.form.sellPrice));
      const finraFee = parseFloat(this.finraFee(this.numSharesBought));
      const secFee = parseFloat(this.secFee(this.numSharesBought, this.form.sellPrice));
      return parseFloat(fxFee + finraFee + secFee).toFixed(2);
    },
    usdBalanceAfterSell() {
      return parseFloat(parseFloat(this.numSharesBought * this.form.sellPrice) - this.feeToSell).toFixed(2);
    },
    gbpBalanceAfterSell() {
      return parseFloat(this.convertUSDtoGBP(this.usdBalanceAfterSell)).toFixed(2);
    },
    profit() {
      return parseFloat(this.gbpBalanceAfterSell - this.form.balance).toFixed(2);
    },
    profitPercentage() {
      return parseFloat((this.profit / this.form.balance) * 100).toFixed(2);
    },
    feeToSellLoss() {
      const fxFee = parseFloat(this.calcFXFee(this.numSharesBought * this.form.stopLoss));
      const finraFee = parseFloat(this.finraFee(this.numSharesBought));
      const secFee = parseFloat(this.secFee(this.numSharesBought, this.form.stopLoss));
      return parseFloat(fxFee + finraFee + secFee).toFixed(2);
    },
    gbpBalanceAfterSellLoss() {
      return parseFloat(this.convertUSDtoGBP(parseFloat(this.numSharesBought * this.form.stopLoss) - this.feeToSellLoss)).toFixed(2);
    },
    loss() {
      return parseFloat(this.gbpBalanceAfterSellLoss - this.form.balance).toFixed(2);
    },
    lossPercentage() {
      return parseFloat((this.loss / this.form.balance) * 100).toFixed(2);
    },
  },
  created() {
    const savedForm = localStorage.getItem('form');
    if (savedForm) {
      this.form = JSON.parse(savedForm);
    }
  },
  watch: {
    form: {
      handler(newVal) {
        localStorage.setItem('form', JSON.stringify(newVal));
      },
      deep: true
    }
  }
}
</script>

<style>
body {
  background-color: #1a2631;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #dddddd;
}

main {
  padding-top: 50px;
}

h1 {
  color: white;
}

.holder {
  max-width: 768px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 20px;
  padding: 0px 10px
}

@media screen and (max-width: 768px) {
  .holder {
    grid-template-columns: 1fr;
    gap: 0px;
  }

  main {
    padding-bottom: 50px;
    padding-top: 0px !important;
  }

  .profit-calc,
  .loss-calc {
    padding: 20px 10px;
  }

  .output {
    grid-row: 1;
    margin-top: 0px !important;
  }

  .input {
    grid-row: 2;
  }
}

label {
  display: block;
  width: 100%;
  text-align: left;
  font-weight: 600;
}

.input,
.output {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  flex-direction: column;
  margin: 0 auto;
}

.output {
  display: grid;
  gap: 22px;
  margin-top: 32px;
  grid-template-rows: 1fr 1fr;
  grid-template-columns: 100%;
}

.profit-calc,
.loss-calc {
  width: 100%;
  /* border: 1px solid #ccc; */
  background-color: #34495e;
  border-radius: 5px;
  padding: 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.profit-calc label,
.loss-calc label {
  text-align: center;
  color: white;
}

label {
  margin-top: 10px;
}

.input-row,
button {
  display: flex;
  align-items: center;
  border: 1px solid #2c3e50;
  padding: 10px;
  border-radius: 5px;
  background-color: #2c3e50;
  color: white;
}

button {
  align-content: center;
  justify-content: center;
  cursor: pointer;
  margin-top: 22px;
  text-align: center;
  font-size: 16px;
  font-weight: 600;
  color: #DDD;
}

button:hover {
  opacity: 0.8;
}

.input-icon {
  width: 30px;
}

.input-row:has(input:focus) {
  border: 1px solid white;
}

.input-row input {
  flex: 1;
  padding: 10px;
  margin-left: 10px;
  border: 0;
  outline: 0;
  text-align: right;
  background-color: transparent;
  color: white !important;
  font-size: 16px;
}

table {
  width: calc(100% - 40px);
}

table tr th {
  width: 50%;
  text-align: right;
  padding-right: 10px;
  font-weight: 500;
}

table tr td {
  width: 50%;
  text-align: left;
  padding-left: 10px;
}

table tr td.red {
  color: red;
}

table tr td.green {
  color: green;
}

table tr:nth-last-child(2) td,
table tr:nth-last-child(2) th {
  padding-bottom: 10px;
}


table tr:first-child td,
table tr:first-child th,
table tr:last-child td,
table tr:last-child th {
  border-top: 3px dotted #576e83;
  padding-top: 10px;
}

table tr:last-child td,
table tr:last-child th {
  font-weight: 700;
}

#popup {
  display: none !important;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background-color: #34495e;
  padding: 20px;
  border-radius: 5px;
  width: 90%;
  max-width: 500px;
}

#popup.active {
  display: flex !important;
}

.buttons button {
  width: 50%;
}

.buttons {
  width: 100%;
  display: flex;
  align-content: center;
  justify-content: center;
  gap: 20px;
}
</style>
