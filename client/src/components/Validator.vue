<template>
<div id="container" class="col-lg-12 col-md-12 col-sm-12">
  <form id="validate">
    <div class="row">
      <div class="col-lg-6 col-md-12 col-sm-12 col-xs-12">
        <h4>Enter Fedora CoreOS Config:</h4>
        <div class="co-p-validate-wrapper">
          <div class="co-p-validate-lines">
            <div v-for="index in lines" :key="index" v-bind:id="'lineno-' + index">{{ index }}</div>
          </div>
          <!-- eslint-disable-next-line -->
          <textarea v-model="fcc_config" id="validate-config" wrap="off" spellcheck="false" autofocus="" rows="40"></textarea>
        </div>
      </div>
      <div class="col-lg-6 col-md-12 col-sm-12 col-xs-12">
        <h4>Transpiled Ignition Config:</h4>
        <div class="co-p-validate-results-wrapper">
          <!-- eslint-disable-next-line -->
          <textarea readonly v-model="ignition_config" id="validate-results" wrap="off" spellcheck="false" autofocus="" rows="40"></textarea>
        </div>
      </div>
    </div>
    <!-- eslint-disable-next-line -->
    <input type="submit" id="validate-submit" class="btn btn-primary" value="Transpile" data-category="Validator" v-on:click="submit">
    <!-- eslint-disable-next-line -->
    <button type="button" id="generate-url" class="btn btn-secondary" v-on:click="showURL">Generate URL From FCC</button>
    <!-- eslint-disable-next-line -->
    <textarea readonly v-if="visible" v-model="encoded_url" id="encoded-url-box" wrap="off" spellcheck="false" autofocus="" rows="3"></textarea>
  </form>
</div>
</template>

<script>
import axios from 'axios';

const serverURL = '/config/';
const clientURL = window.location.origin.concat('/');

