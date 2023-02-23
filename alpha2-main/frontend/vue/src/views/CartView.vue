<template>
  <div id="app">
    <div id="top-wrapper">
          <font-awesome-icon icon="fa-solid fa-chevron-left" onclick=""/>
          <span>알파카트(일괄매수)</span>
      </div>
      <div class="scrollable" ref="scrollable" v-on:scroll="handleScroll">
        <table class="cart-table">
            <thead>
              <tr>
                  <th>선택</th>
                  <th>종목명</th>
                  <th>매수가</th>
                  <th>수량</th>
              </tr>
            </thead>
            <tbody>
            <tr v-for="(stock, index) in stocks" :key="index">
                <td><input type="checkbox" v-model="stock.checked" @click="stockClick(stock)"></td>
                <td>{{ stock.name }}</td>
                <td>{{ stock.price }}</td>
                <td>
                  <div class="display-flex">
                    <button type="button" @click="decrementQuantity(stock)">-</button>
                    {{ stock.quantity }}
                    <button type="button" @click="incrementQuantity(stock)">+</button>
                  </div>
                </td>
            </tr>
            </tbody>
        </table>
      </div>
      <div class="tabs">
        <div class="tab" @click="selectedTab = 'first'" :class="{ 'active': selectedTab === 'first' }">주식<br>포트폴리오</div>
        <div class="tab" @click="selectedTab = 'sec'" :class="{ 'active': selectedTab === 'sec' }">배당<br>포트폴리오</div>
        <div class="tab" @click="selectedTab = 'third'" :class="{ 'active': selectedTab === 'third' }">총자산<br>포트폴리오</div>
      </div>

      <div v-if="selectedTab == 'first'">
        <div class="al-center">
            <div class="display-inblock">
            <Pie :data="chartData" :options="options"/>
          </div>
          <div class="display-inblock-width">
            <b>기대수익률</b><br>
            {{ percent }}%<br><br>
            <b>위험</b><br>
            {{ danger }}%<br><br>
          </div>
        </div>
        <div class="display-flex m-top">
          <div><span id="csps">합계: {{checkedStockPricesSum}} 원</span></div>
          <button id="buy" type="button" @click="modal">매수</button>
        </div>
      </div>
      
      <div v-else-if="selectedTab == 'sec'">
        <Bar :data="divs" :options="divs.options" ref="barChart"/>
      </div>
      
      <div v-else-if="selectedTab == 'third'">
        <div style="display:inline-block; width:150px">
          <b>보유 포트폴리오</b>
          <Pie :data="chartData" :options="options"/>
        </div>
        <div style="display:inline-block; width:150px">
          <b>목표 포트폴리오</b>
          <Pie :data="darts" :options="options"/>
        </div>
      </div>            
  </div>
</template>

<script>
import axios from 'axios';
import dividend from "@/assets/dividend_3.json"
import stockReturn from "@/assets/stock_return_data.json"
import corr from "@/assets/corr.json"
import { Chart as ChartJS, ArcElement, Tooltip, Legend,Title,BarElement,CategoryScale,LinearScale } from 'chart.js'
import { Pie, Bar } from 'vue-chartjs'

