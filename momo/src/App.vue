<template>
  <div class="peach-planner">
    <h1>桃の食べ頃プランナー</h1>

    <!-- 桃の入力フォーム -->
    <form @submit.prevent="addPeach">
      <input v-model="newPeach.name" maxlength="10" placeholder="桃の品種名" required>

      <!-- 開始時期 -->
      <div class="period-selector">
        <select v-model="newPeach.startMonth" @change="updateStartPeriod" required>
          <option value="">月を選択</option>
          <option v-for="month in months" :key="month" :value="month">{{ month }}月</option>
        </select>
        <select v-model="newPeach.startDecade" @change="updateStartPeriod" required>
          <option value="">時期を選択</option>
          <option value="1">上旬</option>
          <option value="2">中旬</option>
          <option value="3">下旬</option>
        </select>
      </div>

      <!-- 終了時期 -->
      <div class="period-selector">
        <select v-model="newPeach.endMonth" @change="updateEndPeriod" required>
          <option value="">月を選択</option>
          <option v-for="month in months" :key="month" :value="month">{{ month }}月</option>
        </select>
        <select v-model="newPeach.endDecade" @change="updateEndPeriod" required>
          <option value="">時期を選択</option>
          <option value="1">上旬</option>
          <option value="2">中旬</option>
          <option value="3">下旬</option>
        </select>
      </div>

      <select v-model="newPeach.color" required>
        <option value="yellow">黄色</option>
        <option value="pink">ピンク</option>
      </select>
      <select v-model="newPeach.texture" required>
        <option value="hard">硬い</option>
        <option value="soft">柔らかい</option>
      </select>
      <button type="submit">追加</button>
    </form>

    <!-- 桃の一覧 -->
    <h2>桃の一覧</h2>
    <draggable v-model="peaches" @end="updateOrder" item-key="id">
      <template #item="{ element, index }">
        <li class="draggable-item">
          {{ element.name }} - {{ getPeriodLabel(element.startPeriod) }}から{{ getPeriodLabel(element.endPeriod) }}
          <br>
          色: {{ element.color === 'yellow' ? '黄色' : 'ピンク' }},
          硬さ: {{ element.texture === 'hard' ? '硬い' : '柔らかい' }}
          <button @click="editPeach(index)">編集</button>
          <button @click="removePeach(index)">削除</button>
        </li>
      </template>
    </draggable>

    <!-- チャート生成ボタン -->
    <button @click="generateChart" :disabled="peaches.length === 0">チャートを生成</button>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import draggable from 'vuedraggable';