export default {
  name: 'Validator',

  data() {
    return {
      fcc_config: '',
      ignition_config: '',
      encoded_url: '',
      visible: false,
      current_lineno: [],
    };
  },

  computed: {
    lines() {
      const defaultLines = 40;
      const fcc = this.fcc_config.split(/\r|\r\n|\n/);
      const fccLines = fcc.length;
      return Math.max(defaultLines, fccLines);
    },
  },

  methods: {
    // clean the ignition config textbox
    cleanIgnitionBox() {
      this.ignition_config = '';
    },
    // reset the color of error lines
    clearLineColorAll() {
      this.current_lineno.forEach((lineno) => {
        document.getElementById(`lineno-${lineno}`).style.setProperty('color', '#ccc');
        document.getElementById(`lineno-${lineno}`).style.setProperty('background', 'initial');
      });
      this.current_lineno = [];
    },
    // set the color of error lines
    setLineColorAll(linenoList) {
      this.clearLineColorAll();
      linenoList.forEach((lineno) => {
        this.current_lineno.push(lineno);
        document.getElementById(`lineno-${lineno}`).style.color = 'red';
        document.getElementById(`lineno-${lineno}`).style.background = '#fcd9dd';
      });
    },
    // generate url from fcc
    convertFccToUrl() {
      return clientURL.concat(encodeURIComponent(this.fcc_config));
    },
    // send fcc to server and set ignition config from response
    toIgnitionConfig() {
      this.encoded_url = this.convertFccToUrl();
      const postData = { config_string: this.fcc_config };
      axios.post(serverURL, postData)
        .then((res) => {
          try {
            if (res.data.success) {
              this.clearLineColorAll();
              document.getElementById('validate-results').style.color = 'green';
              this.ignition_config = JSON.stringify(res.data.ignition_config, null, 2);
            } else {
              this.setLineColorAll(res.data.err_lines);
              document.getElementById('validate-results').style.color = 'red';
              this.ignition_config = res.data.err_msg;
            }
          } catch (err) {
            this.clearLineColorAll();
            this.ignition_config = '';
          }
        })
        .catch((error) => {
          this.clearLineColorAll();
          // eslint-disable-next-line
          console.error(error);
        });
    },
    // submit button callback
    submit(e) {
      e.preventDefault();
      this.cleanIgnitionBox();
      this.toIgnitionConfig();
    },
    // url button callback
    showURL() {
      this.visible = !!this.fcc_config;
      this.encoded_url = this.convertFccToUrl();
    },
    // scroll function for fcc textbox
    handleScroll() {
      const { scrollTop } = document.querySelector('#validate-config');
      const elem = document.querySelector('.co-p-validate-lines');
      elem.style['margin-top'] = `${-1 * scrollTop}px`;
    },
  },

  created() {
    if (this.$route.params.fcc_config) {
      this.fcc_config = this.$route.params.fcc_config;
      this.encoded_url = this.convertFccToUrl();
      this.showURL();
      this.toIgnitionConfig();
    }
  },

  mounted() {
    // Add event listener to extend line number
    document.getElementById('validate-config').addEventListener('scroll', this.handleScroll);
    // Enable triple click to select all content
    document.getElementById('validate-config').addEventListener('click', (e) => {
      if (e.detail === 3) {
        document.getElementById('validate-config').select();
      }
    });
    // Enable triple click to select all content
    document.getElementById('validate-results').addEventListener('click', (e) => {
      if (e.detail === 3) {
        document.getElementById('validate-results').select();
      }
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h4 {
  margin-top: 0.5em;
  margin-bottom: 0.5em;
}
.row {
  margin-bottom: 0.6em;
}
.co-p-validate-wrapper {
  position: relative;
  height: 560px;
  width: 100%;
  font-size: 14px;
  line-height: 1.428571429;
  border-radius: 4px;
  -webkit-box-shadow: inset 0 1px 1px rgba(0,0,0,0.075);
  box-shadow: 0 0 3px rgba(153,153,153,.75);
  border: 0px;
  overflow: hidden;
}
.co-p-validate-results-wrapper {
  position: relative;
  height: 560px;
  width: 100%;
  font-size: 14px;
  line-height: 1.428571429;
  border-radius: 4px;
  -webkit-box-shadow: inset 0 1px 1px rgba(0,0,0,0.075);
  box-shadow: 0 0 3px rgba(153,153,153,.75);
  border: 0px;
  overflow: hidden;
}
#validate-config {
  width: 94%;
  left: 40px;
  padding-right: 0;
  position: absolute;
  background: transparent;
  border: 0px;
  font-size: 10pt;
  font-family: monospace;
  line-height: 14px !important;
  resize: none;
}
#validate-config:focus {
  outline-color: transparent;
  outline-style: none;
}
#validate-results {
  width: 98%;
  left: 1%;
  position: absolute;
  background: transparent;
  border: 0px;
  font-size: 10pt;
  font-family: monospace;
  line-height: 14px !important;
  resize: none;
}
#validate-results:focus {
  outline-color: transparent;
  outline-style: none;
}
#validate-submit {
  margin-top: 0.5em;
  margin-bottom: 1em;
}
#container {
  overflow: auto;
}
#generate-url {
  margin-top: 0.5em;
  margin-left: 0.9em;
  margin-bottom: 1em;
}
#encoded-url-box {
  width: 100%;
  position: relative;
  background: transparent;
  border: 0px;
  font-size: 10pt;
  font-family: monospace;
  line-height: 14px !important;
  box-shadow: 0 0 3px rgba(153,153,153,.75);
  resize: none;
}
#encoded-url-box:focus {
  outline-color: transparent;
  outline-style: none;
}
.co-p-validate-lines {
  width: 35px;
  padding-right: 5px;
  position: absolute;
  top: 0px;
  left: 0px;
  border-right: 1px solid #eee;
  font-size: 10pt;
  font-family: monospace;
  line-height: 14px !important;
  text-align: right;
  color: #ccc;
  padding-top: 2px;
  height: 560px;
}
</style>
