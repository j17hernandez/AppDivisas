<template>
  <v-container>
    <h2 class="text-center">Aplicación de divisas</h2>
    <v-row>
      <v-spacer></v-spacer>
      <v-col cols="10">
        <v-tabs>
          <v-tab>Conversión</v-tab>
          <v-tab>Historial</v-tab>
          <v-tab-item>
            <Divisas />
          </v-tab-item>
          <v-tab-item>
            <Historial />
          </v-tab-item>
        </v-tabs>
      </v-col>
      <v-spacer></v-spacer>
    </v-row>
  </v-container>
</template>
<script>
import Divisas from "@/components/divisas.vue";
import Historial from "@/components/historial.vue";
export default {
  name: "AppDivisas",
  components: {
    Divisas,
    Historial,
  }
};
</script>
