<template>
  <div class="container container-lg">
    <tabs fill class="flex-column flex-md-row">
      <card shadow>
        <tab-pane title="Total Purchases for the month">
          <div class="center">
            <date-picker v-model="value" type="date" range placeholder="Select date range" 
                :disabled-date="(date) => date >= new Date()"></date-picker>
            <button class="button" @click="onReload">Reload Chart</button>
          </div>
          <line-chart :data="lineone"></line-chart>
        </tab-pane>
        <tab-pane title="Total revenue by month">
          <div class="center">
            <date-picker v-model="value" type="date" range placeholder="Select date range" 
                :disabled-date="(date) => date >= new Date()"></date-picker>
            <button class="button" @click="onReload">Reload Chart</button>
          </div>
          <line-chart :data="linetwo"></line-chart>
        </tab-pane>
        <tab-pane title="Statistics of photo status">
          <pie-chart :data=chartData></pie-chart>
        </tab-pane>
      </card>
    </tabs>
  </div>
</template>

<script>
import ImageCard from './ImageCard';
import axios from 'axios';
import DatePicker from "vue2-datepicker";
import "vue2-datepicker/index.css";
import Tabs from "@/components/Tabs/Tabs.vue";
import TabPane from "@/components/Tabs/TabPane.vue";

export default {
  
  components:{
    Tabs,
    TabPane,
    ImageCard,
    DatePicker
  },
  data() {
    return {
      value: [],
      lineone: null,
      linetwo: null,
      chartData: [],
      user : JSON.parse(window.localStorage.getItem('user'))
    };
  },
  computed: {
    images() {
      return this.$store.state.approved_images;
    },
     isLoggedIn() {
      return this.$store.getters.isLoggedIn;
    }
  },
  methods: {
    onReload() {
      let dt = {
        "startDate": this.value[0],
        "endDate": this.value[1],
        "userId": this.user.userId
      }
      axios.post("https://imago.azurewebsites.net/api/v1/User/GetSellPhotoStatistic", dt)
      .then((response) => {
        this.lineone = [{name: 'Total Photo', data: JSON.parse(JSON.stringify(response.data[0]))}],
        this.linetwo = [{name: 'Total Amount', data: JSON.parse(JSON.stringify(response.data[1]))}]
        console.log(response.data[0])
      })
      .catch((error) => {
        console.log(error);
      });
    }
  },
  mounted() {
    var x = new Date();
    this.value.push(x);
    x.setDate(1);
    x.setMonth(x.getMonth()-3);
    
    var y = new Date();
    this.value.push(y);

    let user = this.user;
    console.log(user.userId);
    axios.get("https://imago.azurewebsites.net/api/v1/User/GetPhotoStatusStatistic/" + user.userId)
    .then((response) => {
      this.chartData = { 
        "Approved Photo": response.data.approvedPhoto,
        "Pending Photo": response.data.pendingPhoto,
        "Denied Photo": response.data.deniedPhoto
      }
    })
    .catch((error) => {
      console.log(error);
    });
    let dt = {
      "startDate": this.value[0],
      "endDate": this.value[1],
      "userId": this.user.userId
    }
    axios.post("https://imago.azurewebsites.net/api/v1/User/GetSellPhotoStatistic", dt)
    .then((response) => {
      this.lineone = [{name: 'Total Photo', data: JSON.parse(JSON.stringify(response.data[0]))}],
      this.linetwo = [{name: 'Total Amount', data: JSON.parse(JSON.stringify(response.data[1]))}]
      console.log(response.data[0])
    })
    .catch((error) => {
      console.log(error);
    });

  },
};
</script>

<style>
.button {
  height: 34px;
}
.center {
  margin: auto;
  width: 60%;
  text-align: center;
  padding: 10px;
}
.linechartkick {
    margin-left: 10%;
  width: 80%;
}
.date {
    margin-right: 5%;
}
.content {
    margin-top: 5%;
}
</style>