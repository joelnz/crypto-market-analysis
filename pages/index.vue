<template>
  <div class="container">
    <h1>Real-time Crypto Market Overview</h1>
    <button @click="getCryptoData">Get Latest Data</button>

    <!-- Display the list of cryptocurrencies -->
    <ul v-if="cryptos.length">
      <li v-for="crypto in cryptos" :key="crypto.symbol">
        <strong>{{ crypto.name }} ({{ crypto.symbol }})</strong> - Price: ${{
          crypto.price
        }}
        - Change: {{ crypto.changePercent24Hr }}%
      </li>
    </ul>

    <!-- Display loading message while ChatGPT is analyzing -->
    <p v-if="isLoading">{{ loadingText }}</p>

    <!-- Display the typed summary -->
    <p class="white" v-if="displayedSummary">{{ displayedSummary }}</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      cryptos: [],
      summary: '',
      displayedSummary: '', // This will be used for the typing effect
      isLoading: false, // To show the loading message
      loadingText: 'Analyzing data', // Text for loading animation
      loadingInterval: null, // To store the interval ID for clearing later
    };
  },
  methods: {
    async getCryptoData() {
      try {
        // Start the loading animation
        this.startLoadingAnimation();

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
        this.cryptos = response.data.Data.map((crypto) => ({
          name: crypto.CoinInfo.FullName,
          symbol: crypto.CoinInfo.Name,
          price: crypto.RAW.USD.PRICE.toFixed(2),
          changePercent24Hr: crypto.RAW.USD.CHANGEPCT24HOUR.toFixed(2),
        }));

        // Use ChatGPT to generate a summary of the market conditions
        const cryptoSummary = this.cryptos
          .map(
            (crypto) =>
              `${crypto.name} (${crypto.symbol}): $${crypto.price} (${crypto.changePercent24Hr}% change in the last 24 hours)`
          )
          .join(', ');

        const chatGptResponse = await axios.post(
          'https://api.openai.com/v1/chat/completions',
          {
            model: 'gpt-4-0613',
            messages: [
              { role: 'system', content: 'You are a financial analyst.' },
              {
                role: 'user',
                content: `Give a very concise paragraph of tips on what a smart investor would do with this data: ${cryptoSummary}`,
              },
            ],
          },
          {
            headers: {
              Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
            },
          }
        );

        this.summary = chatGptResponse.data.choices[0].message.content;
        this.isLoading = false; // Hide loading message
        clearInterval(this.loadingInterval); // Stop the loading animation
        this.typeOutSummary(); // Start typing out the summary
      } catch (error) {
        console.error('Error:', error);
        this.isLoading = false;
        clearInterval(this.loadingInterval); // Stop the loading animation
        this.summary = 'An error occurred while fetching or analyzing data.';
      }
    },
    startLoadingAnimation() {
      this.isLoading = true;
      let dots = '';
      this.loadingInterval = setInterval(() => {
        if (dots.length >= 3) {
          dots = '';
        } else {
          dots += '.';
        }
        this.loadingText = `Analyzing data${dots}`;
      }, 500); // Adjust speed of dots animation here
    },
    typeOutSummary() {
      this.displayedSummary = ''; // Reset displayedSummary before typing
      let index = 0;
      const typingSpeed = 50; // Adjust typing speed here (milliseconds per word)

      const typingInterval = setInterval(() => {
        if (index < this.summary.length) {
          this.displayedSummary += this.summary[index];
          index++;
        } else {
          clearInterval(typingInterval);
        }
      }, typingSpeed);
    },
  },
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background-color: black;
  color: limegreen;
  font-family: 'Monaco', 'Courier New', monospace;
  font-size: 12px;
}

.container {
  width: 100%;
  max-width: 800px;
  background-color: black;
  color: limegreen;
  font-family: 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  padding: 20px;
  text-align: left;
}

h1 {
  font-size: 14px;
  margin-bottom: 20px;
}

button {
  background-color: limegreen;
  color: black;
  border: 2px solid limegreen;
  font-family: 'Monaco', 'Courier New', monospace;
  padding: 5px 10px;
  cursor: pointer;
}

button:hover {
  background-color: black;
  color: limegreen;
}

ul {
  list-style-type: none;
  padding-left: 0;
}

li {
  margin-bottom: 10px;
}

p {
  margin-top: 20px;
}
.white{
  color:white
}
</style>