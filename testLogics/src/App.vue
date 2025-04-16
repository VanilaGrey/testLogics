<script setup>
import { ref, watch } from "vue";

// Вводимые значения
const priceInput = ref("");
const qtyInput = ref("");
const amountInput = ref("");

// Числовые значения
const price = ref(0);
const qty = ref(0);
const amount = ref(0);

// Общие значения
const events = ref([]);
const lastChanged = ref(null);
const debounceTimeout = ref(null);
const localStorageState = ref("");
const counter = ref(parseInt(localStorage.getItem("counter")) || 1);

// Функция обновления localStorageState
function updateLocalStorageState() {
  const data = {
    counter: localStorage.getItem("counter") || null,
    price: localStorage.getItem("price") || null,
    qty: localStorage.getItem("qty") || null,
    amount: localStorage.getItem("amount") || null,
  };
  localStorageState.value = JSON.stringify(data);
}

// Сразу вызываем при загрузке
updateLocalStorageState();

// Лог событий
function logEvent(message, type = "default") {
  events.value.unshift({
    time: new Date().toLocaleTimeString(),
    message,
    type,
  });
}
// Функция пересчета значений
function updateValues() {
  const parsedPrice = parseFloat(priceInput.value);
  const parsedQty = parseFloat(qtyInput.value);
  const parsedAmount = parseFloat(amountInput.value);

  if (lastChanged.value === "price") {
    price.value = isNaN(parsedPrice) ? 0 : parsedPrice;
    if (!isNaN(parsedQty)) {
      qty.value = parsedQty;
      amount.value = +(price.value * qty.value).toFixed(2);
      amountInput.value = amount.value;
    }
  } else if (lastChanged.value === "qty") {
    qty.value = isNaN(parsedQty) ? 0 : parsedQty;
    if (!isNaN(parsedPrice)) {
      price.value = parsedPrice;
      amount.value = +(price.value * qty.value).toFixed(2);
      amountInput.value = amount.value;
    }
  } else if (lastChanged.value === "amount") {
    amount.value = isNaN(parsedAmount) ? 0 : parsedAmount;

    if (!isNaN(parsedPrice) && parsedPrice !== 0) {
      qty.value = +(parsedAmount / parsedPrice).toFixed(2);
      qtyInput.value = qty.value;
    } else if (!isNaN(parsedQty) && parsedQty !== 0) {
      price.value = +(parsedAmount / parsedQty).toFixed(2);
      priceInput.value = price.value;
    }
  }
}

// дебаунс функция
function debounceUpdate(field) {
  lastChanged.value = field;
  logEvent(`Изменено поле ${field}`);

  if (debounceTimeout.value) clearTimeout(debounceTimeout.value);
  debounceTimeout.value = setTimeout(() => {
    updateValues();
  }, 300);
}

// Вотчеры для значений
watch(priceInput, () => debounceUpdate("price"));
watch(qtyInput, () => debounceUpdate("qty"));
watch(amountInput, () => debounceUpdate("amount"));

// Обработка отправки
function handleSubmit() {
  const payload = {
    price: price.value,
    qty: qty.value,
    amount: amount.value,
    counter: counter.value,
  };

  logEvent(`Нажата кнопка отправки. Отправляем: ${JSON.stringify(payload)}`);
  logEvent(`Состояние localStorage до отправки: ${localStorageState.value}`);

  setTimeout(() => {
    const isEven = payload.amount % 2 === 0;
    if (isEven) {
      localStorage.setItem("price", payload.price);
      localStorage.setItem("qty", payload.qty);
      localStorage.setItem("amount", payload.amount);
      localStorage.setItem("counter", payload.counter);

      counter.value += 1;
      localStorage.setItem("counter", counter.value); // для синхронизации localStorage с актуальным состоянием

      updateLocalStorageState(); // обновляем отображение
    }

    const backendResponse = { success: isEven };
    logEvent(
      `Ответ от сервера: ${JSON.stringify(
        backendResponse
      )} | Состояние localStorage: ${localStorageState.value}`,
      isEven ? "success" : "error"
    );
  }, 1000);
}
</script>

<template>
  <div class="main">
    <!-- инпуты и кнопка -->
    <div class="main__header">
      <div class="main__block">
        <label>Цена: <input v-model="priceInput" /></label>
        <span> Цена: {{ price }}</span>
      </div>
      <div class="main__block">
        <label>Кол-во: <input v-model="qtyInput" /> </label>
        <span>Кол-во: {{ qty }}</span>
      </div>
      <div class="main__block">
        <label>Сумма: <input v-model="amountInput" /> </label>
        <span>Сумма: {{ amount }}</span>
      </div>
      <div class="main__block">
        <button @click="handleSubmit">Отправить</button>
        <span>localStorage: {{ localStorageState }}</span>
      </div>
    </div>

    <!-- Лог событий -->
    <div class="main__container">
      <h3>События</h3>
      <div :class="['main__logs', { 'main__logs--bordered': events.length }]">
        <div
          v-for="(event, index) in events"
          :key="index"
          :class="['main__log', event.type, index % 2 === 0 ? 'even' : 'odd']"
        >
          [{{ event.time }}] {{ event.message }}
        </div>
      </div>
    </div>
  </div>
</template>

\
<style lang="scss" scoped>
.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
  width: max-content;
  margin: 0 auto;
  padding: 100px;

  &__header {
    display: flex;
    gap: 30px;
  }

  &__block {
    display: flex;
    flex-direction: column;
    gap: 30px;
  }

  &__container {
    align-self: start;
  }

  &__logs {
    overflow: auto;
    max-height: 300px;
    max-width: 1200px;

    &--bordered {
      border: 1px solid grey;
    }
  }

  &__log {
    padding: 8px 12px;
    border-radius: 6px;
    transition: background-color 0.2s;

    &.even {
      background-color: #f5f5f5;
    }

    &.odd {
      background-color: #eaeaea;
    }

    &.success {
      border-left: 4px solid #4caf50;
      background-color: #e6f4ea;
      color: #2e7d32;
    }

    &.error {
      border-left: 4px solid #f44336;
      background-color: #fdecea;
      color: #b71c1c;
    }
  }
}
</style>
