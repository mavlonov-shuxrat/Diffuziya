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
  Латунь: "#D4AF37"
};

// Совместимость металлов
const metalResults = {
  "Золото-Серебро": {
    success: true,
    name: "Бижутерный сплав",
    usage: "Используется для украшений, ювелирных изделий."
  },
  "Золото-Железо": {
    success: false,
    description: "Образуются хрупкие соединения, сплав не прочный."
  },
  "Серебро-Железо": {
    success: false,
    description: "Коррозия и потеря прочности."
  },
  "Медь-Бронза": {
    success: true,
    name: "Бронзовый сплав",
    usage: "Используется в архитектуре, механизмах и монетах."
  },
  // Можно добавить больше комбинаций
};

const leftMetal = ref("");
const rightMetal = ref("");
const showLeft = ref(false);
const showRight = ref(false);
const isMixed = ref(false);
const isHeating = ref(false);
const showResult = ref(false);
const resultData = ref({});

const leftOptions = computed(() =>
  Object.keys(metals).filter((m) => m !== rightMetal.value)
);
const rightOptions = computed(() =>
  Object.keys(metals).filter((m) => m !== leftMetal.value)
);

const selectLeft = (metal) => {
  leftMetal.value = metal;
  showLeft.value = false;
  isMixed.value = false;
};

const selectRight = (metal) => {
  rightMetal.value = metal;
  showRight.value = false;
  isMixed.value = false;
};

const startMix = () => {
  if (leftMetal.value && rightMetal.value) {
    isHeating.value = true;
    isMixed.value = true;

    // Через 3 секунды прекращаем нагрев и показываем результат
    setTimeout(() => {
      isHeating.value = false;

      const key1 = `${leftMetal.value}-${rightMetal.value}`;
      const key2 = `${rightMetal.value}-${leftMetal.value}`;
      const data = metalResults[key1] || metalResults[key2] || {
        success: false,
        description: "Металы несовместимы, сплав не устойчив."
      };
      resultData.value = data;
      showResult.value = true;
    }, 3000);
  }
};

const closeResult = () => {
  showResult.value = false;
};

