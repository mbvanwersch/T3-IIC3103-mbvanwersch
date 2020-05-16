<style>
h1   {
      font-family:Optima;
      text-align:center;
      background-color:#3366CC;
      padding: 30px;
      font-size: 230%;
      position: fixed;
      width: 100%;
     }
h2   {
      font-family:Optima;
      text-align:center;
     }
h3   {
      font-family:Optima;
     }
p   {
     font-family:Optima;
    }
ul {
    font-family:Optima;
    display: inline;
    background-color:#3366CC;
    list-style-type: none;
    margin: 0;
    padding: 1px;
    overflow: hidden;
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 1;
   }

tr {
    font-family:Optima;
}

body, html {
    margin-left:3%;
    margin-right:3%;
}

button {
  font-family:Optima;
  background-color: #3366CC;
  text-align: center;
  border: none;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  color: #FFFFFF;
}

li {
  display: inline;
  font-family:Optima;
  font-size: 16px;
  text-decoration: none;
  float: left;
  padding: 20px;
  color: #FFFFFF;
  background-color: #3366CC;
}

.column {
  float: left;
  width: 33.33%;
}

.row:after {
  content: "";
  display: table;
  clear: both;
}

a {
  text-decoration: none;
  color:#FFFFFF;
  }

</style>

<template>
  <div id="app">

  <ul>
    <li style="font-size: 20px;" ><b><a href="#acciones">Información bursátil en vivo </a></b></li>
    <li>|</li>
    <li><a href="#acciones">Ver Acciones</a></li>
    <li>|</li>
    <li><a href="#mercados">Ver Mercados</a></li>
    <li>|</li>
    <li>Estado: {{ estado }}</li>
    <li>|</li>
    <li v-if="estado == 'Desconectado'"><button v-on:click="conectar">Conectar</button><br/></li>
    <li v-else><button v-on:click="desconectar">Desconectar</button><br/></li>
  </ul>

    <br id="acciones">
    <br>
    <br>
    <br>
    <h2>Acciones</h2>
          <template v-for="(key, value) in stocks_dictionary">
          <h3>{{ key.nombre }}</h3>
          <div class="row">
            <div class="column">
              <p>Ticker: {{ value }}</p>
              <p>País de origen: {{ key.pais }}</p>
              <p>Volumen total transado: {{ key.volumen_total }}</p>
            </div>
            <div class="column">
              <br>
              <p>Alto histórico: ${{ key.alto_historico }}</p>
              <p>Bajo histórico: ${{ key.bajo_historico }}</p>
            </div>
            <div class="column">
              <br>
              <p>Último precio: ${{ key.ultimo_precio }}</p>
              <p>Variación porcentual: {{ key.variacion_porcentual }}%</p>
            </div>
          </div>
            <GChart
              type="LineChart"
              :data="chartData[value]"
              :options="chartOptions"
            />
            <br>
            <br>
          </template>

    <br id="mercados">
    <br>
    <br>
    <hr>
    <h2>Mercados de valores</h2>
      <template v-for="(key, value) in exchange_dictionary">
        <h3>{{ key.nombre }}</h3>
        <div class="row">
          <div class="column">
            <p>Sigla: {{ key.sigla }}</p>
            <p>Volumen compra: {{ key.volumen_compra }}</p>
          </div>
          <div class="column">
            <p>Volumen venta: {{ key.volumen_venta }}</p>
            <p>Volumen total: {{ key.volumen_total }}</p>
          </div>
          <div class="column">
            <p>Cantidad de acciones: {{ key.cant_acciones }}</p>
            <p>Participación de mercado: {{ key.part_mercado }}%</p>
          </div>
        </div>
        <br>
        <br>
      </template>
    <br>
    <br>
    <br>
    <br>
    <br>
    <br>
    <br>
    <br>
  </div>
</template>


<script>
import io from "socket.io-client";
import { GChart } from 'vue-google-charts'

var socket = io.connect('wss://le-18262636.bitzonte.com', {
    path: '/stocks'
});

