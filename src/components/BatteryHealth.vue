<template>
  <div>
    <h1>电池损耗计算器</h1>
    <div>
      <label for="designCapacity">电池设计容量（mAh）：</label>
      <input type="number" id="designCapacity" v-model.number="designCapacity" min="1" />
    </div>
    <div>
      <label for="currentCapacity">当前满电容量（mAh）：</label>
      <input type="number" id="currentCapacity" v-model.number="currentCapacity" :max="designCapacity" min="0" />
    </div>
    <div>
      <label for="cycleCount">电池循环次数：</label>
      <input type="number" id="cycleCount" v-model.number="cycleCount" min="0" />
    </div>
    <div>
      <label for="initialDate">首次使用日期：</label>
      <input type="date" id="initialDate" v-model="initialDate" />
    </div>
    <div>
      <label for="currentDate">当前日期：</label>
      <input type="date" id="currentDate" v-model="currentDate" :min="today" />
    </div>
    <button @click="calculate">计算</button>
    <div v-if="result">
      <h2>结果</h2>
      <p>电池损耗百分比：{{ result.lossPercentage.toFixed(2) }}%</p>
      <p>平均每次循环损失电量：{{ result.lossPerCycle.toFixed(2) }} mAh</p>
      <p>预计容量降低到3000mAh以下的时间：{{ result.predictedDate }}</p>
      <p>更换建议：建议在 {{ result.predictedMonthYear }} 之前更换电池</p>
      <p>若对续航要求较高，可提前1-2个月更换。</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    const today = new Date().toISOString().split('T')[0]; // 获取今天的日期，格式为 YYYY-MM-DD
    return {
      designCapacity: 0,
      currentCapacity: 0,
      cycleCount: 0,
      initialDate: today, // 默认值为今天
      currentDate: today, // 默认值为今天
      today, // 用于限制当前日期不能早于今天
      result: null,
    };
  },
  methods: {
    calculate() {
      // 校验：当前满电容量是否大于设计容量
      if (this.currentCapacity > this.designCapacity) {
        alert('满电容量大于设计容量！您使用的是扩容电池吗？');
        return;
      }

      const initialDate = this.parseDate(this.initialDate);
      const currentDate = this.parseDate(this.currentDate);

      // 校验：当前日期不能早于首次使用日期
      if (currentDate < initialDate) {
        alert('当前日期不能早于首次使用日期');
        return;
      }

      const designCapacity = this.designCapacity;
      const currentCapacity = this.currentCapacity;
      const cycleCount = this.cycleCount;

      // 校验：当前容量是否已低于或等于3000mAh
      if (currentCapacity <= 3000) {
        alert('当前容量已低于或等于3000mAh，你该换电池了！');
        return;
      }

      // 计算电池损耗相关数据
      const lossPercentage = ((designCapacity - currentCapacity) * 100.0) / designCapacity;
      const lossPerCycle = (designCapacity - currentCapacity) / cycleCount;
      const requiredLoss = currentCapacity - 3000;
      const requiredCycles = requiredLoss / lossPerCycle;
      const requiredCyclesRounded = Math.ceil(requiredCycles);
      const daysPassed = Math.ceil((currentDate - initialDate) / (1000 * 60 * 60 * 24));
      const cyclesPerDay = cycleCount / daysPassed;
      const daysNeeded = requiredCyclesRounded / cyclesPerDay;
      const daysNeededRounded = Math.ceil(daysNeeded);
      const predictedDate = new Date(currentDate.getTime() + daysNeededRounded * 24 * 60 * 60 * 1000);

      const predictedMonthYear = predictedDate.toLocaleDateString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
      });

      this.result = {
        lossPercentage,
        lossPerCycle,
        predictedDate: predictedDate.toLocaleDateString('zh-CN'),
        predictedMonthYear,
      };
    },
    parseDate(dateStr) {
      const [year, month, day] = dateStr.split('-'); // 日期格式改为 YYYY-MM-DD
      return new Date(year, month - 1, day);
    },
  },
};
</script>

<style scoped>
div {
  margin: 10px 0;
}
</style>
