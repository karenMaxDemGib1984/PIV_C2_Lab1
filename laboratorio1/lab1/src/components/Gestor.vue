<script setup>
import { ref, computed } from 'vue'

const length      = ref(12)
const password    = ref('')
const toast       = ref(false)
const usage       = ref('')
const usageError  = ref('')
const history     = ref([])

const opts = ref({
  upper: true,
  lower: true,
  nums:  true,
  syms:  true,
})

function generate() {
  if (!validateUsage()) return

  const len = length.value
  let charset = ''
  if (opts.value.upper) charset += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  if (opts.value.lower) charset += 'abcdefghijklmnopqrstuvwxyz'
  if (opts.value.nums)  charset += '0123456789'
  if (opts.value.syms)  charset += '@#$%&*!?+-'
  if (!charset)         charset  = 'abcdefghijklmnopqrstuvwxyz'

  const arr = new Uint32Array(len)
  crypto.getRandomValues(arr)
  let pw = ''
  for (let i = 0; i < len; i++) pw += charset[arr[i] % charset.length]
  password.value = pw

  history.value.unshift({ usage: usage.value.trim(), pw, bits: entropyBits.value })
  if (history.value.length > 5) history.value.pop()
}

function validateUsage() {
  const val = usage.value.trim()
  if (!val) {
    usageError.value = 'Este campo es obligatorio antes de generar.'
    return false
  }
  if (val.length < 3) {
    usageError.value = 'Mínimo 3 caracteres (ej: "Banco", "Gmail").'
    return false
  }
  if (val.length > 40) {
    usageError.value = 'Máximo 40 caracteres.'
    return false
  }
  usageError.value = ''
  return true
}

const entropyBits = computed(() => {
  let pool = 0
  if (opts.value.upper) pool += 26
  if (opts.value.lower) pool += 26
  if (opts.value.nums)  pool += 10
  if (opts.value.syms)  pool += 10
  if (!pool) pool = 26
  return Math.round(length.value * Math.log2(pool))
})

const strengthInfo = computed(() => {
  const e = entropyBits.value
  if (e < 28) return { label: 'Muy débil',   pct: 15,  bar: '#FFAAB8', col: '#7A1028' }
  if (e < 36) return { label: 'Débil',       pct: 30,  bar: '#FFD8DF', col: '#A02040' }
  if (e < 60) return { label: 'Moderada',    pct: 55,  bar: '#FFD8DF', col: '#A02040' }
  if (e < 80) return { label: 'Muy fuerte',  pct: 80,  bar: '#A8DF8E', col: '#1A4A28' }
  return             { label: 'Excepcional', pct: 100, bar: '#A8DF8E', col: '#0D2E18' }
})

function copyPassword() {
  if (!password.value) return
  navigator.clipboard.writeText(password.value).then(() => {
    toast.value = true
    setTimeout(() => (toast.value = false), 1800)
  })
}

const checks = [
  { key: 'upper', label: 'Mayúsculas', cls: 'ci-upper' },
  { key: 'lower', label: 'Minúsculas', cls: 'ci-lower' },
  { key: 'nums',  label: 'Números',    cls: 'ci-nums'  },
  { key: 'syms',  label: 'Símbolos',   cls: 'ci-syms'  },
]

const usageSuggestions = ['Banco', 'Gmail', 'Netflix', 'Instagram', 'Trabajo']
</script>