ChartJS.register(ArcElement, Tooltip, Legend)
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)
export default {
  name: 'App',
  components: {
    Pie, Bar
  },
  data() {
    return {
      percent: '',
      danger: '',
      corr: corr,
      stockReturn: stockReturn,
      dividend: dividend,
      categoryList: ['주식포트폴리오'],
      stocks: [
      { name: "신한지주", code: "055550", price: "39,050", quantity: 1, checked: false },
      { name: "LG전자", code: "066570", price: "113,000", quantity: 1, checked: false },
      { name: "현대차", code: "005380", price: "179,200", quantity: 1, checked: false },
      { name: "삼성전자", code: "005930", price: "62,600", quantity: 1, checked: false },
      { name: "대우중공업", code: "042670", price: "8,920", quantity: 1, checked: false },
      { name: "SK", code: "034730", price: "183,600", quantity: 1, checked: false },
      { name: "KT", code: "030200", price: "32,100", quantity: 1, checked: false },
      { name: "네이버", code: "035420", price: "213,000", quantity: 1, checked: false },
      ],
      selectedTab: "first",
      divs:{
        labels: [ '01', '02', '03','04', '05', '06','07', '08', '09','10', '11', '12'],
        datasets: [
          {
            label:'배당금',
            backgroundColor: 'red',
            data: [400, 200, 1200, 390, 100, 4000, 390, 800, 3000, 200, 120, 11000],
          },
          {
            label:'추가',
            backgroundColor: '#f87979',
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
          },
        ],
        options: {
          responsive: true,
            scales: {
              x: {
                stacked: true,
              },
              y: {
                stacked: true
              }
    }
    },},
      darts:{
        labels:['주식','채권','실물자산','가상화폐'],
        datasets:[
          {
            backgroundColor: ['#41B883', '#E46651', '#00D8FF', '#DD1B16'],
            data:[]
          }
        ]
      }
    }
  },
  computed:{
      checkedStocks() {
        if(this.stocks){
          return this.stocks.filter(stock => stock.checked);
        }
        return [];

  },
  chartData() {
    const checkedStocks = this.checkedStocks;
    const checkedStockNames = checkedStocks.map(stock => stock.name);
    const checkedStockPrices = checkedStocks.map(stock => Number(stock.price.replace(',', ''))*stock.quantity);
    const sum = checkedStockPrices.reduce((acc, val) => acc + val, 0);
    this.checkedStockPricesSum = sum;
    function getKeyByValue(obj, value) {
      return Object.keys(obj).find(key => obj[key] === value);
    }
    function getValueByKey(obj, key) {
      return obj[key];
    }
    const codeData = checkedStocks.map(data => getValueByKey(data,'code'))
    console.log(checkedStockPrices)
    const filteredData = checkedStocks.map(checkedStock => getKeyByValue(this.dividend.stock_code, checkedStock.code));
    console.log(filteredData)
    const rate = filteredData.map(data => getValueByKey(this.dividend.div_rate, data))
    console.log(rate)
    const count = filteredData.map(data => getValueByKey(this.dividend.div_count, data))
    console.log(count)
    const month = filteredData.map(data => getValueByKey(this.dividend.div_months, data))
    console.log(month)

    const divdata = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    for (let s=0; s<checkedStockPrices.length; s++){
      for (let i=0; i<month[s].map(str => Number(str)).length; i++){
        divdata[month[s][i]-1] += Number(checkedStockPrices[s])*(Number(rate[s])/(Number(count[s])*100))
      }
    }
    
    this.divs.datasets[1].data = divdata
     //percent
     const ret = checkedStocks.map(checkedStock => getValueByKey(this.stockReturn, checkedStock.code));
    console.log(ret)
    const ret2 = ret.map(data => getValueByKey(data,'expected return'))
    var retsum = 0
    for (let s=0; s<ret2.length; s++){
      retsum += Number(checkedStockPrices[s]/sum)*ret2[s]
    }
    this.percent = retsum.toFixed(3)
    //danger
    function transpose(matrix) {
      return matrix[0].map((col, i) => matrix.map(row => row[i]));
    }
    var weights = []
    const cor = checkedStocks.map(checkedStock => getValueByKey(this.corr, checkedStock.code));
    var corrmat = []
    var std_devs = ret.map(data => getValueByKey(data,'annual volatility'))
    for (let s=0; s<checkedStockPrices.length; s++){
      weights[s]=checkedStockPrices[s]/sum
    }
    console.log(cor)
    for (let s=0; s<codeData.length; s++){
      for (let k=0; k<codeData.length; k++){
        // corrmat.push(cor[s].map(co => getValueByKey(cor[s], cor[k])))
      }
    }
    console.log(corrmat)
    var result = []
    for (var i = 0; i < weights.length; i++) {
      result.push(weights[i] *std_devs[i]*std_devs[i]*100);
    }
    console.log(result)
    
    return {
      
      labels: checkedStockNames,
      datasets: [
        {
          backgroundColor: ['#41B883', '#E46651', '#00D8FF', '#DD1B16'],
          data: checkedStockPrices,
        }
      ],
      checkedStockPricesSum: 0,
    };
    }
  },
  methods: {
    updateDiv() {
      this.checkedStocks
      },
    updatePort() {
            const headers = { 'Authorization': `JWT ${localStorage.getItem('access_token')}` };
            axios
                .get("http://34.64.108.15:8000/api/portfolio/", { headers })
                  .then(res => {
                    this.darts.datasets[0].data = [res.data.results[0].stock,res.data.results[0].bond,res.data.results[0].real_asset,res.data.results[0].crypto];
                  })
                  .catch(error => {
                    console.log(error);
                  });
    },
    updateStock() {
      const headers = { 'Authorization': `JWT ${localStorage.getItem('access_token')}` };
            axios
                .get("http://34.64.108.15:8000/api/stock/cart/", { headers })
                  .then(res => {
                    this.stocks.push({name: res.data.results.stock_name, code: res.data.results.stock_code, price: res.data.results.stock_cp, quantity: res.data.results.stock_pg, checked: false});
                  })
                  .catch(error => {
                    console.log(error);
                  });
    },
    stockClick(stock) {
      stock.checked = true;
    },
    incrementQuantity(stock) {
      stock.quantity += 1;
    },
    decrementQuantity(stock) {
      if (stock.quantity > 1){
        stock.quantity -= 1;
      }
    },
    handleScroll() {
        const scrollable = this.$refs.scrollable;
        if (scrollable.scrollTop + scrollable.offsetHeight >= scrollable.scrollHeight) {
          console.log('맨 아래 도달');
        }
      }
  },
  mounted() {
    this.updateDiv()
    this.updatePort()
    this.updateStock()
  }
}
</script>

