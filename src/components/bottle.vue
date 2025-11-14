<script setup>
import { ref, computed } from "vue";

// Цвета металлов
const metals = {
  Золото: "#FFD700",
  Серебро: "#C0C0C0",
  Железо: "#B7410E",
  Медь: "#B87333",
  Платина: "#E5E4E2",
  Никель: "#AFAFAF",
  Бронза: "#CD7F32",
  Латунь: "#D4AF37",
  Алюминий: "#D9D9D9",
  Олово: "#C9C0BB",
  Цинк: "#BAC6C9",
  Углерод: "#2E2E2E",
  Иридий: "#E0E0E0"
};

// Совместимость
const metalResults = {
  "Золото-Серебро": {
    success: true,
    name: "Электрум",
    usage: "Ювелирное дело, декоративные покрытия, старинные монеты.",
    features: "Сочетает блеск золота и холодное сияние серебра."
  },
  "Медь-Олово": {
    success: true,
    name: "Бронза",
    usage: "Подшипники, инструменты, монеты.",
    features: "Высокая прочность и износостойкость."
  },
  "Медь-Цинк": {
    success: true,
    name: "Латунь",
    usage: "Инструменты, фурнитура, гильзы.",
    features: "Пластичный, коррозионностойкий."
  },
  "Алюминий-Медь": {
    success: true,
    name: "Дюралюминий",
    usage: "Авиастроение.",
    features: "Лёгкий и прочный."
  },
  "Железо-Углерод": {
    success: true,
    name: "Сталь",
    usage: "Везде.",
    features: "Прочная и долговечная."
  },
  "Никель-Железо": {
    success: true,
    name: "Перманлой",
    usage: "Экраны и электроника.",
    features: "Высокая магнитная проницаемость."
  },
  "Платина-Иридий": {
    success: true,
    name: "Платиново-иридиевый сплав",
    usage: "Эталон массы.",
    features: "Экстремальная устойчивость."
  },

  // Неудачные
  "Золото-Железо": { success: false, description: "Хрупкие соединения.", inefficiency: "Быстро разрушается." },
  "Серебро-Железо": { success: false, description: "Не образуют устойчивый сплав.", inefficiency: "Коррозия." },
  "Медь-Алюминий": { success: false, description: "Гальваническая коррозия.", inefficiency: "Разрушается при нагреве." },
  "Никель-Алюминий": { success: false, description: "Нужна точная температура.", inefficiency: "Образует хрупкие фазы." },
  "Железо-Медь": { success: false, description: "Плохое смачивание.", inefficiency: "Расслоение." },
  "Платина-Алюминий": { success: false, description: "Хрупкий слой.", inefficiency: "Трескается." }
};

const leftMetal = ref("");
const rightMetal = ref("");
const isMixed = ref(false);
const isHeating = ref(false);
const isShaking = ref(false);
const showResult = ref(false);
const resultData = ref({});
const showSelectWindow = ref(false);
const selectSide = ref("");

const leftOptions = computed(() =>
  Object.keys(metals).filter((m) => m !== rightMetal.value)
);
const rightOptions = computed(() =>
  Object.keys(metals).filter((m) => m !== leftMetal.value)
);

const openSelect = (side) => {
  selectSide.value = side;
  showSelectWindow.value = true;
};

const selectMetal = (metal) => {
  if (selectSide.value === "left") leftMetal.value = metal;
  else rightMetal.value = metal;
  showSelectWindow.value = false;
  isMixed.value = false;
};

const startMix = () => {
  if (leftMetal.value && rightMetal.value) {
    isHeating.value = true;
    isMixed.value = true;
    isShaking.value = false;

    setTimeout(() => {
      isHeating.value = false;

      const key1 = `${leftMetal.value}-${rightMetal.value}`;
      const key2 = `${rightMetal.value}-${leftMetal.value}`;
      const data = metalResults[key1] || metalResults[key2] || {
        success: false,
        description: "Металлы несовместимы.",
        inefficiency: "Хрупкий и бесполезный."
      };

      resultData.value = data;
      showResult.value = true;

      // Тряска только при ошибке
      if (!data.success) {
        isShaking.value = true;
        setTimeout(() => (isShaking.value = false), 600);
      }

    }, 3000);
  }
};

const closeResult = () => { showResult.value = false; };
const clearMix = () => {
  leftMetal.value = "";
  rightMetal.value = "";
  isMixed.value = false;
  isHeating.value = false;
  isShaking.value = false;
  showResult.value = false;
  resultData.value = {};
};

const leftColor = computed(() => (leftMetal.value ? metals[leftMetal.value] : null));
const rightColor = computed(() => (rightMetal.value ? metals[rightMetal.value] : null));
</script>