export default {
  name: 'app',
  components: {
    GChart
  },
  data: function() {
    return {
      estado: 'Conectado',
      stocks_dictionary: {},
      exchange_dictionary: {},
      nombre_sigla_stock: {},
      stock_exchange: {},
      volumen_total_mercados: 0,
      chartData: {},
      chartOptions: {chart: {title: 'Company Performance'}, hAxis: { textPosition: 'none', title: 'Tiempo' }, vAxis: {title: 'Valor acción'}}
    }
  },

  created() {
      console.log("Estoy conectado");
      socket.emit('STOCKS'),
      socket.emit('EXCHANGES'),

      socket.on('STOCKS', (data) => {
        console.log(data);
        for (const m of data) {
            //console.log(m);
            this.$set(this.stocks_dictionary, m.ticker, {volumen_total: 0, alto_historico: 0, bajo_historico: 0, ultimo_precio: 0, variacion_porcentual: 0, nombre: m.company_name, pais: m.country})
            //console.log(this.stocks_dictionary);
            this.$set(this.nombre_sigla_stock, m.company_name, m.ticker)
            //console.log(this.nombre_sigla_stock);
            this.$set(this.chartData, m.ticker, []);
            this.chartData[m.ticker].push(['Tiempo', m.ticker]);

        }
        console.log(this.stocks_dictionary);
      }),

      socket.on('EXCHANGES', (data) => {
        console.log(data);
        for (var key in data) {
            this.$set(this.exchange_dictionary, key, {sigla: key, nombre: data[key]['name'], volumen_compra: 0, volumen_venta: 0, volumen_total: 0, cant_acciones: data[key]['listed_companies'].length, part_mercado: 0, acciones: Array()})
            var index = 0;
            while (index < data[key]['listed_companies'].length) {
              this.exchange_dictionary[key]['acciones'].push(this.nombre_sigla_stock[data[key]['listed_companies'][index]]);
              this.$set(this.stock_exchange, this.nombre_sigla_stock[data[key]['listed_companies'][index]], key)
              index++;
            }
            //console.log(this.exchange_dictionary);
            //console.log(this.stock_exchange);
        }
        console.log(this.exchange_dictionary);
      }),

      this.getRealtimeData()
  },

  methods: {
    getRealtimeData() {
        socket.on('UPDATE', (data) => {
          console.log(data.ticker, ' UPDATE: value ', data.value);
          if (data.ticker in this.stocks_dictionary) {
            const precio_anterior = this.stocks_dictionary[data.ticker]['ultimo_precio'];
            //console.log(this.stocks_dictionary[data.ticker]);
            //console.log("Actualizo el valor");
            this.$set(this.stocks_dictionary[data.ticker], 'ultimo_precio', data.value)
            //this.stocks_dictionary[data.ticker]['ultimo_precio'] = data.value;
            //console.log(this.stocks_dictionary[data.ticker]);
            if (this.stocks_dictionary[data.ticker]['bajo_historico'] == 0) {
              //console.log("Actualizo el menor por primera vez");
              this.$set(this.stocks_dictionary[data.ticker], 'bajo_historico', data.value)
              //this.stocks_dictionary[data.ticker].bajo_historico = data.value;
            }
            if (data.value < this.stocks_dictionary[data.ticker]['bajo_historico']) {
              //console.log("Actualizo el menor");
              this.$set(this.stocks_dictionary[data.ticker], 'bajo_historico', data.value)
              //this.stocks_dictionary[data.ticker].bajo_historico = data.value;
            }
            if (data.value > this.stocks_dictionary[data.ticker]['alto_historico']) {
              //console.log("Actualizo el mayor");
              this.$set(this.stocks_dictionary[data.ticker], 'alto_historico', data.value)
              //this.stocks_dictionary[data.ticker].alto_historico = data.value;
            }
            if (precio_anterior != 0) {
              this.$set(this.stocks_dictionary[data.ticker], 'variacion_porcentual', (data.value - precio_anterior)*100 / precio_anterior)
              //this.stocks_dictionary[data.ticker]['variacion_porcentual'] = (data.value - precio_anterior)*100 / precio_anterior;
            }
            //console.log(this.stocks_dictionary[data.ticker]);
            this.chartData[data.ticker].push([new Date(data.time), data.value]);
          }
        }),

        socket.on('BUY', (data) => {
          //console.log(data.ticker, ' BUY: volume ', data.volume);
          if (data.ticker in this.stocks_dictionary) {
            //console.log(this.stocks_dictionary[data.ticker]);
            this.$set(this.stocks_dictionary[data.ticker], 'volumen_total', this.stocks_dictionary[data.ticker]['volumen_total'] + data.volume)
            //this.stocks_dictionary[data.ticker]['volumen_total'] = this.stocks_dictionary[data.ticker]['volumen_total'] + data.volume;
            //console.log("Actualizo el volumen");
            //console.log(this.stocks_dictionary[data.ticker]);
          }
          if (data.ticker in this.stock_exchange) {
            this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'volumen_compra', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_compra'] + data.volume)
            //this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_compra'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_compra'] + data.volume;
            this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'volumen_total', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] + data.volume)
            //this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] + data.volume;
            this.volumen_total_mercados = this.volumen_total_mercados + data.volume;
            this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'part_mercado', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total']*100 / this.volumen_total_mercados)
            //this.exchange_dictionary[this.stock_exchange[data.ticker]]['part_mercado'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total']*100 / this.volumen_total_mercados;
            //console.log(this.exchange_dictionary[this.stock_exchange[data.ticker]]);
          }
        }),

        socket.on('SELL', (data) => {
           //console.log(data.ticker, ' SELL: volume ', data.volume);
           if (data.ticker in this.stocks_dictionary) {
             //console.log(this.stocks_dictionary[data.ticker]);
             this.$set(this.stocks_dictionary[data.ticker], 'volumen_total', this.stocks_dictionary[data.ticker]['volumen_total'] + data.volume)
             //this.stocks_dictionary[data.ticker]['volumen_total'] = this.stocks_dictionary[data.ticker]['volumen_total'] + data.volume;
             //console.log("Actualizo el volumen");
             //console.log(this.stocks_dictionary[data.ticker]);
           }
           if (data.ticker in this.stock_exchange) {
             this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'volumen_venta', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_venta'] + data.volume)
             //this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_venta'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_venta'] + data.volume;
             this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'volumen_total', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] + data.volume)
             //this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total'] + data.volume;
             this.volumen_total_mercados = this.volumen_total_mercados + data.volume;
             this.$set(this.exchange_dictionary[this.stock_exchange[data.ticker]], 'part_mercado', this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total']*100 / this.volumen_total_mercados)
             //this.exchange_dictionary[this.stock_exchange[data.ticker]]['part_mercado'] = this.exchange_dictionary[this.stock_exchange[data.ticker]]['volumen_total']*100 / this.volumen_total_mercados;
             //console.log(this.exchange_dictionary[this.stock_exchange[data.ticker]]);
           }
        })
    },

    conectar: function (event) {
      socket.connect();
      this.estado = 'Conectado';
      console.log("Me conecté");
    },

    desconectar: function (event) {
      socket.disconnect();
      this.estado = 'Desconectado';
      console.log("Me deconecté");
    }
  }
};
</script>

<style>
.small {
  max-width: 600px;
  margin: 150px auto;
}
</style>
