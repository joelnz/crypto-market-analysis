<template>
  <div>
    <h1>Events Near You</h1>
    <button @click="getEvents">Find Events</button>
    <p v-if="summary">{{ summary }}</p>
    <ul v-if="events.length">
      <li v-for="event in events" :key="event.id">
        <strong>{{ event.name.text }}</strong> - {{ event.start.local }}
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      events: [],
      summary: null,
    };
  },
  methods: {
    async getEvents() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
          const lat = position.coords.latitude;
          const lng = position.coords.longitude;

         try {
  const eventbriteResponse = await axios.get(
    `https://www.eventbriteapi.com/v3/events/search/`,
    {
      params: {
        'location.latitude': lat,
        'location.longitude': lng,
        'sort_by': 'date',
        'start_date.range_start': new Date().toISOString(), // For events starting from now
        'token': process.env.EVENTBRITE_API_KEY, // Your Eventbrite API key
      },
      headers: {
        Authorization: `Bearer ${process.env.EVENTBRITE_API_KEY}`,
      },
    }
  );

  this.events = eventbriteResponse.data.events;

  if (this.events.length === 0) {
    this.summary = "No events found near your location.";
  } else {
    // Summarize the events
    const eventDescriptions = this.events.map(event => event.name.text).join(', ');
    const chatGptResponse = await axios.post(
      'https://api.openai.com/v1/chat/completions',
      {
        model: 'gpt-4-0613', // Update this to your specific model
        messages: [
          { role: 'system', content: 'You are a helpful assistant.' },
          { role: 'user', content: `Summarize the following events happening near the user: ${eventDescriptions}` },
        ],
      },
      {
        headers: {
          Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
        },
      }
    );

    this.summary = chatGptResponse.data.choices[0].message.content;
  }
} catch (error) {
  console.error('Error:', error);
  this.summary = "An error occurred while fetching events.";
}
        });
      } else {
        alert('Geolocation is not supported by this browser.');
      }
    },
  },
};
</script>