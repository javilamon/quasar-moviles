<template>
  <q-page class="flex flex-center q-pa-md">
    <q-card class="q-pa-md" style="max-width: 400px; width: 100%">
      <!-- Cuadros de fecha -->
      <q-card-section>
        <div class="q-mb-xs">
          <q-input
            v-model="fechaInicial"
            type="date"
            label="Fecha Inicial"
            outlined
            dense
            :class="{
              'input-valid': fechaInicialValida,
              'input-invalid': !fechaInicialValida,
            }"
            @click="showFechaInicial = true"
          />
        </div>
        <div class="q-mb-xs">
          <q-input
            v-model="fechaFinal"
            type="date"
            label="Fecha Final"
            outlined
            dense
            :class="{
              'input-valid': fechaFinalValida,
              'input-invalid': !fechaFinalValida,
            }"
            @click="showFechaFinal = true"
          />
        </div>
      </q-card-section>

      <!-- Toggles -->
      <q-card-section>
        <div class="q-mb-xs">
          <q-toggle
            v-model="selected.todos"
            label="Todos"
            val="todos"
            @change="selectAll"
          />
        </div>
        <div class="q-mb-xs">
          <q-toggle v-model="selected.fisico" label="Físico" val="fisico" />
        </div>
        <div class="q-mb-xs">
          <q-toggle
            v-model="selected.electronico"
            label="Electrónico"
            val="electronico"
          />
        </div>
        <div class="q-mb-xs">
          <q-toggle
            v-model="selected.efectivo"
            label="Efectivo"
            val="efectivo"
          />
        </div>
        <div class="q-mb-xs">
          <q-toggle v-model="selected.calle" label="Calle" val="calle" />
        </div>
        <div class="q-mb-xs">
          <q-toggle
            v-model="selected.anulados"
            label="Anulados"
            val="anulados"
          />
        </div>
      </q-card-section>

      <!-- Botones -->
      <q-card-section class="flex justify-center">
        <q-btn
          label="Consultar"
          color="primary"
          @click="consultar"
          class="q-mr-sm"
        />
        <q-btn label="Limpiar" color="secondary" @click="limpiar" />
      </q-card-section>
    </q-card>

    <!-- Diálogos de fecha -->
    <q-dialog
      v-model="showFechaInicial"
      @update:modelValue="showFechaInicial = $event"
      @hide="fechaInicialSeleccionada"
    >
      <q-card>
        <q-card-section>
          <q-date
            v-model="fechaInicial"
            @input="fechaInicialSeleccionada"
            mask="YYYY-MM-DD"
          />
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog
      v-model="showFechaFinal"
      @update:modelValue="showFechaFinal = $event"
      @hide="fechaFinalSeleccionada"
    >
      <q-card>
        <q-card-section>
          <q-date
            v-model="fechaFinal"
            @input="fechaFinalSeleccionada"
            mask="YYYY-MM-DD"
          />
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- Alerta de fecha incorrecta -->
    <q-dialog v-model="showAlertaFechaInvalida">
      <q-card>
        <q-card-section>
          <div class="text-h6">¡Fecha incorrecta!</div>
          <div>La fecha no puede ser posterior a la fecha actual.</div>
          <q-btn
            label="Aceptar"
            color="primary"
            @click="showAlertaFechaInvalida = false"
          />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import { ref, watch, computed } from "vue";

export default {
  name: "ConsultaPeriodo",
  setup() {
    const selected = ref({
      todos: false,
      fisico: false,
      electronico: false,
      efectivo: false,
      calle: false,
      anulados: false,
    });

    const selectAll = (value) => {
      if (value) {
        // Activar todos los toggles
        selected.value.fisico = true;
        selected.value.electronico = true;
        selected.value.efectivo = true;
        selected.value.calle = true;
        selected.value.anulados = true;
      } else {
        // Desactivar todos los toggles
        selected.value.fisico = false;
        selected.value.electronico = false;
        selected.value.efectivo = false;
        selected.value.calle = false;
        selected.value.anulados = false;
      }
    };

    // Watcher para cambiar los toggles individuales cuando se activa "Todos"
    watch(
      () => selected.value.todos,
      (newValue) => {
        if (newValue) {
          selected.value.fisico = true;
          selected.value.electronico = true;
          selected.value.efectivo = true;
          selected.value.calle = true;
          selected.value.anulados = true;
        } else {
          selected.value.fisico = false;
          selected.value.electronico = false;
          selected.value.efectivo = false;
          selected.value.calle = false;
          selected.value.anulados = false;
        }
      }
    );

    const fechaInicial = ref("");
    const fechaFinal = ref("");
    const showFechaInicial = ref(false);
    const showFechaFinal = ref(false);
    const showAlertaFechaInvalida = ref(false);

    const fechaInicialValida = computed(() => {
      return fechaInicial.value !== "";
    });

    const fechaFinalValida = computed(() => {
      return fechaFinal.value !== "";
    });

    const consultar = () => {
      if (!fechaInicialValida.value || !fechaFinalValida.value) {
        // Mostrar alerta si alguna de las fechas no es válida
        return;
      }

      // Validar si la fecha final es posterior a la fecha inicial
      const fechaInicio = new Date(fechaInicial.value);
      const fechaFin = new Date(fechaFinal.value);

      if (fechaFin < fechaInicio) {
        console.log("Fecha final debe ser posterior a la fecha inicial");
        return;
      }

      // Validar si alguna de las fechas es posterior a hoy
      const fechaHoy = new Date();
      if (fechaInicio > fechaHoy || fechaFin > fechaHoy) {
        showAlertaFechaInvalida.value = true;
        return;
      }

      const seleccionados = [];
      if (selected.value.todos) seleccionados.push("Todos");
      if (selected.value.fisico) seleccionados.push("Físico");
      if (selected.value.electronico) seleccionados.push("Electrónico");
      if (selected.value.efectivo) seleccionados.push("Efectivo");
      if (selected.value.calle) seleccionados.push("Calle");
      if (selected.value.anulados) seleccionados.push("Anulados");

      console.log("Filtros seleccionados:", seleccionados);
      console.log("Fechas seleccionadas:", {
        fechaInicial: fechaInicial.value,
        fechaFinal: fechaFinal.value,
      });
    };

    const limpiar = () => {
      fechaInicial.value = "";
      fechaFinal.value = "";
      selected.value.todos = false;
      selected.value.fisico = false;
      selected.value.electronico = false;
      selected.value.efectivo = false;
      selected.value.calle = false;
      selected.value.anulados = false;
    };

    const fechaInicialSeleccionada = () => {
      showFechaInicial.value = false;
    };

    const fechaFinalSeleccionada = () => {
      showFechaFinal.value = false;
    };

    return {
      selected,
      selectAll,
      fechaInicial,
      fechaFinal,
      showFechaInicial,
      showFechaFinal,
      fechaInicialValida,
      fechaFinalValida,
      consultar,
      limpiar,
      fechaInicialSeleccionada,
      fechaFinalSeleccionada,
      showAlertaFechaInvalida,
    };
  },
};
</script>

<style scoped>
.q-page {
  height: 100vh;
}

.input-invalid input {
  border-color: red !important;
}
</style>
