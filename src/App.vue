<template>
  <div class="app-container">
    <div v-if="currentQuestionIndex < questions.length" class="card">
      <div class="progress">
        <div class="progress-bar" :style="{ width: progress + '%' }"></div>
      </div>
      <div class="question">{{ questions[currentQuestionIndex].text }}</div>
      <div class="option" v-for="(option, index) in questions[currentQuestionIndex].options" :key="index">
        <label>
          <input :type="questions[currentQuestionIndex].multiple ? 'checkbox' : 'radio'"
                 :name="'q'+currentQuestionIndex"
                 :value="option.tag"
                 v-model="userAnswers[currentQuestionIndex]">
          {{ option.text }}
        </label>
      </div>
      <div class="btn" @click="nextQuestion">下一题</div>
    </div>

    <div v-else class="card">
      <div class="result-title">🎓 专业匹配结果</div>
      <div class="result-card" v-for="rec in recommendations" :key="rec.name">
        <div class="result-name">{{ rec.name }}</div>
        <div>{{ rec.brief }}</div>
        <div style="font-size: 13px; color: #555">推荐理由：{{ rec.reason }}</div>
      </div>
      <div class="btn" @click="restart">重新测试</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentQuestionIndex: 0,
      userAnswers: [],
      tagScores: {},
      questions: [
        {
          text: '你更喜欢以下哪类课程？',
          multiple: false,
          options: [
            { text: '物理 / 数学 / 信息技术', tag: '理工' },
            { text: '历史 / 政治 / 哲学', tag: '人文' },
            { text: '绘画 / 音乐 / 表演', tag: '艺术设计' },
            { text: '经济 / 管理 / 商业', tag: '经管' }
          ]
        },
        {
          text: '你擅长以下哪种能力？（可多选）',
          multiple: true,
          options: [
            { text: '逻辑推理', tag: '信息工程' },
            { text: '表达沟通', tag: '心理学' },
            { text: '动手实践', tag: '机械设计' },
            { text: '艺术创作', tag: '艺术设计' }
          ]
        },
        {
          text: '你更向往哪种职业环境？',
          multiple: false,
          options: [
            { text: '研发或工程类公司', tag: '信息工程' },
            { text: '咨询 / 教育 / 心理相关', tag: '心理学' },
            { text: '创意或艺术类行业', tag: '艺术设计' },
            { text: '投行 / 商业公司', tag: '金融学' }
          ]
        }
      ]
    }
  },
  computed: {
    progress() {
      return (this.currentQuestionIndex / this.questions.length) * 100;
    },
    recommendations() {
      const sorted = Object.entries(this.tagScores).sort((a, b) => b[1] - a[1]);
      return sorted.slice(0, 3).map(([tag]) => {
        const mapping = {
          '心理学': {
            brief: '研究人类心理与行为的科学。',
            reason: '你表现出良好的人际沟通与心理洞察力。'
          },
          '信息工程': {
            brief: '涉及编程、算法与系统开发。',
            reason: '你具备出色的逻辑推理与技术倾向。'
          },
          '金融学': {
            brief: '研究资金流动与金融体系运作。',
            reason: '你偏好商业逻辑与策略思维。'
          },
          '艺术设计': {
            brief: '注重创意表达与艺术实践。',
            reason: '你展现出独特的审美与创造能力。'
          },
          '机械设计': {
            brief: '结合力学、材料与制造的工程专业。',
            reason: '你偏好动手能力与实际操作。'
          },
          '人文': {
            brief: '探索文化、哲学与历史发展。',
            reason: '你兴趣广泛并具有人文关怀。'
          },
          '理工': {
            brief: '强调科学方法与工程解决方案。',
            reason: '你具备分析能力与工程潜质。'
          },
          '经管': {
            brief: '关注资源管理与企业运营。',
            reason: '你擅长综合分析并关注实用价值。'
          }
        }
        return {
          name: tag,
          ...mapping[tag] || {brief: '暂无简介', reason: '暂无理由'}
        }
      });
    }
  },
  methods: {
    nextQuestion() {
      const ans = this.userAnswers[this.currentQuestionIndex];
      const opts = this.questions[this.currentQuestionIndex].options;
      if (Array.isArray(ans)) {
        ans.forEach(tag => this.tagScores[tag] = (this.tagScores[tag] || 0) + 1);
      } else {
        const tag = opts.find(o => o.tag === ans)?.tag;
        if (tag) this.tagScores[tag] = (this.tagScores[tag] || 0) + 1;
      }
      this.currentQuestionIndex++;
    },
    restart() {
      this.currentQuestionIndex = 0;
      this.userAnswers = [];
      this.tagScores = {};
    }
  }
}
</script>

<style>
body {
  font-family: "Helvetica Neue", sans-serif;
  background: #f3f6fa;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
}
.app-container {
  max-width: 480px;
  width: 100%;
  padding: 20px;
}
.card {
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.05);
  padding: 20px;
  margin-bottom: 20px;
  transition: all 0.3s ease-in-out;
}
.progress {
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 20px;
}
.progress-bar {
  height: 100%;
  background: #3b82f6;
}
.question {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
}
.option {
  margin-bottom: 12px;
}
.option input {
  margin-right: 8px;
}
.btn {
  background: #3b82f6;
  color: white;
  padding: 12px;
  text-align: center;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  margin-top: 16px;
}
.btn:hover {
  background: #2563eb;
}
.result-title {
  font-size: 22px;
  font-weight: 700;
  margin-bottom: 12px;
}
.result-card {
  border-left: 6px solid #3b82f6;
  background: #f0f7ff;
  padding: 12px 16px;
  margin-bottom: 12px;
  border-radius: 6px;
}
.result-name {
  font-weight: bold;
  margin-bottom: 6px;
}
</style>