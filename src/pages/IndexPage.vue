<template>
  <q-page class="flex flex-center">
    <q-card class="q-pa-md" style="width: 400px">
      <q-card-section>
        <div class="text-h6">Digite su número de teléfono celular</div>
      </q-card-section>

      <q-card-section>
        <q-input
          v-model="telefono"
          type="text"
          label="Número de teléfono celular"
          outlined
          dense
          :class="{ 'input-valid': esValido }"
          :rules="[validaTelefono]"
          maxlength="10"
          @keyup.enter="enviar"
        />
      </q-card-section>

      <q-card-section>
        <q-btn label="Enviar" color="primary" @click="enviar" />
      </q-card-section>
    </q-card>

    <q-dialog v-model="dialogoVisible">
      <q-card>
        <q-card-section class="text-h6"> ¡Es correcto! </q-card-section>
        <q-card-section v-if="empleado">
          <div>
            <strong>Identificación:</strong> {{ empleado.identificacion }}
          </div>
          <div><strong>Primer Nombre:</strong> {{ empleado.primerNombre }}</div>
          <div>
            <strong>Segundo Nombre:</strong> {{ empleado.segundoNombre }}
          </div>
          <div>
            <strong>Primer Apellido:</strong> {{ empleado.primerApellido }}
          </div>
          <div>
            <strong>Segundo Apellido:</strong> {{ empleado.segundoApellido }}
          </div>
          <div>
            <strong>Nombre Completo:</strong> {{ empleado.nomCompleto }}
          </div>
          <div>
            <strong>Código Operadora:</strong> {{ empleado.codOperadora }}
          </div>
          <div>
            <strong>Teléfono Celular:</strong> {{ empleado.telefonoCelular }}
          </div>
          <div>
            <strong>Teléfono Convencional:</strong>
            {{ empleado.telefonoConvencional }}
          </div>
          <div><strong>Correo:</strong> {{ empleado.correo }}</div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn
            flat
            label="OK"
            color="primary"
            @click="dialogoVisible = false"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { ref, computed } from "vue";
import { useQuasar } from "quasar";
import axios from "axios";

const $q = useQuasar();
const telefono = ref("");
const dialogoVisible = ref(false);
const empleado = ref(null);

const validaTelefono = (val) => {
  if (!/^\d{10}$/.test(val)) {
    return "El número de teléfono debe tener exactamente 10 dígitos";
  }
  return true;
};

const esValido = computed(() => {
  return /^\d{10}$/.test(telefono.value);
});

const enviar = async () => {
  if (esValido.value) {
    try {
      const response = await axios.get(
        `http://192.168.41.118:8080/api/andrea/empleados-andrea/${telefono.value}/`
      );
      empleado.value = response.data;
      dialogoVisible.value = true;
    } catch (error) {
      $q.notify({
        type: "negative",
        message: "Error al obtener los datos del empleado",
      });
    }
  } else {
    $q.notify({
      type: "negative",
      message: "El número de teléfono no es válido",
    });
  }
};
</script>

<style scoped>
.input-valid input {
  border-color: green !important;
}
</style>
