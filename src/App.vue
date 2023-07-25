<template>
  <div class="main-container container">
    <div class="text-center d-flex flex-column justify-content-around h-100">
      <section>
        <h1 class="title">Adivina el número</h1>
        <h2 class="subtitle mt-4 animate__animated animate__tada">
          {{ numberSelected ? currentNumber.label : "?" }}
        </h2>
      </section>

      <section>
        <section class="fw-bold" v-if="!numberSelected">
          <p class="description text-muted">
            ¿Estás listo para el desafío? Elige una opción.
          </p>
        </section>

        <!-- :class="[
              item.selected == null
                ? 'btn btn-outline-dark'
                : item.valid
                ? 'btn-success text-white border-success'
                : 'btn-danger text-white border-danger',
            ]" -->
        <div class="mt-4">
          <button
            type="button"
            class="option btn mx-3 mb-4 mb-md-0 animate__animated animate__flash"
            :class="{
              'btn-outline-dark': !item.used,
              'text-white': item.used,
              'btn-dark border-dark': (item.selected && !item.valid),
              'btn-danger border-danger': (!item.selected && !item.valid && item.used),
              'btn-success border-success': item.valid && item.used
            }"
            :disabled="item.disabled"
            v-for="item in numbersList"
            :key="item"
            @click="verifyNumber(item)"
          >
            {{ item.label }}
          </button>
        </div>

        <p class="message-finish fw-bold mt-5" v-html="message"></p>

        <div class="mt-5">
          <button
            type="button"
            class="btn btn-warning btn-lg text-white"
            @click="init"
          >
            Reiniciar 🗘
          </button>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";

import _ from "underscore";

export default {
  name: "App",
  setup() {
    let currentNumber = ref(0);
    let allNumbers = ref([]);
    let numbersList = ref([]);
    let message = ref("");

    const generateNumbers = () => {
      // Reiniciar listas
      allNumbers.value = []
      numbersList.value = []

      // Generar números del 1 al 100
      for (let i = 1; i < 100; i++) {
        // Crear un objeto por cada número de la lista, con propiedades de utilidad
        allNumbers.value.push({
          valid: null,
          label: i,
          selected: false, // Define que número eligió el usuario
          disabled: false,
          used: false
        });
      }

      // se utiliza el método "shuffle" para mezclar la lista de números de manera aleatoria
      allNumbers.value = _.shuffle(allNumbers.value);

      // Obtener 5 números aleatorios de la lista para mostrar al usuario
      numbersList.value = _.sample(allNumbers.value, 5);

      // Con el método "sample" se obtiene un elemento de la última lista de 5 elementos
      currentNumber.value = _.sample(numbersList.value, 1)[0];

      // Alternativa sin "_"
      // currentNumber.value = allNumbers[Math.floor(Math.random() * allNumbers.length)]
    };

    const verifyNumber = (number) => {
      if(number.used) {
        return
      }

      if (!number.selected) {
        // Cambiamos valores en ciertas propiedades para una mejor identificación
        numbersList.value.forEach((item) => {
          item.selected = false
          item.disabled = true
          item.used = true
        });

        // Asignamos en "true" el valor de la propiedad "selected" al item seleccionado por el usuario
        number.selected = true;
        number.disabled = false
      }

      if (parseInt(number.label) === parseInt(currentNumber.value.label)) {
        reproducirAudio("success");
        number.valid = true;
        message.value = "🏆 <br /> Genial, ganaste!";
      } else {
        reproducirAudio("error");
        number.valid = false;
        message.value = "Opps, mala elección 🙃 <br /> ¡Inténtalo nuevamente!";
      }
    };

    const reproducirAudio = (val) => {
      var audio = new Audio(require(`@/assets/${val}.mp3`));
      audio.play();
    };

    const numberSelected = computed(() =>
      numbersList.value.find((item) => item.selected)
    );

    const init = () => {
      message.value = null;

      generateNumbers();
    };

    // Inicializar juego
    init();

    return {
      currentNumber,
      numbersList,
      allNumbers,
      verifyNumber,
      init,
      message,
      numberSelected,
    };
  },
};
</script>

<style lang="scss">
.main-container {
  height: 100vh;

  @media screen and (min-width: 768px) {
    height: 60vh;
  }

  @media screen and (min-width: 992px) {
    height: 100vh;
  }
}

.title,
.subtitle,
.message-finish {
  font-family: "Bungee Inline", cursive;
}

.title {
  font-size: 2.5rem;
}

.subtitle {
  font-size: 5rem;
  color: #fdb735;
}

.message-finish {
  font-size: 1.3em;
}

.description {
  font-size: 1.1em;
  font-weight: 400;
}

.option {
  width: 75px;
  height: 75px;
  font-family: "Bungee Inline", cursive !important;
  font-size: 26px !important;
}
</style>
