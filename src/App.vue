<template>
  <div id="app">
    <TopBar/>
    <div class="section">
      <div class="side-bar">
        <h5>List of Risk Types</h5>
        <div v-for="risk_type of risk_types" v-bind:key="risk_type.id">
          <span
            class="anchor"
            v-bind:class="{active: risk_type.id === currentContent}"
            v-on:click="setContentId(risk_type.id)"
          >{{ risk_type.risk_name }}</span>
        </div>
      </div>
      <div class="right-bar">
        <h5>FORM VIEW</h5>
        <h5>{{ risk_type.risk_name }}</h5>
        <h6 v-if="risk_type.risk_name">Risk type: {{ risk_type.risk_type }}</h6>
        <form>
          <div v-for="field of risk_type.risk_fields" v-bind:key="field.id">
            <span style="font-size:12px">{{ field.field_name }}:</span>
            <input
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'text'"
              type="text"
            >
            <input
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'number'"
              type="number"
            >
            <input
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'date'"
              type="date"
            >
            <select
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'enum'"
              name
              id
            >
              <option
                v-for="choice of JSON.parse(field.enum_choices)"
                :key="choice"
                v-bind:value="choice"
              >{{choice}}</option>
            </select>
          </div>
          <div>
            <button v-if="risk_type.risk_name" v-on:click="handleSubmit">Submit</button>
          </div>
        </form>
      </div>
      <div class="raw-data">
        <h5 style="font-size:30px">Raw Data</h5>
        <pre v-if="Object.keys(form_fields).length">
          {{ form_fields }}
        </pre>
        <div v-if="Object.keys(http_response).length">
          <h5 style="font-size:30px">HTTP Response</h5>
          <pre>
            {{ http_response }}
          </pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Axios from "axios";

import TopBar from "./components/TopBar.vue";

export default {
  name: "app",
  data: function() {
    return {
      currentContent: 1,
      risk_types: null,
      risk_type: {},
      form_fields: {},
      http_response: {}
    };
  },
  components: {
    TopBar
  },
  methods: {
    setContentId: function(id) {
      this.currentContent = id;
      Axios.get(
        `https://0n06dowzs1.execute-api.us-east-2.amazonaws.com/dev_us_east_2/risk_type/${
          this.currentContent
        }`
      ).then(response => {
        this.risk_type = response.data;
        this.form_fields = {};
      });
    },
    handleSubmit: function(ev) {
      ev.preventDefault();
      Axios.post(
        `https://0n06dowzs1.execute-api.us-east-2.amazonaws.com/dev_us_east_2/risk`,
        {
          json_body: JSON.stringify(this.form_fields)
        }
      ).then(response => {
        this.http_response = response.data;
        this.form_fields = {};
        alert("Risk created successfully");
      });
    }
  },
  mounted() {
    Axios.get(
      "https://0n06dowzs1.execute-api.us-east-2.amazonaws.com/dev_us_east_2/risk_type"
    ).then(response => {
      this.risk_types = response.data;
    });
    Axios.get(
      `https://0n06dowzs1.execute-api.us-east-2.amazonaws.com/dev_us_east_2/risk_type/1`
    ).then(response => {
      this.risk_type = response.data;
    });
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.section {
  display: flex;
  width: 100%;
}

.side-bar {
  width: 30%;
  height: 100%;
  font-size: 30px;
  text-transform: capitalize;
  text-align: left;
  border-color: rgb(218, 213, 213);
  border-width: 0 1px 1px 0;
  border-style: solid;
}

.right-bar {
  width: 50%;
  height: 100%;
  font-size: 30px;
  border-color: rgb(218, 213, 213);
  border-width: 0 1px 1px 0;
  border-style: solid;
  left: 30%;
}

.raw-data {
  left: 80%;
  width: 20%;
  height: 100%;
  border-color: rgb(218, 213, 213);
  border-width: 0 1px 1px 0;
  border-style: solid;
  overflow: auto;
}

.anchor {
  cursor: pointer;
}

.active {
  text-decoration: underline;
}
</style>
