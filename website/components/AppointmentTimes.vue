<template>
  <div class="mb-3">
    <ul class="mb-0">
      <template v-if="initialAppointments.length > 0">
        <li v-for="appointment in initialAppointments" :key="appointment.id">
          {{ appointment.time }}
          <span v-if="appointment.type">({{ appointment.type }})</span>
        </li>
      </template>
      <template v-else>
        <li v-if="storeVaccineTypes">Vaccine Type: {{ storeVaccineTypes }}</li>
        <li>
          View available appointment times on the
          {{ store.properties.provider_brand_name }} website.
        </li>
      </template>
    </ul>

    <div v-if="moreAppointments.length > 0">
      <div :id="`location-${store.properties.id}-more-appointments-toggle`">
        <a
          href="#"
          :onclick="`document.getElementById('location-${store.properties.id}-more-appointments').style.display = 'block'; document.getElementById('location-${store.properties.id}-more-appointments-toggle').style.display = 'none'; return false;`"
          >View {{ moreAppointments.length }} other appointment times</a
        >
      </div>

      <ul
        :id="`location-${store.properties.id}-more-appointments`"
        class="mb-0"
        style="display: none"
      >
        <li v-for="appointment in moreAppointments" :key="appointment.id">
          {{ appointment.time }}
          <span v-if="appointment.type">({{ appointment.type }})</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { DateTime } from "luxon";

export default {
  props: {
    store: {
      type: Object,
      required: true,
    },
  },

  head() {
    return {
      bodyAttrs: {
        class: this.withReload ? "has-navbar-with-reload" : "",
      },
    };
  },

  computed: {
    initialAppointments() {
      if (!this.store.properties.appointments) {
        return [];
      }

      return this.normalizeAppointments(
        this.store.properties.appointments.slice(0, 5)
      );
    },

    moreAppointments() {
      if (!this.store.properties.appointments) {
        return [];
      }

      return this.normalizeAppointments(
        this.store.properties.appointments.slice(5)
      );
    },

    storeVaccineTypes() {
      let storeVaccineTypes;

      if (this.store.properties.appointment_vaccine_types) {
        const types = [];
        if (this.store.properties.appointment_vaccine_types.jj) {
          types.push("Johnson & Johnson");
        }
        if (this.store.properties.appointment_vaccine_types.moderna) {
          types.push("Moderna");
        }
        if (this.store.properties.appointment_vaccine_types.pfizer) {
          types.push("Pfizer");
        }

        if (types.length > 0) {
          storeVaccineTypes = types.join(", ");
        }
      }

      return storeVaccineTypes;
    },
  },

  methods: {
    formatTime(time) {
      return DateTime.fromISO(time).toLocaleString({
        month: "numeric",
        day: "numeric",
        year: "numeric",
        hour: "numeric",
        minute: "numeric",
        timeZoneName: "short",
        timeZone: this.store.properties.time_zone,
      });
    },

    formatDate(date) {
      return DateTime.fromISO(date).toLocaleString({
        month: "numeric",
        day: "numeric",
        year: "numeric",
        timeZone: this.store.properties.time_zone,
      });
    },

    normalizeAppointments(appointments) {
      return appointments.map((appointment) => {
        let normalized;

        if (appointment.time && appointment.type) {
          let { type } = appointment;
          switch (type) {
            case "both_doses":
              type = "First Dose";
              break;
            case "second_dose_moderna":
              type = "Second Dose Only - Moderna";
              break;
            case "second_dose_pfizer":
              type = "Second Dose Only - Pfizer";
              break;
            default:
              type = appointment.type;
              break;
          }

          normalized = {
            time: this.formatTime(appointment.time),
            type,
          };
        } else if (appointment.date && appointment.type) {
          normalized = {
            time: this.formatDate(appointment.date),
            type: appointment.type,
          };
        } else if (appointment.date) {
          normalized = {
            time: this.formatDate(appointment.date),
            type: null,
          };
        } else if (appointment.time) {
          normalized = {
            time: this.formatTime(appointment.time),
            type: null,
          };
        } else {
          normalized = {
            time: this.formatTime(appointment),
            type: null,
          };
        }

        normalized.id = `${normalized.time}-${normalized.type}`;

        return normalized;
      });
    },
  },
};
</script>

<style></style>