<style scoped>

/* top-wrapper */
#top-wrapper {
  text-align: left;
  height: 30px;
  font-size: 15px;
  padding-left: 5px;
}

#top-wrapper span {
  margin-left: 10px;
}

.display-absolute {
  display: absolute;
  left: 0;
}
.m-top {
  margin-top: 5px;
}
.p-top {
  padding-top: 5px;
}

.scrollable {
    width: 100%px;
    height: 170px;
    overflow-y: scroll;
  }
 .tabs {
    display: flex;
    justify-content: space-between;
    border-bottom: 2px solid #ccc;
    border-top: 2px solid #ccc;
    font-size: 12px;
  }
  
  .tab {
    padding-left: 10px;
    padding-right: 10px;
    cursor: pointer;
  }
  
  .tab.active {
    background-color: #ccc;
    font-weight: 900;
    text-decoration : underline;
  }
  
  .tab-content {
    margin-top: 10px;
  }
  .cart-table {
    width: 100%;
    font-size: 13px;;
  }
  tr {
    background-color: rgba(236, 236, 236, 0);
  }
  td {
    text-align: left;
    padding: 4px;
    padding-left: 10px;
  }
  th {
    background-color: #cccccc;
    color: black;
    padding: 4px;
    font-size: 13px
  }
  input[type=checkbox] {
      cursor: pointer;
  }
  .display-flex {
    display: flex;
    justify-content: space-around;
    vertical-align: bottom;
  }

  .display-inblock {
    display: inline-block;
    width: 200px;
    height: 200px;
    vertical-align: top;
  }
  .display-inblock-width {
    display: inline-block;
    width: 80px;
    height: 40px;
    margin-top: 60px;
    vertical-align: top;
  }

  button img{
    width: 100px;
    height: 30px;
    object-fit: cover;
  }
  #chart {
    display: inline-block;
  }

  .al-center {
    text-align: center;
  }

  #csps {
    font-size: 20px;
  }

  #buy {
    padding: 5px;
    margin: 2px;
    width: 100px;
    text-align: center;
    border-radius: 5px;
    background-color:#117FE3;   
    color: white;
    font-weight: bold;
  }
</style>