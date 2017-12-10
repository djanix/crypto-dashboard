<style scoped>
  h1 {
    font-size: 48px;
  }

  h2 {
    font-size: 32px;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }
</style>

<template>
  <section>
    <h1 class="mb-5">{{title}}</h1>

    <h2 class="mb-5">
      Investment: {{cashFormat(total.investment)}}<br>
      Value: {{cashFormat(total.value)}}<br>
      Increase: {{round(total.increase)}}%<br>
    </h2>

    <v-data-table
      v-bind:headers="headers"
      v-bind:pagination.sync="paginationSync"
      must-sort
      :items="userCryptosAggregate"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-right">{{ props.item.coinmarketcap.name }}</td>
        <td class="text-xs-right">{{ props.item.coinmarketcap.symbol }}</td>
        <td class="text-xs-right">{{ cashFormat(props.item.coinmarketcap.price_cad) }}</td>
        <td class="text-xs-right">{{ round(props.item.qty, 3) }}</td>
        <td class="text-xs-right">{{ cashFormat(props.item.investment) }}</td>
        <td class="text-xs-right">{{ cashFormat(props.item.value) }}</td>

        <td v-bind:class="`text-xs-right ${calculateColor(props.item.increase)}--text text--darken-3`">
          {{ round(props.item.increase) }}%
        </td>

        <td v-bind:class="`text-xs-right ${calculateColor(props.item.coinmarketcap.percent_change_1h)}--text text--darken-3`">
          {{ props.item.coinmarketcap.percent_change_1h }}%
        </td>

        <td v-bind:class="`text-xs-right ${calculateColor(props.item.coinmarketcap.percent_change_24h)}--text text--darken-3`">
          {{ props.item.coinmarketcap.percent_change_24h }}%
        </td>

        <td v-bind:class="`text-xs-right ${calculateColor(props.item.coinmarketcap.percent_change_7d)}--text text--darken-3`">
          {{ props.item.coinmarketcap.percent_change_7d }}%
        </td>
      </template>
    </v-data-table>
  </section>
</template>

<script>
  import axios from 'axios';
  import _ from 'lodash';

  export default {
    name: 'index',

    data() {
      return {
        title: 'Crypto Portfolio',

        headers: [
          { text: 'Name', value: 'coinmarketcap.name' },
          { text: 'Symbol', value: 'coinmarketcap.symbol' },
          { text: 'Price', value: 'price' },
          { text: 'Quantity', value: 'qty' },
          { text: 'Investment', value: 'investment' },
          { text: 'Current Value', value: 'value' },
          { text: 'Increase (%)', value: 'increase' },
          { text: '1h', value: 'coinmarketcap.percent_change_1h' },
          { text: '24h', value: 'coinmarketcap.percent_change_24h' },
          { text: '7d', value: 'coinmarketcap.percent_change_7d' },
        ],

        paginationSync: {
          sortBy: null,
          page: 1,
          rowsPerPage: -1,
          descending: true,
          totalItems: 100,
        },

        cryptos: {},

        userCryptos: {},
      };
    },

    computed: {
      userCryptosAggregate() {
        if (_.isEmpty(this.userCryptos) || _.isEmpty(this.cryptos)) return [];

        const quantities = _.mapValues(this.userCryptos, crypto =>
          _.reduce(crypto.items, (total, item) =>
            total + item.qty, 0,
          ),
        );

        return _.map(quantities, (qty, id) => {
          const crypto = _.find(this.cryptos, cr => cr.id === id);
          const value = _.round(crypto.price_cad * qty, 2).toFixed(2);
          const investment = this.userCryptos[id].investment.toFixed(2);

          return {
            id,
            coinmarketcap: crypto,
            investment,
            qty,
            value,
            increase: ((value - investment) / investment) * 100,
          };
        });
      },

      total() {
        const defaultTotal = {
          investment: 0,
          value: 0,
          increase: 0,
        };

        const totalObj = _.reduce(this.userCryptosAggregate, (total, crypto) =>
          Object.assign({}, total, {
            investment: total.investment + parseFloat(crypto.investment),
            value: total.value + parseFloat(crypto.value),
          }), defaultTotal);

        totalObj.increase = ((totalObj.value - totalObj.investment) / totalObj.investment) * 100;

        return totalObj;
      },
    },

    created() {
      this.getCoinmarketcapData();
      this.getUserData();
    },

    methods: {
      getCoinmarketcapData() {
        // https://coinmarketcap.com/api/
        axios.get('https://api.coinmarketcap.com/v1/ticker/', {
          params: {
            convert: 'CAD',
            limit: 50,
          },
        })
        .then((response) => {
          const data = response.data;
          const cryptoList = _.keys(this.userCryptos);
          this.cryptos = _.filter(data, d => _.includes(cryptoList, d.id));
        });
      },

      getUserData() {
        axios.get('/static/data.json')
          .then((response) => {
            this.userCryptos = response.data;
          });
      },

      calculateColor(value) {
        return parseFloat(value) >= 0 ? 'green' : 'red';
      },

      round: (num, roundVal = 2) => _.round(num, roundVal),

      cashFormat: num => `${parseFloat(num).toFixed(2)}$`,
    },
  };
</script>
