<template>
  <v-container>
    <v-card>
      <v-card-title>
        <v-row>
          <v-col cols="12" sm="12" md="6">
            <h4>Fecha de consulta: {{ today }}</h4>
          </v-col>
          <v-col cols="12" md="6" sm="12">
            <v-text-field
              v-model="search"
              outlined
              dense
              append-icon="mdi-magnify"
              placeholder="Search..."
            />
          </v-col>
        </v-row>
      </v-card-title>
      <v-col cols="12">
        <v-data-table :headers="headers" :items="items" :search="search" />
      </v-col>
    </v-card>
  </v-container>
</template>
<script>
export default {
  name: "History",
  async created() {
    await this.getNameDivisas();
  },
  data() {
    return {
      today: "",
      search: "",
      headers: [
        {
          text: "Nombre",
          align: "start",
          sortable: false,
          value: "name"
        },
        {
          text: "Abreviatura",
          align: "start",
          sortable: false,
          value: "siglas"
        },
        {
          text: "Precio",
          align: "start",
          sortable: false,
          value: "precio"
        }
      ],
      itemsOrigen: [],
      items: []
    };
  },
  methods: {
    getDate() {
      var today = new Date();
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0");
      var yyyy = today.getFullYear();

      today = yyyy + "-" + mm + "-" + dd;
      this.today = today;
      this.$axios
        .get(
          `https://openexchangerates.org/api/historical/${today}.json?app_id=2eddb9945017483bad6b5a48043eeea4&base=USD`
        )
        .then(resp => {
          const rates = resp.data.rates;
          const ratesKey = Object.keys(rates);
          ratesKey.forEach(i => {
            this.itemsOrigen.find(item => {
              if (item.siglas == i) {
                item["precio"] = rates[i].toString();
              }
            });
          });
          this.items = this.itemsOrigen;
        });
    },
    getNameDivisas() {
      this.$axios
        .get(
          "https://openexchangerates.org/api/currencies.json?app_id=2eddb9945017483bad6b5a48043eeea4"
        )
        .then(resp => {
          const data = resp.data;
          const keys = Object.keys(data);
          keys.forEach(i => {
            const completeName = data[i].concat(" ", `(${i})`);
            this.items.push(completeName);
            this.itemsOrigen.push({
              completeName: completeName,
              name: data[i],
              siglas: i
            });
          });
          this.getDate();
        });
    }
  }
};
</script>
