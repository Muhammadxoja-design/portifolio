<template>
    <section class="page lg:p-10 bg-black text-white min-h-screen">
        <div class="max-w-4xl mx-auto">
            <h2 class="text-2xl lg:text-3xl font-bold mb-4">Sinuslar teoremasi — interaktiv yechuvchi</h2>
            <p class="mb-4 text-menu-text">Formula: <code class="font-mono">a / sin(A) = b / sin(B) = c / sin(C)</code>
            </p>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div class="p-5 bg-gray-900 rounded-2xl shadow-lg">
                    <h3 class="font-semibold mb-3">Berilganlar</h3>

                    <label class="block mb-2">a (ma'lum tomon)
                        <input v-model.number="a" type="number" min="0" step="0.01"
                            class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
                    </label>

                    <label class="block mb-2">A (a ga qarshi burchak, darajada)
                        <input v-model.number="A" type="number" min="0" max="180" step="0.1"
                            class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
                    </label>

                    <label class="block mb-2">Topilishi kerak: (tanlang)</label>
                    <select v-model="mode" class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700 mb-3">
                        <option value="side">Tomon (b yoki c)</option>
                        <option value="angle">Burchak (B yoki C)</option>
                    </select>

                    <div v-if="mode === 'side'">
                        <label class="block mb-2">Topiladigan tomon nomi
                            <select v-model="targetSideName"
                                class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700">
                                <option value="b">b</option>
                                <option value="c">c</option>
                            </select>
                        </label>
                        <label class="block mb-2">Target burchak (qarshi burchak) darajada
                            <input v-model.number="targetAngle" type="number" min="0" max="180" step="0.1"
                                class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
                        </label>
                    </div>

                    <div v-else>
                        <label class="block mb-2">Topiladigan burchak nomi
                            <select v-model="targetAngleName"
                                class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700">
                                <option value="B">B</option>
                                <option value="C">C</option>
                            </select>
                        </label>
                        <label class="block mb-2">Balandlik yoki tomon bilinishini tanlang (agar kerak)
                            <input v-model.number="targetSideKnown" type="number" min="0" step="0.01"
                                placeholder="(masalan b yoki c — agar kerak bo'lsa)"
                                class="w-full rounded px-3 py-2 mt-1 bg-black border border-gray-700" />
                        </label>
                    </div>

                    <div class="flex gap-3 mt-3">
                        <button @click="solve" class="px-4 py-2 rounded-xl border">Top</button>
                        <button @click="reset" class="px-4 py-2 rounded-xl bg-white/5">Reset</button>
                    </div>

                    <div class="mt-4 p-3 bg-black/30 rounded">
                        <p class="text-sm text-menu-text">Natija:</p>
                        <div class="mt-2">
                            <div v-if="resultText.length" class="text-sm" v-html="resultText"></div>
                            <p v-else class="text-menu-text">—</p>
                        </div>
                    </div>
                </div>

                <div class="p-5 bg-gray-900 rounded-2xl shadow-lg">
                    <h3 class="font-semibold mb-3">Qisqacha eslatma</h3>
                    <p class="text-menu-text mb-2">SSA holatida (ikki tomon va ularga tegishli bo'lmagan burchak) noaniq
                        holat mumkin: sinus qiymati ikkita burchakka mos keladi (x va 180° - x). Bu komponent ikkinchi
                        yechimni aniqlab beradi va foydalanuvchiga ko'rsatadi.</p>
                    <ul class="list-disc pl-5 text-menu-text text-sm">
                        <li>Agar sinX &gt; 1 — yechim yo'q (impossible).</li>
                        <li>Agar sinX = 1 — yagona yechim (right-angle).</li>
                        <li>Agar 0 &lt; sinX &lt; 1 — 1 yoki 2 yechim bo'lishi mumkin; bu komponent ikkalasini ham
                            hisoblaydi.</li>
                    </ul>
                </div>
            </div>

        </div>
    </section>
</template>

<script>
import DevConfig from '~/developer.json';

export default {
  data() {
    return {
      a: 8,
      A: 40,
      mode: 'side',
      targetSideName: 'b',
      targetAngle: 60,
      // angle mode
      targetAngleName: 'B',
      targetSideKnown: null,
      resultText: ''
    }
  },
  setup() {
    return { config: DevConfig }
  },
  methods: {
    reset() {
      this.a = 8; this.A = 40; this.mode = 'side'; this.targetSideName = 'b'; this.targetAngle = 60; this.targetAngleName = 'B'; this.targetSideKnown = null; this.resultText = ''
    },
    solve() {
      this.resultText = ''
      if (!this.a || !this.A) {
        this.resultText = 'Iltimos a va A qiymatlarini kiriting.'
        return
      }
      const radA = this.A * Math.PI / 180
      // base ratio
      const ratio = this.a / Math.sin(radA)

      if (this.mode === 'side') {
        if (!this.targetAngle && this.targetAngle !== 0) { this.resultText = 'Iltimos target burchakni kiriting.'; return }
        const radX = this.targetAngle * Math.PI / 180
        const sinX = Math.sin(radX)
        // x_side = ratio * sinX
        const xSide = ratio * sinX
        // ambiguous? If solving for side given targetAngle, it's straightforward (no ambiguity for side) — but if they gave angle and ask for side, it's direct.
        this.resultText = `<strong>${this.targetSideName}</strong> = ${ (Math.round(xSide*100)/100) } birlik. <br/><small>Hisoblash: ${this.targetSideName} = a·sin(${this.targetAngle}°)/sin(A)</small>`
        return
      }

      // mode === 'angle' — solving for unknown angle X given side name in targetSideKnown (if provided) is more complex — we handle SSA.
      if (this.mode === 'angle') {
        if (!this.targetSideKnown) { this.resultText = 'Iltimos target tomon qiymatini kiriting (masalan b yoki c soni).'; return }
        const xside = this.targetSideKnown
        // sinX = xside * sinA / a
        const sinX = xside * Math.sin(radA) / this.a
        if (sinX > 1 + 1e-9) {
          this.resultText = 'Imkonsiz: sin qiymati 1 dan katta — yechim yo\'q.'
          return
        }
        if (Math.abs(sinX - 1) < 1e-9) {
          const Xdeg = 90
          this.resultText = `Yagona yechim: ${this.targetAngleName} ≈ ${Xdeg}° (right angle).`;
          return
        }
        if (sinX < 0) {
          this.resultText = 'Imkonsiz: sin qiymati manfiy — bunday holat oddiy uchburchakda kamroq uchraydi.'
          return
        }
        // two possible angle solutions
        const X1 = Math.asin(sinX) * 180 / Math.PI
        const X2 = 180 - X1
        // check if X2 + A < 180 to be valid triangle
        const sol = []
        if (A + X1 < 180) sol.push(X1)
        if (A + X2 < 180 && Math.abs(X2 - X1) > 1e-6) sol.push(X2)

        if (!sol.length) {
          this.resultText = 'Berilgan qiymatlar bilan uchburchak hosil bo\'lmaydi.'
          return
        }

        let html = ''
        sol.forEach((deg, idx) => {
          const other = 180 - this.A - deg
          const sideB = ratio * Math.sin(deg * Math.PI / 180)
          html += `<strong>Yechim ${idx+1}:</strong> ${this.targetAngleName} ≈ ${Math.round(deg*100)/100}°; boshqa burchak ≈ ${Math.round(other*100)/100}°; tegishli tomon ≈ ${Math.round(sideB*100)/100} birlik.<br/>`
        })
        this.resultText = html
      }
    }
  },
  mounted() {
    // auto-solve on mount
    this.solve()
  }
}
</script>

<style scoped>
/* minimal scoped rules — rely on global utilities */
</style>
