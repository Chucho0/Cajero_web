<template>
  <div class="padre">
    <div>
      <p class="alerta">{{ alerta }}</p>
    </div>
    <div class="cajas" v-show="data_user">
      <p class="titulo">Cuenta Bancaria</p>
      <input class="caja" type="number" id="pre" placeholder="Ingresar saldo inicial" v-model="valor" />
      <input class="caja" type="text" id="user" placeholder="Ingresar usuario" v-model="user" />
      <input class="caja" type="text" id="contra" placeholder="Ingresar contraseña" v-model="contra" />
      <button class="button" @click="validar()">Agregar</button>
    </div>
    <div class="sistema" v-show="data_contened">
      <div class="informacion">
        <p class="textousuario">Usuario actual {{ user }}</p>
        <p class="textosaldo">Tu saldo es {{ puntuacion(valor) }}</p>
        <form @submit.prevent="realizarOperacion">
          <select class="registro" v-model="operacion">
            <option value="consignar">Consignar</option>
            <option value="retirar">Retirar</option>
          </select>
          <input class="caja" type="number" v-model="plus" placeholder="Ingrese un valor">
          <button class="button">Agregar</button>
        </form>
      </div>
      <div class="retiros" v-show="data_retiros">
        <p class="textoretiro">Resumen del retiro</p>
        <table>
          <tbody>
            <tr v-for="(cantidad, denominacion) in desgloseBilletes" :key="denominacion">
              <td>Entregado {{ cantidad }} billete(s) de {{ puntuacion(denominacion) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
let alerta = ref("");
function ocultaralerta() { setTimeout(() => { alerta.value = "" }, 5000) }
const puntuacion = (number) => {
  return new Intl.NumberFormat("es-CO", { style: "currency", currency: "COP" }).format(number);
};
let data_user = ref(true);
let data_contened = ref(false);
let data_retiros = ref(false);
const users = ref([]);
const valor = ref("");
const user = ref("");
const contra = ref("");
const plus = ref("");
const operacion = ref("");
const desgloseBilletes = ref({ 50000: 0, 20000: 0, 10000: 0 });
function cambiardivs() {
  data_user.value = false
  data_contened.value = true
}
const validar = () => {
  if (user.value === "") {
    alerta.value = "Nombre de usuario no puede estar vacio"
    ocultaralerta()
  } else if (valor.value === "") {
    alerta.value = "Debe ingresar un valor"
    ocultaralerta()
  } else if (contra.value === "") {
    alerta.value = "Ingrese una contraseña"
    ocultaralerta()
  } else {
    alerta.value = "Registro de usuario exitoso"
    ocultaralerta()
    users.value.push({
      nombre: user.value,
      contrasena: contra.value,
      saldo: valor.value
    })
    console.log(users.value);
    cambiardivs()
  }
}
const realizarOperacion = () => {
  if (operacion.value === "consignar") {
    consignar();
  } else if (operacion.value === "retirar") {
    pedirContrasena();
  }
};
const consignar = () => {
  if (isNaN(Number(plus.value)) || Number(plus.value) <= 0) {
    alerta.value = "Ingrese un valor válido para la consignación";
    ocultaralerta()
  } else {
    valor.value = Number(valor.value) + Number(plus.value);
    alerta.value = `Consignación exitosa. Nuevo saldo: ${puntuacion(valor.value)}`;
    ocultaralerta()
  }
};
const pedirContrasena = () => {
  const inputPassword = prompt("Ingrese su contraseña para retirar");
  if (inputPassword === users.value.find(u => u.nombre === user.value)?.contrasena) {
    retirar();
  } else {
    alerta.value= "Contraseña incorrecta. No se puede realizar el retiro";
    ocultaralerta()
  }
};
const retirar = () => {
  if (isNaN(Number(plus.value)) || Number(plus.value) <= 0) {
    alerta.value = "Ingrese un valor válido para el retiro";
    ocultaralerta()
  } else if (Number(plus.value) % 10000 !== 0) {
    alerta.value = "Solo se permiten retiros en múltiplos de 10.000";
    ocultaralerta()
  } else if (Number(plus.value) > Number(valor.value)) {
    alerta.value = "No tiene suficiente saldo para realizar el retiro";
    ocultaralerta()
  } else {
    valor.value = Number(valor.value) - Number(plus.value);
    alerta.value = `Retiro exitoso. Nuevo saldo: ${puntuacion(valor.value)}`;
    ocultaralerta()
    actualizarDesgloseBilletes(Number(plus.value));
  }
};
const actualizarDesgloseBilletes = (monto) => {
  desgloseBilletes.value = { 50000: 0, 20000: 0, 10000: 0 };
  data_retiros.value = true;
  while (monto >= 50000) {
    desgloseBilletes.value[50000]++;
    monto -= 50000;
  }
  while (monto >= 20000) {
    desgloseBilletes.value[20000]++;
    monto -= 20000;
  }
  while (monto >= 10000) {
    desgloseBilletes.value[10000]++;
    monto -= 10000;
  }
};
</script>

<style scoped>
.padre {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: gray;
  height: 100vh;
}

.cajas {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 100px;

}

.titulo {
  font-size: 30px;
  margin-bottom: 20px;
}

.caja {
  margin-bottom: 20px;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #000;
  width: 700px;
}

.button {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #000;
  background-color: #000;
  color: #fff;
  cursor: pointer;
}

.sistema {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 100px;
}

.informacion {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.textosaldo {
  font-size: 30px;
  margin-bottom: 20px;
}

.textousuario {
  font-size: 30px;
  margin-bottom: 20px;
}

.registro {
  margin-bottom: 20px;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #000;
}

.retiros {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
  width: 100%;
}

.textoretiro {
  font-size: 30px;
  margin-bottom: 20px;
}

table {
  border: 1px solid #000;
  border-collapse: collapse;
  width: 100%;
}

td {
  border: 1px solid #000;
  padding: 10px;
}

tr {
  background-color: #f2f2f2;
}

.alerta {
  font-size: 30px;
  text-align: center;
  margin-top: 20px;
}
</style>