<template>
  <section class="page lg:p-10 bg-black text-white min-h-screen">
    <div class="max-w-4xl mx-auto">
      <h2 class="text-2xl lg:text-3xl font-bold mb-4">Uchburchak yuzini — Burchak Sinusi yordamida</h2>
      <p class="mb-4 text-menu-text">Formula: <code class="font-mono">A = ½ · a · b · sin(C)</code></p>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 items-start">
        <!-- Controls -->
        <div class="p-5 bg-gray-900 rounded-2xl shadow-lg">
          <h3 class="font-semibold mb-3">Parametrlar</h3>

          <label class="block mb-3">
            <span class="text-sm text-menu-text">a (tom uzunligi)</span>
            <input v-model.number="a" type="number" min="0" step="0.01" class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
          </label>

          <label class="block mb-3">
            <span class="text-sm text-menu-text">b (tom uzunligi)</span>
            <input v-model.number="b" type="number" min="0" step="0.01" class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
          </label>

          <label class="block mb-3">
            <span class="text-sm text-menu-text">C (ichki burchak, darajada)</span>
            <input v-model.number="C" type="number" min="0" max="180" step="0.1" class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
          </label>

          <div class="flex gap-3 mt-2">
            <button @click="calcArea" class="px-4 py-2 rounded-xl border border-neon-blue hover:opacity-90">Hisobla</button>
            <button @click="resetDefaults" class="px-4 py-2 rounded-xl bg-white/5">Reset</button>
          </div>

          <div class="mt-4 p-3 bg-black/30 rounded">
            <p class="text-sm text-menu-text">Natija:</p>
            <p class="text-2xl font-bold">{{ areaDisplay }}</p>
            <p v-if="note" class="mt-2 text-sm text-menu-text">{{ note }}</p>
          </div>

          <div class="mt-4 text-xs text-menu-text">
            <p>Misol: a = 10, b = 6, C = 60° → A ≈ 25.98</p>
          </div>
        </div>

        <!-- Visualization / Explanation -->
        <div class="p-5 bg-gray-900 rounded-2xl shadow-lg">
          <h3 class="font-semibold mb-3">Tushuntirish va ko'rinish</h3>
          <p class="text-menu-text mb-4">Agar ikkita tom va ularning orasidagi burchak ma'lum bo'lsa, balandlikni b · sin(C) yoki a · sin(B) orqali topib, yuzani ½·base·height formulasiga qo'yamiz.</p>

          <!-- SVG triangle (simple, responsive) -->
          <div class="w-full flex justify-center mb-3">
            <svg :width="svgWidth" :height="svgHeight" viewBox="0 0 200 140" class="max-w-full">
              <polygon :points="svgPoints" fill="rgba(255,255,255,0.03)" stroke="rgba(255,255,255,0.12)" stroke-width="2" />
              <!-- labels -->
              <text x="10" y="130" class="text-sm" fill="#B6C2D1">a: {{ a }}</text>
              <text x="150" y="130" class="text-sm" fill="#B6C2D1">b: {{ b }}</text>
              <text x="92" y="35" class="text-sm" fill="#FFD6A5">C: {{ C }}°</text>
            </svg>
          </div>

          <p class="text-sm text-menu-text">Ishlatish: qiymatlarni kiriting va <em>Hisobla</em> tugmasini bosing. Agar GSAP o'rnatilgan bo'lsa, natija yonidagi kartada silliq animatsiya ishlaydi.</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import DevConfig from '~/developer.json';
// Optional: uncomment the next line and run `npm i gsap` to enable entrance animations
// import { gsap } from 'gsap';

export default {
    data() {
        return {
            a: 10,
            b: 6,
            C: 60,
            area: null,
            note: ''
        }
    },
    setup() {
        return { config: DevConfig }
    },
    computed: {
        areaDisplay() {
            return this.area === null ? '—' : (Math.round(this.area * 100) / 100) + ' birlik²'
        },
        // SVG helpers
        svgWidth() { return 320 },
        svgHeight() { return 220 },
        svgPoints() {
            // produce a simple triangle based on a,b,C for illustration only
            const A = this.a || 1
            const B = this.b || 1
            const Cdeg = Math.min(Math.max(this.C || 60, 1), 179)
            const Crad = Cdeg * Math.PI / 180
            // place side a on bottom from (10,130) to (190,130)
            // compute third vertex based on side lengths ratio (simplified scaling)
            const scale = 10
            const x1 = 10
            const y1 = 130
            const x2 = 190
            const y2 = 130
            // position apex using law of cos (cosC = (a^2 + b^2 - c^2)/(2ab)) but we don't have c; make heuristic using angle
            const midX = (x1 + x2) / 2
            const height = Math.min(80, Math.abs(Math.sin(Crad) * (A + B) * 0.7))
            const apexX = midX
            const apexY = y1 - height
            return `${x1},${y1} ${x2},${y2} ${apexX},${apexY}`
        }
    },
    methods: {
        calcArea() {
            if (!this.a || !this.b || this.C === null || this.C === undefined) {
                this.note = 'Iltimos barcha parametrlarni kiriting.'
                this.area = null
                return
            }
            const rad = this.C * Math.PI / 180
            const computed = 0.5 * this.a * this.b * Math.sin(rad)
            if (Number.isNaN(computed) || !isFinite(computed)) {
                this.note = 'Kiritilgan qiymatlar bilan natija hosil bo\'lmadi.'
                this.area = null
            } else {
                this.area = computed
                this.note = ''
                // Optional animation when GSAP available
                // if (typeof gsap !== 'undefined') gsap.fromTo(this.$el.querySelector('.p-3'), {opacity:0, y:8}, {opacity:1, y:0, duration:0.6})
            }
        },
        resetDefaults() {
            this.a = 10; this.b = 6; this.C = 60; this.calcArea()
        }
    },
    mounted() {
        this.calcArea()
    }
}
</script>

<style scoped>
/* keep styles minimal; your global tailwind will handle most */
</style>