const clearMix = () => {
  leftMetal.value = "";
  rightMetal.value = "";
  isMixed.value = false;
  isHeating.value = false;
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
        :class="{ heating: isHeating }"
        :style="{
          background: leftColor && rightColor
            ? `linear-gradient(to right, ${leftColor} 0%, ${leftColor} 50%, ${rightColor} 50%, ${rightColor} 100%)`
            : 'rgba(255,255,255,0.05)'
        }"
      >
        <div
          class="mix-overlay"
          :class="{ active: isMixed }"
          :style="{
            '--left': leftColor,
            '--right': rightColor
          }"
        ></div>
        <div class="heat-overlay" :class="{ active: isHeating }"></div>
        <div class="gloss"></div>
      </div>

      <div class="buttons">
        <div class="dropdown-container">
          <button @click="showLeft = !showLeft">{{ leftMetal || "+" }}</button>
          <ul v-if="showLeft" class="dropdown">
            <li v-for="metal in leftOptions" :key="metal" @click="selectLeft(metal)">{{ metal }}</li>
          </ul>
        </div>

        <div class="dropdown-container">
          <button @click="showRight = !showRight">{{ rightMetal || "+" }}</button>
          <ul v-if="showRight" class="dropdown">
            <li v-for="metal in rightOptions" :key="metal" @click="selectRight(metal)">{{ metal }}</li>
          </ul>
        </div>
      </div>
    </div>

    <div class="animation-control">
      <button @click="startMix" :disabled="!leftColor || !rightColor">Старт</button>
      <button @click="clearMix">Очистить</button>
    </div>

    <!-- Всплывающее окно результата -->
    <div v-if="showResult" class="result-popup">
      <div class="result-content">
        <button class="close-btn" @click="closeResult">×</button>
        <div v-if="resultData.success">
          <h3>Успешное слияние!</h3>
          <p>Вы получили: <b>{{ resultData.name }}</b></p>
          <p>Применение: {{ resultData.usage }}</p>
        </div>
        <div v-else>
          <h3>Слияние неудачно</h3>
          <p>{{ resultData.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Основная бутылка и эффекты (как ранее) */
.bottle-container { width:100%; position:relative; height:200px; display:flex; align-items:center; }
.bottle { position:relative; width:100%; height:200px; border-radius:200px; border:1px solid rgba(255,255,255,0.35); box-shadow: inset 0 0 15px rgba(255,255,255,0.18), inset 0 0 30px rgba(255,255,255,0.08), 0 8px 25px rgba(0,0,0,0.25); overflow:hidden; transition: background 2s ease-in-out; }
.mix-overlay { position:absolute; top:-25%; bottom:-25%; left:-5%; right:-5%; opacity:0; background: linear-gradient(to right, var(--left) 0%, var(--left) 40%, var(--right) 60%, var(--right) 100%); filter: blur(14px) saturate(160%); transition: opacity 2s ease-in-out, filter 2s ease-in-out, transform 2s ease-in-out; pointer-events:none; border-radius:50%; }
.mix-overlay.active { opacity:1; filter: blur(8px) saturate(150%); transform: scale(1.01); }
.heating { animation: shakeHeating 0.6s ease-in-out infinite; }
.heat-overlay { position:absolute; top:0; left:0; right:0; bottom:0; background: rgba(255,80,0,0); pointer-events:none; border-radius:200px; transition: background 3s ease-in-out; }
.heat-overlay.active { background: rgba(255,80,0,0.15); }
@keyframes shakeHeating { 0% { transform: translate(0px,0px) rotate(0deg); } 25% { transform: translate(-1px,1px) rotate(-0.3deg); } 50% { transform: translate(1px,-1px) rotate(0.3deg); } 75% { transform: translate(-1px,1px) rotate(-0.2deg); } 100% { transform: translate(0,0) rotate(0deg); }
}
.gloss { position:absolute; top:8%; left:12%; width:76%; height:18%; border-radius:50%; background: linear-gradient(to bottom, rgba(255,255,255,0.45), rgba(255,255,255,0.02)); filter:blur(4px); pointer-events:none; mix-blend-mode:screen; }
.buttons { position:absolute; width:100%; display:flex; justify-content:space-between; padding:0 40px; }
.dropdown-container { position: relative; }
.buttons button { width:50px; height:50px; border-radius:50%; background-color: rgba(255,255,255,0.12); border:1px solid rgba(255,255,255,0.25); box-shadow:0 4px 20px rgba(0,0,0,0.2); cursor:pointer; font-size:25px; color:#fff; }
.dropdown { position:absolute; top:60px; left:0; background: rgba(255,255,255,0.12); border-radius:12px; backdrop-filter: blur(8px); list-style:none; padding:6px 0; width:140px; }
.dropdown li { padding:8px 12px; cursor:pointer; color:white; text-align:center; }
.dropdown li:hover { background: rgba(255,255,255,0.08); }
.animation-control { display:flex; gap:10px; padding-top:18px; }
.animation-control button { padding:12px 18px; border-radius:10px; background-color: rgba(255,255,255,0.12); border:1px solid rgba(255,255,255,0.22); color:#fff; cursor:pointer; }
.animation-control button:disabled { opacity:0.35; cursor:not-allowed; }

/* Всплывающее окно результата */
.result-popup {
  position: fixed;
  top:0; left:0; right:0; bottom:0;
  background: rgba(0,0,0,0.5);
  display:flex;
  align-items:center;
  justify-content:center;
  z-index:9999;
}
.result-content {
  background: #1a1a1a;
  padding: 25px;
  border-radius: 12px;
  width: 300px;
  text-align: center;
  color: #fff;
  position: relative;
}
.result-content h3 { margin-bottom: 10px; }
.close-btn {
  position:absolute;
  top:8px; right:10px;
  background:transparent;
  border:none;
  font-size:22px;
  cursor:pointer;
  color:#fff;
}
</style>
