<template>
  <div>
    <h1>Crypto Market Overview</h1>
    <button @click="getCryptoData">Get Latest Crypto Data</button>
    <p v-if="summary">{{ summary }}</p>
    <ul v-if="cryptos.length">
      <li v-for="crypto in cryptos" :key="crypto.symbol">
        <strong>{{ crypto.name }} ({{ crypto.symbol }})</strong> - 
        Price: ${{ crypto.price }} - 
        Change: {{ crypto.changePercent24Hr }}%
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      cryptos: [],
      summary: null,
    };
  },
  methods: {
    async getCryptoData() {
      try {
        // Fetch real-time cryptocurrency data from CryptoCompare
        const response = await axios.get(
          `https://min-api.cryptocompare.com/data/top/mktcapfull`,
          {
            params: {
              limit: 10,
              tsym: 'USD',
              api_key: process.env.CRYPTOCOMPARE_API_KEY,
            },
          }
        );

        // Process the data to extract useful information
        this.cryptos = response.data.Data.map(crypto => ({
          name: crypto.CoinInfo.FullName,
          symbol: crypto.CoinInfo.Name,
          price: crypto.RAW.USD.PRICE.toFixed(2),
          changePercent24Hr: crypto.RAW.USD.CHANGEPCT24HOUR.toFixed(2),
        }));

        // Use ChatGPT to generate a summary of the market conditions
        const cryptoSummary = this.cryptos.map(crypto => 
          `${crypto.name} (${crypto.symbol}): $${crypto.price} (${crypto.changePercent24Hr}% change in the last 24 hours)`
        ).join(', ');

        const chatGptResponse = await axios.post(
          'https://api.openai.com/v1/chat/completions',
          {
            model: 'gpt-4-0613',
            messages: [
              { role: 'system', content: 'You are a financial analyst.' },
              { role: 'user', content: `Provide a brief and clear market summary based on the following data: ${cryptoSummary}` },
            ],
          },
          {
            headers: {
              Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
            },
          }
        );

        this.summary = chatGptResponse.data.choices[0].message.content;
      } catch (error) {
        console.error('Error:', error);
        this.summary = "An error occurred while fetching crypto data.";
      }
    },
  },
};
</script>