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
            <input
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'text'"
              type="text"
              :placeholder="field.field_name"
            >
            <input
              v-model="form_fields[field.field_name]"
              v-if="field.field_type === 'number'"
              type="number"
              :placeholder="field.field_name"
            >
            <div v-if="field.field_type === 'date'">
              <span
                style="width:20%;font-size:16px;margin-right:13%"
              >{{field.field_name.toUpperCase()}}:</span>
              <input
                style="width:40% !important"
                v-model="form_fields[field.field_name]"
                type="date"
                :placeholder="field.field_name"
              >
            </div>
            <div v-if="field.field_type === 'enum'">
              <span
                style="width:20%;font-size:16px;margin-right:13%"
              >{{field.field_name.toUpperCase()}}:</span>
              <select style="width:40% !important" v-model="form_fields[field.field_name]" name id>
                <option
                  v-for="choice of JSON.parse(field.enum_choices)"
                  :key="choice"
                  v-bind:value="choice"
                >{{choice}}</option>
              </select>
            </div>
          </div>
          <div>
            <span style="color:red">{{ error_message }}</span>
          </div>
          <div>
            <button v-if="risk_type.risk_name && !submitting" v-on:click="handleSubmit">Submit</button>
            <span v-if="submitting">Submitting Risk Data...</span>
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
    <Footer/>
  </div>
</template>

<script>
import Axios from "axios";

import TopBar from "./components/TopBar.vue";
import Footer from "./components/Footer.vue";

export default {
  name: "app",
  data: function() {
    return {
      currentContent: 1,
      risk_types: null,
      risk_type: {},
      form_fields: {},
      http_response: {},
      error_message: "",
      submitting: false
    };
  },
  components: {
    TopBar,
    Footer
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
      let canSubmit = true;

      if (!Object.keys(this.form_fields).length) {
        this.error_message = "* Please fill out all the fields";
        return;
      }

      if (
        Object.keys(this.form_fields).length !==
        Object.keys(this.risk_type).length - 1
      ) {
        this.error_message = "* Please fill out all the fields";
        return;
      }

      Object.keys(this.form_fields).forEach(key => {
        if (!this.form_fields[key]) {
          canSubmit = false;
        }
      });

      if (!canSubmit) {
        this.error_message = "* Please fill out all the fields";
        return;
      }

      const promise = Axios.post(
        `https://0n06dowzs1.execute-api.us-east-2.amazonaws.com/dev_us_east_2/risk`,
        {
          json_body: JSON.stringify(this.form_fields)
        }
      );
      this.submitting = true;
      promise.then(response => {
        this.http_response = response.data;
        this.form_fields = {};
        this.error_message = "";
        alert("Risk created successfully");
        this.submitting = false;
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

input,
select {
  font-size: 16px !important;
  line-height: 22px !important;
  letter-spacing: normal !important;
  font-family: Circular, -apple-system, BlinkMacSystemFont, Roboto,
    "Helvetica Neue", sans-serif !important;
  color: rgb(72, 72, 72) !important;
  font-weight: normal !important;
  background-color: transparent !important;
  width: 60% !important;
  border-width: 1px !important;
  border-style: initial !important;
  border-color: initial !important;
  border-image: initial !important;
  padding: 11px !important;
  border-radius: 2px !important;
  border-bottom-style: solid !important;
}

button {
  width: 60%;
  height: 40px;
  font-size: 24px;
  margin-top: 20px;
  color: white;
  background-color: #303e4e;
  border-radius: 3px;
  margin-bottom: 20px;
}
</style>