export default {
  components: {
    draggable
  },
  setup() {
    const peaches = ref([]);
    const newPeach = ref({
      name: '',
      startMonth: '',
      startDecade: '',
      endMonth: '',
      endDecade: '',
      color: 'yellow',
      texture: 'hard'
    });

    const months = [5, 6, 7, 8, 9, 10];

    const getPeriodValue = (month, decade) => `${month}-${decade}`;

    const updateStartPeriod = () => {
      if (newPeach.value.startMonth && newPeach.value.startDecade) {
        const startValue = getPeriodValue(newPeach.value.startMonth, newPeach.value.startDecade);
        const endValue = getPeriodValue(newPeach.value.endMonth, newPeach.value.endDecade);
        if (endValue && startValue > endValue) {
          newPeach.value.endMonth = newPeach.value.startMonth;
          newPeach.value.endDecade = newPeach.value.startDecade;
        }
      }
    };

    const updateEndPeriod = () => {
      if (newPeach.value.endMonth && newPeach.value.endDecade) {
        const startValue = getPeriodValue(newPeach.value.startMonth, newPeach.value.startDecade);
        const endValue = getPeriodValue(newPeach.value.endMonth, newPeach.value.endDecade);
        if (startValue && endValue < startValue) {
          newPeach.value.startMonth = newPeach.value.endMonth;
          newPeach.value.startDecade = newPeach.value.endDecade;
        }
      }
    };

    const addPeach = () => {
      const newPeach = {
        id: Date.now(), // ユニークなIDとして現在のタイムスタンプを使用
        name: newPeachData.value.name,
        startPeriod: `${newPeachData.value.startMonth}-${newPeachData.value.startDecade}`,
        endPeriod: `${newPeachData.value.endMonth}-${newPeachData.value.endDecade}`,
        color: newPeachData.value.color,
        texture: newPeachData.value.texture
      };

      peaches.value.push(newPeach);

      // フォームをリセット
      newPeachData.value = {
        name: '',
        startMonth: '',
        startDecade: '',
        endMonth: '',
        endDecade: '',
        color: 'yellow',
        texture: 'hard'
      };
    };

    const editPeach = (index) => {
      const peach = peaches.value[index];
      newPeach.value = {
        ...peach,
        startMonth: peach.startPeriod.split('-')[0],
        startDecade: peach.startPeriod.split('-')[1],
        endMonth: peach.endPeriod.split('-')[0],
        endDecade: peach.endPeriod.split('-')[1]
      };
      peaches.value.splice(index, 1);
    };

    const removePeach = (index) => {
      peaches.value.splice(index, 1);
    };

    const getPeriodLabel = (value) => {
      if (!value) return '';
      const [month, decade] = value.split('-');
      const decadeLabel = ['上旬', '中旬', '下旬'][parseInt(decade) - 1];
      return `${month}月${decadeLabel}`;
    };

    const updateOrder = () => {
      // ドラッグ＆ドロップ後の処理
      // 必要に応じて追加の処理を行うことができます
      console.log('Order updated:', peaches.value);
    };

    const generateChart = () => {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');

      // 表示する月の範囲を決定
      const startMonth = Math.min(...peaches.value.map(p => parseInt(p.startPeriod.split('-')[0])));
      const endMonth = Math.max(...peaches.value.map(p => parseInt(p.endPeriod.split('-')[0])));
      const monthCount = endMonth - startMonth + 1;

      // キャンバスのサイズを設定
      const monthWidth = 200; // 1ヶ月あたりの幅
      canvas.width = monthCount * monthWidth;
      canvas.height = 450;

      const width = canvas.width;
      const height = canvas.height;

      // 背景を描画
      ctx.fillStyle = '#FFFFFF';
      ctx.fillRect(0, 0, width, height);

      // タイトルを描画
      ctx.fillStyle = '#000000';
      ctx.font = 'bold 20px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('桃の食べ頃カレンダー', width / 2, 25);

      // 月と旬の区切り線を描画
      ctx.strokeStyle = '#000000';
      ctx.lineWidth = 2;
      ctx.font = '16px Arial';
      ctx.textAlign = 'center';

      for (let i = 0; i < monthCount; i++) {
        const month = startMonth + i;
        const x = i * monthWidth;

        // 月の境界線（実線）
        ctx.beginPath();
        ctx.moveTo(x, 60);
        ctx.lineTo(x, height);
        ctx.stroke();

        // 月のラベル
        ctx.fillText(`${month}月`, x + monthWidth / 2, 50);

        // 旬の区切り線（破線）
        ctx.setLineDash([5, 5]);
        ctx.lineWidth = 1;
        for (let j = 1; j <= 2; j++) {
          const subX = x + (monthWidth * j / 3);
          ctx.beginPath();
          ctx.moveTo(subX, 60);
          ctx.lineTo(subX, height);
          ctx.stroke();
        }
        ctx.setLineDash([]);
      }

      // 桃のラベルを描画
      ctx.lineWidth = 1;
      let currentRow = 0;
      peaches.value.forEach((peach) => {
        const startMonth = parseInt(peach.startPeriod.split('-')[0]);
        const startDecade = parseInt(peach.startPeriod.split('-')[1]);
        const endMonth = parseInt(peach.endPeriod.split('-')[0]);
        const endDecade = parseInt(peach.endPeriod.split('-')[1]);

        const startX = (startMonth - startMonth) * monthWidth + ((startDecade - 1) * monthWidth / 3);
        const endX = (endMonth - startMonth + 1) * monthWidth + (endDecade * monthWidth / 3);
        const labelWidth = endX - startX;
        const labelHeight = 30;

        const y = 70 + currentRow * (labelHeight + 10);

        // 桃のラベルの背景
        ctx.fillStyle = peach.color === 'yellow' ? '#FFD700' : '#FFC0CB';

        if (peach.texture === 'soft') {
          // 柔らかい桃の場合、角丸の四角形を描画
          const radius = labelHeight / 2;
          ctx.beginPath();
          ctx.moveTo(startX + radius, y);
          ctx.lineTo(endX - radius, y);
          ctx.quadraticCurveTo(endX, y, endX, y + radius);
          ctx.lineTo(endX, y + labelHeight - radius);
          ctx.quadraticCurveTo(endX, y + labelHeight, endX - radius, y + labelHeight);
          ctx.lineTo(startX + radius, y + labelHeight);
          ctx.quadraticCurveTo(startX, y + labelHeight, startX, y + labelHeight - radius);
          ctx.lineTo(startX, y + radius);
          ctx.quadraticCurveTo(startX, y, startX + radius, y);
          ctx.fill();
          ctx.stroke();
        } else {
          // 硬い桃の場合、通常の四角形を描画
          ctx.fillRect(startX, y, labelWidth, labelHeight);
          ctx.strokeRect(startX, y, labelWidth, labelHeight);
        }

        // 桃のラベルのテキスト
        ctx.fillStyle = '#000000';
        ctx.font = '14px Arial';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(peach.name, startX + labelWidth / 2, y + labelHeight / 2);

        currentRow++;
      });

      // キャンバスを画像データURLに変換
      const imageDataUrl = canvas.toDataURL('image/png');

      // 新しいタブで画像を開く
      const newTab = window.open();
      newTab.document.write('<img src="' + imageDataUrl + '" alt="Peach Chart">');
    };

    return {
      peaches,
      updateOrder,
      newPeach,
      months,
      addPeach,
      editPeach,
      removePeach,
      getPeriodLabel,
      generateChart,
      updateStartPeriod,
      updateEndPeriod
    };
  }
}
</script>

<style scoped>
.peach-planner {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.period-selector {
  display: flex;
  gap: 10px;
}

.draggable-list {
  list-style-type: none;
  padding: 0;
}

.draggable-item {
  cursor: move;
  padding: 10px;
  margin-bottom: 5px;
  background-color: #f0f0f0;
  border: 1px solid #ddd;
}
</style>