<template>
  <div class="wrap">
    <div class="card">

      <!-- TOAST -->
      <div v-if="toast" class="toast">¡Copiado al portapapeles!</div>

      <div class="top-bar" />

      <div class="card-title">Generador de Contraseñas Seguras</div>
      <div class="card-subtitle">Contraseñas robustas al instante ✨</div>

      <!-- INPUT DE USO -->
      <div class="section-label">¿Para qué es esta contraseña? <span class="req">*</span></div>
      <div class="input-group">
        <input
          v-model="usage"
          type="text"
          maxlength="40"
          placeholder="Ej: Banco, Gmail, Netflix…"
          :class="['usage-input', { 'input-error': usageError }]"
          @input="usageError = ''"
        />
        <p v-if="usageError" class="error-msg">⚠ {{ usageError }}</p>
      </div>

      <!-- SUGERENCIAS -->
      <div class="suggestions">
        <span class="sug-label">Sugerencias:</span>
        <div class="sug-chips">
          <button
            v-for="sug in usageSuggestions"
            :key="sug"
            :class="['sug-btn', { 'sug-active': usage === sug }]"
            @click="usage = sug; usageError = ''"
          >{{ sug }}</button>
        </div>
      </div>

      <!-- CONTRASEÑA GENERADA -->
      <div class="section-label" style="margin-top:20px">Tu contraseña</div>
      <div class="pw-box" @click="copyPassword">
        <span v-if="password" class="pw-text">{{ password }}</span>
        <span v-else class="pw-placeholder">— genera una contraseña —</span>
        <div class="copy-btn">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor"
               stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
            <rect x="9" y="9" width="13" height="13" rx="2"/>
            <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
          </svg>
        </div>
      </div>

      <!-- BARRA DE FORTALEZA -->
      <div class="strength-bar-track">
        <div
          class="strength-bar-fill"
          :style="{ width: strengthInfo.pct + '%', background: strengthInfo.bar }"
        />
      </div>
      <div class="strength-row">
        <span :style="{ color: strengthInfo.col, fontWeight: 700 }">{{ strengthInfo.label }}</span>
        <span class="entropy-text">{{ entropyBits }} bits de entropía</span>
      </div>

      <!-- SLIDER -->
      <div class="slider-row">
        <span class="slider-lbl">Longitud</span>
        <span class="slider-val">{{ length }}</span>
      </div>
      <input
        type="range" min="6" max="32" step="1"
        v-model.number="length"
        @input="password && generate()"
      />

      <!-- CHECKBOXES -->
      <div class="section-label">Incluir</div>
      <div class="checks">
        <label
          v-for="c in checks"
          :key="c.key"
          class="check-item"
          :class="[c.cls, { active: opts[c.key] }]"
        >
          <input type="checkbox" v-model="opts[c.key]" @change="password && generate()" />
          <div class="check-dot" />
          {{ c.label }}
        </label>
      </div>

      <!-- BOTONES -->
      <button class="btn-primary" @click="generate">Generar nueva contraseña</button>
      <button class="btn-outline" :disabled="!password" @click="copyPassword">
        Copiar al portapapeles
      </button>

      <!-- HISTORIAL -->
      <div v-if="history.length > 0" class="history-section">
        <div class="section-label" style="margin-top:20px">Historial reciente</div>
        <div
          v-for="(item, idx) in history"
          :key="idx"
          class="history-item"
        >
          <span class="hist-usage">{{ item.usage }}</span>
          <span class="hist-pw">{{ item.pw }}</span>
          <span class="hist-bits">{{ item.bits }}b</span>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=DM+Mono:wght@400;500&display=swap');

* { box-sizing: border-box; }

.wrap {
  min-height: 100vh;
  background: #F0FFDF;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  font-family: 'Nunito', sans-serif;
}

.card {
  background: #ffffff;
  border-radius: 24px;
  padding: 28px;
  width: 100%;
  max-width: 420px;
  position: relative;
  border: 2px solid #A8DF8E;
  box-shadow: 0 6px 0 #A8DF8E;
  overflow: hidden;
}