<template>
  <div class="bottle-animation">
    <div class="bottle-container">
      <div
        class="bottle"
        :class="{ heating: isHeating, shaking: isShaking }"
        :style="{
          background: leftColor && rightColor
            ? `linear-gradient(to right, ${leftColor} 0%, ${leftColor} 50%, ${rightColor} 50%, ${rightColor} 100%)`
            : 'rgba(255,255,255,0.05)'
        }"
      >
        <div
          class="mix-overlay"
          :class="{ active: isMixed }"
          :style="{ '--left': leftColor, '--right': rightColor }"
        ></div>

        <div class="heat-overlay" :class="{ active: isHeating }"></div>
        <div class="gloss"></div>
      </div>

      <div class="buttons">
        <button @click="openSelect('left')">{{ leftMetal || "+" }}</button>
        <button @click="openSelect('right')">{{ rightMetal || "+" }}</button>
      </div>
    </div>

    <div class="animation-control">
      <button @click="startMix" :disabled="!leftColor || !rightColor">Старт</button>
      <button @click="clearMix">Очистить</button>
    </div>

    <!-- Выбор -->
    <div v-if="showSelectWindow" class="select-popup">
      <div class="select-content">
        <h3>Выберите металл</h3>
        <div class="metal-buttons">
          <button
            v-for="metal in (selectSide==='left'? leftOptions:rightOptions)"
            :key="metal"
            @click="selectMetal(metal)"
          >
            {{ metal }}
          </button>
        </div>
      </div>
    </div>

    <!-- Результат -->
    <div v-if="showResult" class="result-popup">
      <div class="result-content">
        <button class="close-btn" @click="closeResult">×</button>

        <div v-if="resultData.success">
          <h3>Успешное слияние!</h3>
          <p><b>Название:</b> {{ resultData.name }}</p>
          <p><b>Особенности:</b> {{ resultData.features }}</p>
          <p><b>Применение:</b> {{ resultData.usage }}</p>
        </div>

        <div v-else>
          <h3>Слияние неудачно</h3>
          <p><b>Причина:</b> {{ resultData.description }}</p>
          <p><b>Недостатки:</b> {{ resultData.inefficiency }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bottle-container { width:100%; position:relative; height:200px; display:flex; align-items:center; }
.bottle {
  position:relative; width:100%; height:200px;
  border-radius:200px; border:1px solid rgba(255,255,255,0.35);
  box-shadow: inset 0 0 15px rgba(255,255,255,0.18),
              inset 0 0 30px rgba(255,255,255,0.08),
              0 8px 25px rgba(0,0,0,0.25);
  overflow:hidden;
  transition: background 2s ease-in-out;
}

.shaking {
  animation: shake 0.6s ease-in-out;
}

@keyframes shake {
  0% { transform: translate(0,0) rotate(0deg); }
  25% { transform: translate(-2px,2px) rotate(-0.6deg); }
  50% { transform: translate(2px,-2px) rotate(0.6deg); }
  75% { transform: translate(-2px,2px) rotate(-0.4deg); }
  100% { transform: translate(0,0) rotate(0deg); }
}

.mix-overlay {
  position:absolute; top:-25%; bottom:-25%; left:-5%; right:-5%;
  opacity:0;
  background: linear-gradient(to right, var(--left) 0%, var(--left) 40%, var(--right) 60%, var(--right) 100%);
  filter: blur(14px) saturate(160%);
  transition: opacity 2s ease-in-out, filter 2s ease-in-out, transform 2s ease-in-out;
  pointer-events:none; border-radius:50%;
}
.mix-overlay.active { opacity:1; filter: blur(8px) saturate(150%); transform: scale(1.01); }

.heat-overlay {
  position:absolute; top:0; left:0; right:0; bottom:0;
  background: rgba(255,80,0,0);
  pointer-events:none; border-radius:200px;
  transition: background 3s ease-in-out;
}
.heat-overlay.active { background: rgba(255,80,0,0.15); }

.gloss {
  position:absolute; top:8%; left:12%; width:76%; height:18%;
  border-radius:50%;
  background: linear-gradient(to bottom, rgba(255,255,255,0.45), rgba(255,255,255,0.02));
  filter:blur(4px);
  pointer-events:none; mix-blend-mode:screen;
}

.buttons {
  position:absolute; width:100%; display:flex; justify-content:space-between;
  padding:0 40px;
}
.buttons button {
  width:50px; height:50px; border-radius:50%;
  background:rgba(255,255,255,0.12);
  border:1px solid rgba(255,255,255,0.25);
  color:#fff; font-size:24px; cursor:pointer;
}

.animation-control { display:flex; gap:10px; padding-top:18px; }
.animation-control button {
  padding:12px 18px; border-radius:10px;
  background:rgba(255,255,255,0.12);
  border:1px solid rgba(255,255,255,0.22);
  color:#fff; cursor:pointer;
}
.animation-control button:disabled { opacity:0.35; cursor:not-allowed; }

/* Выбор */
.select-popup {
  position:fixed; top:0; left:0; right:0; bottom:0;
  background:rgba(0,0,0,0.5);
  display:flex; align-items:center; justify-content:center;
  z-index:999;
}
.select-content {
  background:#1a1a1a; padding:20px; border-radius:12px;
  width:300px; text-align:center; color:#fff;
}
.metal-buttons { display:flex; flex-wrap:wrap; gap:10px; justify-content:center; }
.metal-buttons button {
  padding:8px 12px; border-radius:8px;
  background:rgba(255,255,255,0.12);
  border:1px solid rgba(255,255,255,0.22);
  color:#fff; cursor:pointer;
}

/* Результат */
.result-popup {
  position:fixed; top:0; left:0; right:0; bottom:0;
  background:rgba(0,0,0,0.5);
  display:flex; align-items:center; justify-content:center;
  z-index:9999;
}
.result-content {
  background:#1a1a1a; padding:25px; border-radius:12px;
  width:320px; text-align:center; color:#fff; position:relative;
}
.close-btn {
  position:absolute; top:8px; right:10px;
  background:transparent; border:none;
  font-size:22px; cursor:pointer; color:#fff;
}
</style>
