<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <span v-if="error" style="font: bold;color: red"><h1>{{error}}</h1></span><br>
    <select v-model="country1" @change="changeCountry1">
      <option v-for="country in countries" :key="country.text" :value="country.text">
        {{ country.value }}
      </option>
    </select>
    <select v-model="country2" @change="changeCountry2">
      <option v-for="country in countries" :key="country.text" :value="country.text">
        {{ country.value }}
      </option>
    </select>
    <Line :data="data" :options="options" />
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import axios from 'axios'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

// import { Bar } from 'vue-chartjs'
import { Line } from 'vue-chartjs'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)


export default {
  name: 'App',
  components: {
    Line
  },
  methods:{

    extractColumn(arr, column) {
      return arr.map(x => x[column])
    },
    getCountryList(){

      let countries = ["Afghanistan","Albania","Algeria","Andorra","Angola","Antigua and Barbuda","Argentina","Armenia","Australia","Austria","Austrian Empire",
        "Azerbaijan","Baden","Bahamas, The","Bahrain","Bangladesh","Barbados","Bavaria","Belarus","Belgium","Belize","Benin (Dahomey)","Bolivia","Bosnia and Herzegovina","Botswana","Brazil","Brunei","Brunswick and Lüneburg","Bulgaria","Burkina Faso (Upper Volta)","Burma","Burundi","Cabo Verde","Cambodia","Cameroon","Canada","Cayman Islands, The","Central African Republic","Central American Federation","Chad","Chile","China","Colombia","Comoros","Congo Free State, The","Cook Islands","Costa Rica","Cote d’Ivoire (Ivory Coast)","Croatia","Cuba","Cyprus","Czechia","Czechoslovakia","Democratic Republic of the Congo","Denmark","Djibouti","Dominica","Dominican Republic","Duchy of Parma, The","East Germany (German Democratic Republic)","Ecuador","Egypt","El Salvador","Equatorial Guinea","Eritrea","Estonia","Eswatini","Ethiopia","Federal Government of Germany (1848-49)","Fiji","Finland",
        "France","Gabon","Gambia, The","Georgia","Germany","Ghana","Grand Duchy of Tuscany, The","Greece","Grenada","Guatemala","Guinea","Guinea-Bissau","Guyana","Haiti","Hanover","Hanseatic Republics","Hawaii","Hesse","Holy See","Honduras","Hungary","Iceland","India","Indonesia","Iran","Iraq","Ireland","Israel","Italy","Jamaica","Japan","Jordan","Kazakhstan","Kenya","Kingdom of Serbia/Yugoslavia","Kiribati","Korea","Kosovo","Kuwait","Kyrgyzstan","Laos","Latvia","Lebanon","Lesotho","Lew Chew (Loochoo)","Liberia","Libya","Liechtenstein","Lithuania","Luxembourg","Madagascar","Malawi","Malaysia","Maldives","Mali","Malta","Marshall Islands","Mauritania","Mauritius","Mecklenburg-Schwerin","Mecklenburg-Strelitz","Mexico","Micronesia","Moldova","Monaco","Mongolia","Montenegro","Morocco","Mozambique","Namibia","Nassau","Nauru","Nepal","Netherlands, The","New Zealand","Nicaragua","Niger","Nigeria","Niue","North German Confederation","North German Union","North Macedonia","Norway","Oldenburg","Oman","Orange Free State","Pakistan","Palau","Panama","Papal States","Papua New Guinea","Paraguay","Peru","Philippines","Piedmont-Sardinia","Poland","Portugal","Qatar","Republic of Genoa","Republic of Korea (South Korea)","Republic of the Congo","Romania","Russia","Rwanda","Saint Kitts and Nevis","Saint Lucia","Saint Vincent and the Grenadines","Samoa","San Marino","Sao Tome and Principe","Saudi Arabia","Schaumburg-Lippe","Senegal","Serbia","Seychelles","Sierra Leone","Singapore","Slovakia","Slovenia","Solomon Islands, The","Somalia","South Africa","South Sudan","Spain","Sri Lanka","Sudan","Suriname","Sweden","Switzerland","Syria","Tajikistan","Tanzania","Texas","Thailand","Timor-Leste","Togo","Tonga","Trinidad and Tobago","Tunisia","Turkey","Turkmenistan","Tuvalu","Two Sicilies","Uganda","Ukraine","Union of Soviet Socialist Republics","United Arab Emirates, The","United Kingdom, The","Uruguay","Uzbekistan","Vanuatu","Venezuela","Vietnam","Württemberg","Yemen","Zambia","Zimbabwe"
      ]

      return countries.map((country)=>{return {text:country.replaceAll(" ","-").toLowerCase(), value:country}})
    },
    async getCountry(country){

      const api_key = 'guest:guest'

      let url = null;
      let response = null

      try{

        url = `https://api.tradingeconomics.com/historical/country/${country}/indicator/gdp?c=${api_key}`
        response = await axios.get(url)

        let labels = this.extractColumn(response.data, "DateTime")
        labels = labels.map((label)=>new Date(label).getFullYear().toString())

        let points = this.extractColumn(response.data, "Value")

        points.pop()

        return {

          labels:labels,
          points:points
        }

      } catch (error) {

        console.log(error)
          
        this.error = "Request Failed!"

        return {

          labels:[],
          points:[]
        }
      }
    },
    async changeCountry1(ev){

      this.error = null
      this.country1 = ev.target.value;
      this.renderChart()
    },
    async changeCountry2(ev){

      this.error = null
      this.country2 = ev.target.value;
      this.renderChart()
    },
    whichCountry(val){

      return this.getCountryList().find((el)=>el.text == val).value
    },
    async renderChart(){

      const country1 = await this.getCountry(this.country1)
      const country2 = await this.getCountry(this.country2)
      
      this.data = {
        error:null,
        labels: country1.labels,
        datasets:[
          {
            label: this.whichCountry(this.country1),
            backgroundColor: 'blue',
            data: country1.points
          },
          {
            label: this.whichCountry(this.country2),
            backgroundColor: 'green',
            data: country2.points
          }
        ]
      }
    }
  },
  async created(){

    this.renderChart()
  },
  data(){

    let countries = this.getCountryList()

    return{
      country1:"mexico",
      country2:"sweden",
      countries:countries,
      data: {
        labels: [],
        datasets: [],
      },
      options: {
        responsive: true
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
