<template>
  <div id="main-app" class="container">
    <div class="row justify-content-center">
      <AddAppointment @add="addItem" />
      <SearchAppointments
        :myKey="filterKey"
        :myDir="filterDir"
        @searchRecords="searchAppointments"
        @requestKey="changeKey"
        @requestDir="changeDir"
      />
      <AppointmentList
        :appointments="filteredApts"
        @remove="removeItem"
        @edit="editItem"
      />
    </div>
  </div>
</template>

<script>
import AddAppointment from "./components/AddAppointment";
import AppointmentList from "./components/AppointmentList";
import SearchAppointments from "./components/SearchAppointments";
import _ from "lodash";
import axios from "axios";

export default {
  name: "MainApp",
  data: function() {
    return {
      appointments: [],
      filterKey: "petName",
      filterDir: "asc",
      aptIndex: 0,
      searchTerms: ""
    };
  },
  components: {
    AppointmentList,
    AddAppointment,
    SearchAppointments
  },
  mounted() {
    axios.get("./data/appointments.json").then(
      response =>
        (this.appointments = response.data.map(item => {
          item.aptId = this.aptIndex;
          this.aptIndex++;
          return item;
        }))
    );
  },
  computed: {
    searchedApts: function() {
      return this.appointments.filter(item => {
        return (
          item.petName.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.petOwner.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.aptNotes.toLowerCase().match(this.searchTerms.toLowerCase())
        );
      });
    },
    filteredApts: function() {
      return _.orderBy(
        this.searchedApts,
        item => {
          return item[this.filterKey].toLowerCase();
        },
        this.filterDir
      );
    }
  },
  methods: {
    changeKey: function(value) {
      this.filterKey = value;
    },
        changeDir: function(value) {
      this.filterDir = value;
    },
    addItem: function(apt) {
      apt.aptId = this.aptIndex;
      this.aptIndex++;
      this.appointments.push(apt);
    },
    removeItem: function(apt) {
      this.appointments = _.without(this.appointments, apt);
    },
    editItem: function(id, field, text) {
      const aptIndex = _.findIndex(this.appointments, {
        aptId: id
      });
      this.appointments[aptIndex][field] = text;
    },
    searchAppointments: function(terms) {
      this.searchTerms = terms;
    }
  }
};
</script>
