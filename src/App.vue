<template>
  <div class="peach-planner">
    <h1>桃プランナー</h1>

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
    <ul class="peach-list">
      <li v-for="(element, index) in peaches" :key="element.id" class="peach-item">
        {{ element.name }} - {{ getPeriodLabel(element.startPeriod) }}から{{ getPeriodLabel(element.endPeriod) }}
        <br>
        色: {{ element.color === 'yellow' ? '黄色' : 'ピンク' }},
        硬さ: {{ element.texture === 'hard' ? '硬い' : '柔らかい' }}
        <button @click="editPeach(index)">編集</button>
        <button @click="removePeach(index)">削除</button>
      </li>
    </ul>

    <!-- チャート生成ボタン -->
    <button @click="generateChart" :disabled="peaches.length === 0">チャートを生成</button>
  </div>
</template>

<script>
import { ref } from 'vue';
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

    const updateStartPeriod = () => {
      if (newPeach.value.startMonth && newPeach.value.startDecade) {
        const startValue = `${newPeach.value.startMonth}-${newPeach.value.startDecade}`;
        const endValue = `${newPeach.value.endMonth}-${newPeach.value.endDecade}`;
        if (endValue && startValue > endValue) {
          newPeach.value.endMonth = newPeach.value.startMonth;
          newPeach.value.endDecade = newPeach.value.startDecade;
        }
      }
    };

    const updateEndPeriod = () => {
      if (newPeach.value.endMonth && newPeach.value.endDecade) {
        const startValue = `${newPeach.value.startMonth}-${newPeach.value.startDecade}`;
        const endValue = `${newPeach.value.endMonth}-${newPeach.value.endDecade}`;
        if (startValue && endValue < startValue) {
          newPeach.value.startMonth = newPeach.value.endMonth;
          newPeach.value.startDecade = newPeach.value.endDecade;
        }
      }
    };

    const addPeach = () => {
      const peach = {
        id: Date.now(),
        name: newPeach.value.name,
        startPeriod: `${newPeach.value.startMonth}-${newPeach.value.startDecade}`,
        endPeriod: `${newPeach.value.endMonth}-${newPeach.value.endDecade}`,
        color: newPeach.value.color,
        texture: newPeach.value.texture
      };

      peaches.value.push(peach);

      // フォームをリセット
      newPeach.value = {
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
    };

    const generateChart = () => {
      try {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        // スケールファクターを定義
        const scale = 1.5;
        // 余白を定義
        const padding = 16 * scale;

        // 表示する月の範囲を決定
        const allMonths = peaches.value.flatMap(p => [
          parseInt(p.startPeriod.split('-')[0]),
          parseInt(p.endPeriod.split('-')[0])
        ]);
        const startMonth = Math.min(...allMonths);
        const endMonth = Math.max(...allMonths);

        // 始端と終端の月に桃があるか確認
        const hasStartMonth = peaches.value.some(p => parseInt(p.startPeriod.split('-')[0]) === startMonth || parseInt(p.endPeriod.split('-')[0]) === startMonth);
        const hasEndMonth = peaches.value.some(p => parseInt(p.startPeriod.split('-')[0]) === endMonth || parseInt(p.endPeriod.split('-')[0]) === endMonth);

        // 表示する月の範囲を調整
        const displayStartMonth = hasStartMonth ? startMonth : startMonth + 1;
        const displayEndMonth = hasEndMonth ? endMonth : endMonth - 1;
        const monthCount = displayEndMonth - displayStartMonth + 1;

        // キャンバスのサイズを設定（余白を含める）
        const monthWidth = 250 * scale;
        canvas.width = monthCount * monthWidth + (padding * 2);  // 左右の余白を追加
        canvas.height = 400 * scale + (padding * 2);  // 上下の余白を追加

        const width = canvas.width;
        const height = canvas.height;

        // コンテキストのスケールを設定
        ctx.scale(scale, scale);

        // 背景を描画
        ctx.fillStyle = '#F5F9F4';
        ctx.fillRect(0, 0, width / scale, height / scale);

        // 描画位置を余白分ずらす
        ctx.translate(padding / scale, padding / scale);

        // 月を上部に表示
        ctx.fillStyle = '#4A4A4A';
        ctx.font = `bold ${16}px Arial`;
        ctx.textAlign = 'center';
        for (let i = 0; i < monthCount; i++) {
          const month = displayStartMonth + i;
          const x = i * (monthWidth / scale) + (monthWidth / scale) / 2;
          ctx.fillText(`${month}月`, x, 20);
        }


        // 月と旬の区切り線を描画
        ctx.strokeStyle = '#4A4A4A';
        ctx.lineWidth = 1;
        ctx.font = `${14}px Arial`;
        ctx.textAlign = 'center';

        // 線とラベルの間の空白
        const lineMargin = 2 * scale;
        const labelMargin = 4 * scale; // ラベル間の余白を増やす


        // 始端の左の線を描画
        // ctx.beginPath();
        // ctx.moveTo(lineMargin / scale - 4, 30);
        // ctx.lineTo(lineMargin / scale - 4, height / scale);
        // ctx.stroke();

        // 終端の右の線を描画
        // ctx.beginPath();
        // ctx.moveTo(width / scale - lineMargin / scale, 30);
        // ctx.lineTo(width / scale - lineMargin / scale, height / scale);
        // ctx.stroke();

        for (let i = 0; i <= monthCount; i++) {
          const month = displayStartMonth + i;
          const x = i * (monthWidth / scale);

          // 月の境界線（実線）
          // if (i != monthCount) {
          ctx.beginPath();
          ctx.moveTo(x + lineMargin - 4 / scale, 30);
          ctx.lineTo(x + lineMargin - 4 / scale, height / scale);
          ctx.stroke();
          // }

          // 月のラベル
          // if (i < monthCount) {
          //   ctx.fillText(`${month}月`, x + (monthWidth / scale) / 2, 20);
          // }

          // 旬の区切り線（実線、薄い色）
          if (i < monthCount) {
            ctx.strokeStyle = '#CCCCCC';
            for (let j = 1; j <= 2; j++) {
              const subX = x + ((monthWidth / scale) * j / 3);
              ctx.beginPath();
              ctx.moveTo(subX, 30);
              ctx.lineTo(subX, height / scale);
              ctx.stroke();
            }
            ctx.strokeStyle = '#4A4A4A';
          }
        }

        const labelHeight = 25 * scale;

        // 桃のラベルを描画
        ctx.lineWidth = 1;
        let rows = [];
        peaches.value.forEach((peach) => {
          const startMonth = parseInt(peach.startPeriod.split('-')[0]);
          const startDecade = parseInt(peach.startPeriod.split('-')[1]);
          const endMonth = parseInt(peach.endPeriod.split('-')[0]);
          const endDecade = parseInt(peach.endPeriod.split('-')[1]);

          const startX = ((startMonth - displayStartMonth) * monthWidth + ((startDecade - 1) * monthWidth / 3)) / scale;
          const endX = ((endMonth - displayStartMonth + 1) * monthWidth - ((3 - endDecade) * monthWidth / 3)) / scale;
          const labelWidth = endX - startX;

          // 重なりチェックと行の割り当て
          let rowIndex = rows.findIndex(row => {
            return row.every(item => item.endX <= startX || item.startX >= endX);
          });

          if (rowIndex === -1) {
            rows.push([]);
            rowIndex = rows.length - 1;
          }

          rows[rowIndex].push({ startX, endX, peach, labelWidth });
        });

        // 各行のラベルを描画
        rows.forEach((row, rowIndex) => {
          const y = 40 + rowIndex * ((labelHeight / scale) + labelMargin / scale);

          row.forEach(({ startX, endX, peach, labelWidth }) => {
            // グラデーションの作成
            let gradient;
            if (peach.color === 'yellow') {
              gradient = ctx.createLinearGradient(startX, 0, startX + labelWidth, 0);
              gradient.addColorStop(0, "#FFF3C7");
              gradient.addColorStop(1, "#FFF3C7");
            } else {
              gradient = ctx.createLinearGradient(startX, 0, startX + labelWidth, 0);
              gradient.addColorStop(0, "#FEC7B4");
              gradient.addColorStop(1, "#FEC7B4");
            }

            // 桃のラベルの背景
            ctx.fillStyle = gradient;

            // すべての月の境界線に対して余白を適用
            const adjustedStartX = Math.max(startX, lineMargin / scale) + lineMargin / scale;
            const adjustedEndX = Math.min(endX, width / scale - lineMargin / scale) - lineMargin / scale;
            const adjustedLabelWidth = adjustedEndX - adjustedStartX;


            if (peach.texture === 'soft') {
              // 柔らかい桃の場合、両端が丸い形状を描画
              const radius = (labelHeight / scale) / 2;
              ctx.beginPath();
              ctx.moveTo(adjustedStartX + radius, y);
              ctx.lineTo(adjustedEndX - radius, y);
              ctx.quadraticCurveTo(adjustedEndX, y, adjustedEndX, y + radius);
              ctx.lineTo(adjustedEndX, y + (labelHeight / scale) - radius);
              ctx.quadraticCurveTo(adjustedEndX, y + (labelHeight / scale), adjustedEndX - radius, y + (labelHeight / scale));
              ctx.lineTo(adjustedStartX + radius, y + (labelHeight / scale));
              ctx.quadraticCurveTo(adjustedStartX, y + (labelHeight / scale), adjustedStartX, y + (labelHeight / scale) - radius);
              ctx.lineTo(adjustedStartX, y + radius);
              ctx.quadraticCurveTo(adjustedStartX, y, adjustedStartX + radius, y);
              ctx.fill();
            } else {
              // 硬い桃の場合、通常の四角形を描画
              ctx.fillRect(adjustedStartX, y, adjustedLabelWidth, labelHeight / scale);
            }

            // 桃のラベルのテキスト
            ctx.fillStyle = '#000000';
            ctx.font = `${12}px Arial`;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';

            const text = peach.name;
            const maxWidth = adjustedLabelWidth - 10;
            const centerX = adjustedStartX + adjustedLabelWidth / 2;
            const centerY = y + (labelHeight / scale) / 2;

            // テキストを改行して表示
            const words = text.split('');
            let line = '';
            let lines = [];
            let currentWidth = 0;

            for (let i = 0; i < words.length; i++) {
              const testLine = line + words[i];
              const metrics = ctx.measureText(testLine);
              const testWidth = metrics.width;

              if (testWidth > maxWidth && i > 0) {
                lines.push(line);
                line = words[i];
                currentWidth = ctx.measureText(words[i]).width;
              } else {
                line = testLine;
                currentWidth = testWidth;
              }
            }
            lines.push(line);

            // 改行したテキストを描画
            const lineHeight = 14; // 行の高さ
            const totalHeight = lineHeight * lines.length;
            let textY = centerY - (totalHeight / 2) + (lineHeight / 2);

            lines.forEach((line, index) => {
              ctx.fillText(line, centerX, textY);
              textY += lineHeight;
            });
          });
        });

        // キャンバスを画像データURLに変換
        const imageDataUrl = canvas.toDataURL('image/png');

        // 新しいタブで画像を開く
        const newTab = window.open();
        newTab.document.write(`<img src="${imageDataUrl}" alt="Peach Chart" style="width: ${width / scale}px; height: ${height / scale}px;">`);
        newTab.document.close();

      } catch (error) {
      }
    };

    return {
      peaches,
      newPeach,
      months,
      addPeach,
      editPeach,
      removePeach,
      getPeriodLabel,
      generateChart,
      updateStartPeriod,
      updateEndPeriod,
      updateOrder
    };
  }
}
</script>

<style>
:root {
  --background-color: #ffffff;
  --text-color: #4A4A4A;
  --border-color: #dddddd;
  --input-background: #f0f0f0;
}

@media (prefers-color-scheme: dark) {
  :root {
    /* ダークモードでもライトモードと同じ色を使用 */
    --background-color: #ffffff;
    --text-color: #4A4A4A;
    --border-color: #dddddd;
    --input-background: #f0f0f0;
  }
}

body {
  background-color: var(--background-color);
  color: var(--text-color);
}

.peach-planner {
  font-size: 16px;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: var(--background-color);
  color: var(--text-color);
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

input,
select,
button {
  background-color: var(--input-background);
  color: var(--text-color);
  border: 1px solid var(--border-color);
  padding: 5px;
}

.peach-list {
  list-style-type: none;
  padding: 0;
}

.peach-item {
  padding: 10px;
  margin-bottom: 5px;
  background-color: var(--input-background);
  border: 1px solid var(--border-color);
  color: var(--text-color);
}


button {
  cursor: pointer;
  background-color: var(--input-background);
  color: var(--text-color);
  border: 1px solid var(--border-color);
  padding: 5px 10px;
}

button:hover {
  opacity: 0.8;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