.top-bar {
  height: 5px;
  border-radius: 10px;
  background: linear-gradient(90deg, #A8DF8E, #FFAAB8, #FFD8DF, #A8DF8E);
  margin-bottom: 22px;
}

.card-title    { font-size: 1.5rem; font-weight: 800; color: #1A4A28; margin-bottom: 3px; }
.card-subtitle { font-size: 0.78rem; color: #5A9A68; font-weight: 600; margin-bottom: 20px; }

.section-label {
  font-size: 0.62rem;
  font-weight: 700;
  letter-spacing: 0.14em;
  color: #C0304A;
  text-transform: uppercase;
  margin-bottom: 10px;
}

/* ── INPUT DE USO ── */
.input-group { margin-bottom: 10px; }

.usage-input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #A8DF8E;
  border-radius: 14px;
  font-family: 'Nunito', sans-serif;
  font-size: 0.88rem;
  font-weight: 600;
  color: #1A4A28;
  background: #F0FFDF;
  outline: none;
  transition: all 0.2s;
}
.usage-input::placeholder { color: #8DC97A; font-weight: 600; }
.usage-input:focus {
  border-color: #6EC858;
  background: #E4FFCF;
  box-shadow: 0 0 0 3px rgba(168, 223, 142, 0.35);
}
.input-error {
  border-color: #FFAAB8 !important;
  background: #FFF5F7 !important;
  box-shadow: 0 0 0 3px rgba(255, 170, 184, 0.3) !important;
}
.error-msg {
  font-size: 0.72rem;
  font-weight: 700;
  color: #C0304A;
  margin-top: 6px;
  padding-left: 4px;
}
.req { color: #C0304A; margin-left: 2px; }

/* ── SUGERENCIAS ── */
.suggestions {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 8px;
  margin-bottom: 4px;
  width: 100%;
}

.sug-label {
  font-size: 0.68rem;
  font-weight: 700;
  color: #5A9A68;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

/* Los chips van en fila y hacen wrap si no caben */
.sug-chips {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 7px;
  width: 100%;
}

.sug-btn {
  background: #fff;
  border: 2px solid #A8DF8E;
  border-radius: 20px;
  padding: 4px 12px;
  font-size: 0.76rem;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
  color: #1A4A28;
  transition: all 0.18s;
  white-space: nowrap;
}
.sug-btn:hover {
  background: #F0FFDF;
  border-color: #6EC858;
  transform: translateY(-1px);
}
.sug-active {
  background: #FFAAB8 !important;
  border-color: #FFAAB8 !important;
  color: #5A0A1E !important;
  box-shadow: 0 3px 0 #D06070;
}

/* ── PASSWORD BOX ── */
.pw-box {
  background: #F0FFDF;
  border: 2px solid #A8DF8E;
  border-radius: 16px;
  padding: 16px 18px;
  font-family: 'DM Mono', monospace;
  font-size: 0.93rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  cursor: pointer;
  transition: all 0.2s;
  margin-bottom: 14px;
  color: #1A4A28;
  font-weight: 500;
}
.pw-box:hover   { border-color: #6EC858; background: #E4FFCF; }
.pw-text        { flex: 1; word-break: break-all; line-height: 1.4; }
.pw-placeholder { flex: 1; color: #8DC97A; font-style: italic; font-size: 0.85rem; }

.copy-btn {
  width: 32px;
  height: 32px;
  border-radius: 10px;
  background: #FFAAB8;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  cursor: pointer;
  transition: all 0.2s;
  color: #7A1028;
}
.copy-btn:hover { background: #ff8fa3; transform: scale(1.08); }

/* ── STRENGTH BAR ── */
.strength-bar-track {
  height: 6px;
  background: #F0FFDF;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 8px;
  border: 1px solid #A8DF8E;
}
.strength-bar-fill {
  height: 100%;
  border-radius: 10px;
  transition: width 0.4s ease, background 0.4s ease;
}
.strength-row {
  display: flex;
  justify-content: space-between;
  font-family: 'DM Mono', monospace;
  font-size: 0.68rem;
  margin-bottom: 20px;
}
.entropy-text { color: #5A9A68; }

/* ── SLIDER ── */
.slider-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}
.slider-lbl { font-size: 0.83rem; color: #1A4A28; font-weight: 700; }
.slider-val {
  font-family: 'DM Mono', monospace;
  font-size: 0.83rem;
  background: #FFD8DF;
  color: #7A1028;
  padding: 3px 12px;
  border-radius: 20px;
  font-weight: 700;
}
input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  height: 6px;
  border-radius: 10px;
  outline: none;
  cursor: pointer;
  margin-bottom: 20px;
  background: linear-gradient(90deg, #A8DF8E, #FFAAB8);
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #fff;
  cursor: pointer;
  border: 3px solid #FFAAB8;
  box-shadow: 0 2px 6px rgba(255, 170, 184, 0.5);
}

/* ── CHIPS ── */
.checks {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 9px;
  margin-bottom: 22px;
}
.check-item {
  display: flex;
  align-items: center;
  gap: 9px;
  font-size: 0.82rem;
  font-weight: 700;
  cursor: pointer;
  user-select: none;
  padding: 11px 13px;
  border: 2px solid;
  border-radius: 14px;
  transition: all 0.18s;
}
.check-item input { display: none; }
.ci-upper        { border-color: #FFAAB8; background: #fff; color: #7A1028; }
.ci-lower        { border-color: #A8DF8E; background: #fff; color: #1A4A28; }
.ci-nums         { border-color: #FFAAB8; background: #fff; color: #7A1028; }
.ci-syms         { border-color: #A8DF8E; background: #fff; color: #1A4A28; }
.ci-upper.active { background: #FFAAB8;  border-color: #FFAAB8; color: #5A0A1E; }
.ci-lower.active { background: #A8DF8E;  border-color: #A8DF8E; color: #0D2E18; }
.ci-nums.active  { background: #FFD8DF;  border-color: #FFAAB8; color: #7A1028; }
.ci-syms.active  { background: #F0FFDF;  border-color: #A8DF8E; color: #1A4A28; }
.check-dot {
  width: 17px;
  height: 17px;
  border-radius: 50%;
  border: 2px solid currentColor;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 800;
}
.check-item.active .check-dot::after { content: '✓'; }

/* ── BOTONES ── */
.btn-primary {
  width: 100%;
  padding: 14px;
  border-radius: 16px;
  border: none;
  font-family: 'Nunito', sans-serif;
  font-size: 0.92rem;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.2s;
  background: #FFAAB8;
  color: #5A0A1E;
  margin-bottom: 10px;
  box-shadow: 0 4px 0 #D06070;
  letter-spacing: 0.01em;
}
.btn-primary:hover  { transform: translateY(-1px); box-shadow: 0 6px 0 #D06070; background: #ff95a8; }
.btn-primary:active { transform: translateY(2px);  box-shadow: 0 2px 0 #D06070; }

.btn-outline {
  width: 100%;
  padding: 13px;
  border-radius: 16px;
  border: 2px solid #A8DF8E;
  font-family: 'Nunito', sans-serif;
  font-size: 0.92rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  background: #F0FFDF;
  color: #1A4A28;
}
.btn-outline:hover    { background: #DDF8C8; border-color: #6EC858; }
.btn-outline:disabled { opacity: 0.4; cursor: not-allowed; }

/* ── TOAST ── */
.toast {
  position: absolute;
  top: 18px;
  right: 18px;
  background: #A8DF8E;
  color: #0D2E18;
  font-family: 'Nunito', sans-serif;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 6px 14px;
  border-radius: 20px;
  pointer-events: none;
  animation: fadeIn 0.2s ease;
}
@keyframes fadeIn { from { opacity: 0; transform: translateY(-4px); } to { opacity: 1; transform: none; } }

/* ── HISTORIAL ── */
.history-section { margin-top: 4px; }
.history-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #fafafa;
  border-radius: 10px;
  margin-bottom: 6px;
  border: 1.5px solid #F0FFDF;
}
.hist-usage {
  font-size: 0.68rem;
  font-weight: 800;
  color: #C0304A;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  min-width: 60px;
  flex-shrink: 0;
}
.hist-pw {
  font-family: 'DM Mono', monospace;
  font-size: 0.72rem;
  color: #444;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.hist-bits {
  font-size: 0.65rem;
  color: #A8DF8E;
  font-weight: 700;
  flex-shrink: 0;
}
</style>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=DM+Mono:wght@400;500&display=swap');

.wrap {
  min-height: 100vh;
  background: #F0FFDF;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  font-family: 'Nunito', sans-serif;
}

.card {
  background: #ffffff;
  border-radius: 24px;
  padding: 28px;
  width: 100%;
  max-width: 420px;
  position: relative;
  border: 2px solid #A8DF8E;
  box-shadow: 0 6px 0 #A8DF8E;
}

.top-bar {
  height: 5px;
  border-radius: 10px;
  background: linear-gradient(90deg, #A8DF8E, #FFAAB8, #FFD8DF, #A8DF8E);
  margin-bottom: 22px;
}

.card-title    { font-size: 1.5rem; font-weight: 800; color: #1A4A28; margin-bottom: 3px; }
.card-subtitle { font-size: 0.78rem; color: #5A9A68; font-weight: 600; margin-bottom: 20px; }

.section-label {
  font-size: 0.62rem;
  font-weight: 700;
  letter-spacing: 0.14em;
  color: #C0304A;
  text-transform: uppercase;
  margin-bottom: 10px;
}

/* PASSWORD BOX */
.pw-box {
  background: #F0FFDF;
  border: 2px solid #A8DF8E;
  border-radius: 16px;
  padding: 16px 18px;
  font-family: 'DM Mono', monospace;
  font-size: 0.93rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  cursor: pointer;
  transition: all 0.2s;
  margin-bottom: 14px;
  color: #1A4A28;
  font-weight: 500;
}
.pw-box:hover  { border-color: #6EC858; background: #E4FFCF; }
.pw-text       { flex: 1; word-break: break-all; line-height: 1.4; }

.copy-btn {
  width: 32px;
  height: 32px;
  border-radius: 10px;
  background: #FFAAB8;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  cursor: pointer;
  transition: all 0.2s;
  color: #7A1028;
}
.copy-btn:hover { background: #ff8fa3; transform: scale(1.08); }

/* STRENGTH BAR */
.strength-bar-track {
  height: 6px;
  background: #F0FFDF;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 8px;
  border: 1px solid #A8DF8E;
}
.strength-bar-fill {
  height: 100%;
  border-radius: 10px;
  transition: width 0.4s ease, background 0.4s ease;
}
.strength-row {
  display: flex;
  justify-content: space-between;
  font-family: 'DM Mono', monospace;
  font-size: 0.68rem;
  margin-bottom: 20px;
}

/* SLIDER */
.slider-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}
.slider-lbl { font-size: 0.83rem; color: #1A4A28; font-weight: 700; }
.slider-val {
  font-family: 'DM Mono', monospace;
  font-size: 0.83rem;
  background: #FFD8DF;
  color: #7A1028;
  padding: 3px 12px;
  border-radius: 20px;
  font-weight: 700;
}

input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  height: 6px;
  border-radius: 10px;
  outline: none;
  cursor: pointer;
  margin-bottom: 20px;
  background: linear-gradient(90deg, #A8DF8E, #FFAAB8);
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #fff;
  cursor: pointer;
  border: 3px solid #FFAAB8;
  box-shadow: 0 2px 6px rgba(255, 170, 184, 0.5);
}

/* CHIPS */
.checks {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 9px;
  margin-bottom: 22px;
}
.check-item {
  display: flex;
  align-items: center;
  gap: 9px;
  font-size: 0.82rem;
  font-weight: 700;
  cursor: pointer;
  user-select: none;
  padding: 11px 13px;
  border: 2px solid;
  border-radius: 14px;
  transition: all 0.18s;
}
.check-item input { display: none; }

.ci-upper        { border-color: #FFAAB8; background: #fff;    color: #7A1028; }
.ci-lower        { border-color: #A8DF8E; background: #fff;    color: #1A4A28; }
.ci-nums         { border-color: #FFAAB8; background: #fff;    color: #7A1028; }
.ci-syms         { border-color: #A8DF8E; background: #fff;    color: #1A4A28; }

.ci-upper.active { background: #FFAAB8;  border-color: #FFAAB8; color: #5A0A1E; }
.ci-lower.active { background: #A8DF8E;  border-color: #A8DF8E; color: #0D2E18; }
.ci-nums.active  { background: #FFD8DF;  border-color: #FFAAB8; color: #7A1028; }
.ci-syms.active  { background: #F0FFDF;  border-color: #A8DF8E; color: #1A4A28; }

.check-dot {
  width: 17px;
  height: 17px;
  border-radius: 50%;
  border: 2px solid currentColor;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 800;
}
.check-item.active .check-dot::after { content: '✓'; }

/* BUTTONS */
.btn-primary {
  width: 100%;
  padding: 14px;
  border-radius: 16px;
  border: none;
  font-family: 'Nunito', sans-serif;
  font-size: 0.92rem;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.2s;
  background: #FFAAB8;
  color: #5A0A1E;
  margin-bottom: 10px;
  box-shadow: 0 4px 0 #D06070;
  letter-spacing: 0.01em;
}
.btn-primary:hover  { transform: translateY(-1px); box-shadow: 0 6px 0 #D06070; background: #ff95a8; }
.btn-primary:active { transform: translateY(2px);  box-shadow: 0 2px 0 #D06070; }

.btn-outline {
  width: 100%;
  padding: 13px;
  border-radius: 16px;
  border: 2px solid #A8DF8E;
  font-family: 'Nunito', sans-serif;
  font-size: 0.92rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  background: #F0FFDF;
  color: #1A4A28;
}
.btn-outline:hover { background: #DDF8C8; border-color: #6EC858; }

/* TOAST */
.toast {
  position: absolute;
  top: 18px;
  right: 18px;
  background: #A8DF8E;
  color: #0D2E18;
  font-family: 'Nunito', sans-serif;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 6px 14px;
  border-radius: 20px;
  opacity: 0;
  transition: opacity 0.3s;
  pointer-events: none;
}
.toast.show { opacity: 1; }

/* ── INPUT DE USO ── */
.input-group {
  margin-bottom: 12px;
}

.usage-input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #A8DF8E;
  border-radius: 14px;
  font-family: 'Nunito', sans-serif;
  font-size: 0.88rem;
  font-weight: 600;
  color: #1A4A28;
  background: #F0FFDF;
  outline: none;
  transition: all 0.2s;
}

.usage-input::placeholder {
  color: #8DC97A;
  font-weight: 600;
}

.usage-input:focus {
  border-color: #6EC858;
  background: #E4FFCF;
  box-shadow: 0 0 0 3px rgba(168, 223, 142, 0.35);
}

.input-error {
  border-color: #FFAAB8 !important;
  background: #FFF5F7 !important;
  box-shadow: 0 0 0 3px rgba(255, 170, 184, 0.3) !important;
}

.error-msg {
  font-size: 0.72rem;
  font-weight: 700;
  color: #C0304A;
  margin-top: 6px;
  padding-left: 4px;
  font-family: 'Nunito', sans-serif;
}

.req {
  color: #C0304A;
  margin-left: 2px;
}

/* ── SUGERENCIAS ── */
.suggestions {
  display: flex;
  flex-direction: column;      /* ← apila label arriba, chips abajo */
  align-items: flex-start;
  gap: 8px;
  margin-bottom: 18px;
  width: 100%;                 /* ← no se sale del card */
}

.sug-label {
  font-size: 0.68rem;
  font-weight: 700;
  color: #5A9A68;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-right: 2px;
}

.sug-btn {
  background: #fff;
  border: 2px solid #A8DF8E;
  border-radius: 20px;
  padding: 4px 12px;
  font-size: 0.76rem;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  cursor: pointer;
  color: #1A4A28;
  transition: all 0.18s;
}

.sug-btn:hover {
  background: #F0FFDF;
  border-color: #6EC858;
  transform: translateY(-1px);
}

.sug-active {
  background: #FFAAB8 !important;
  border-color: #FFAAB8 !important;
  color: #5A0A1E !important;
  box-shadow: 0 3px 0 #D06070;
}
</style>
