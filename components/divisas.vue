<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-text>
            <ValidationObserver ref="observerDivisas">
              <form ref="form">
                <ValidationProvider
                  v-slot="{ errors }"
                  name="Monto"
                  rules="required|numeric"
                >
                  <v-text-field
                    v-model="amount"
                    label="Monto"
                    placeholder="Escribe el monto"
                    type="number"
                    append-icon="mdi-currency-usd"
                    :error-messages="errors"
                  />
                </ValidationProvider>
                <ValidationProvider
                  v-slot="{ errors }"
                  name="Moneda origen"
                  rules="required"
                >
                  <v-autocomplete
                    v-model="selectedOrigin"
                    label="Moneda origen"
                    :items="items"
                    :error-messages="errors"
                  />
                </ValidationProvider>
                <ValidationProvider
                  v-slot="{ errors }"
                  name="Moneda Objetivo"
                  rules="required"
                >
                  <v-autocomplete
                    v-model="selectedObjetive"
                    label="Moneda Objetivo"
                    :items="items"
                    :error-messages="errors"
                  />
                </ValidationProvider>
              </form>
              <div class="text-center">
                <v-btn color="success" :disabled="isConvert" @click="convertir"
                  >Convertir</v-btn
                >
              </div>
            </ValidationObserver>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col>
        <div v-if="convert.length > 0" class="text-center contain-convert">
          <span>
            {{ convert }}
          </span>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import swal from "sweetalert";
export default {
  name: "DivisasComponent",
  data() {
    return {
      items: [],
      itemsOrigen: [],
      selectedOrigin: "",
      selectedObjetive: "",
      amount: 0,
      mountConverted: {},
      exist: false,
      convert: "",
      historial: [],
      clear: false,
      format: new Intl.NumberFormat()
    };
  },
  async created() {
    await this.getNameDivisas();
    await this.getListDivisas();
  },
  watch: {
    clear(val) {
      if (val) {
        this.selectedOrigin = "";
        this.selectedObjetive = "";
        this.amount = "";
        this.clear = false;
        this.$refs.observerDivisas.reset();
        this.$refs.form.reset();
      }
    }
  },
  methods: {
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
        });
    },
    async getListDivisas() {
      await this.$axios
        .get(
          "https://openexchangerates.org/api/latest.json?app_id=2eddb9945017483bad6b5a48043eeea4"
        )
        .then(resp => {
          const rates = resp.data.rates;
          const ratesKey = Object.keys(rates);
          ratesKey.forEach(i => {
            this.itemsOrigen.find(item => {
              if (item.siglas == i) {
                item["rates"] = rates[i];
              }
            });
          });
        });
    },
    convertir() {
      let sourceRate;
      let targetRate;
      this.$refs.observerDivisas.validate().then(valid => {
        if (valid) {
          if (this.amount > 0) {
            this.itemsOrigen.find(item => {
              if (item.completeName == this.selectedOrigin) {
                sourceRate = item.rates;
                this.mountConverted["origin"] = item.siglas;
              }
              if (item.completeName == this.selectedObjetive) {
                targetRate = item.rates;
                this.mountConverted["objetive"] = item.siglas;
                this.mountConverted["name"] = item.name;
              }
            });
            const amount = new Intl.NumberFormat({
              style: "currency",
              currency: `${this.mountConverted.origin}`
            }).format(this.amount);
            this.mountConverted["amount"] = amount;
            const total = (this.amount / sourceRate) * targetRate;

            const totalFormat = new Intl.NumberFormat({
              style: "currency",
              currency: `${this.mountConverted.objetive}`
            }).format(total);
            this.mountConverted["total"] = totalFormat;
            this.convert = amount.concat(
              " ",
              this.mountConverted.origin,
              ` = ${this.mountConverted.total}`,
              " ",
              this.mountConverted.objetive
            );
            this.mountConverted["covert"] = this.convert;
            this.clear = true;
          } else {
            swal("Error", "El monto debe ser mayor a 0", "error");
          }
        } else {
          swal("Error", "Los campos deben ser validos", "error");
        }
      });
    }
  },
  computed: {
    isConvert() {
      let status = true;
      if (
        this.amount != "" &&
        this.amount > 0 &&
        this.selectedOrigin.length > 0 &&
        this.selectedObjetive.length > 0
      ) {
        status = false;
      }
      return status;
    }
  }
};
</script>
<style>
.contain-convert {
  border: 1px solid black;
  height: 40px;
  border-radius: 5px;
  /* background-color:rgba(0, 0, 0, 0.6) */
}
.contain-convert > span {
  line-height: 40px;
}
</style>
