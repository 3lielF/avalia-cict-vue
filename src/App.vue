<script setup>
import { computed, ref } from 'vue'

const screen = ref('home')
const mode = ref('oral')
const area = ref('Ciências Exatas')
const evaluatorType = ref('area')
const selectedType = ref(null)
const selectedWork = ref(null)
const toast = ref('')
const scores = ref([0, 0, 0, 0])
const comment = ref('')
const draft = ref('')

const works = {
  'PIBIC': [
    { id: 1, title: 'Estudo da convergência de séries numéricas aplicadas', author: 'Marina Silva Costa', advisor: 'Prof. Dr. Carlos Mendes', session: 'Sessão 1', time: '08:00 – 10:00', area: 'Ciências Exatas' },
    { id: 2, title: 'Modelagem matemática de sistemas dinâmicos', author: 'Lucas Pereira Almeida', advisor: 'Prof.ª Dr.ª Ana Souza', session: 'Sessão 1', time: '08:00 – 10:00', area: 'Ciências Exatas' },
    { id: 3, title: 'Algoritmos de otimização aplicados a redes neurais', author: 'Fernanda Lima Rocha', advisor: 'Prof. Dr. Rafael Costa', session: 'Sessão 2', time: '10:30 – 12:00', area: 'Ciências Exatas' }
  ],
  'PIBIC-EM': [
    { id: 4, title: 'Robótica educacional no ensino médio', author: 'João Victor Santos', advisor: 'Prof.ª Carla Dias', session: 'Sessão 1', time: '08:00 – 10:00', area: 'Ciências Exatas' },
    { id: 5, title: 'Experimentos de física com materiais recicláveis', author: 'Beatriz Alves', advisor: 'Prof. Marcos Lima', session: 'Sessão 2', time: '10:30 – 12:00', area: 'Ciências Exatas' }
  ],
  'PITCH (PIBITI)': [
    { id: 6, title: 'Sistema embarcado para monitoramento ambiental', author: 'Lucas Santos', advisor: 'Prof.ª Renata Alves', session: 'Sessão 1', time: '08:00 – 08:20', area: 'Ciências Exatas' },
    { id: 7, title: 'Gamificação no ensino de história', author: 'Mariana Alves', advisor: 'Prof. João Pedro', session: 'Sessão 1', time: '08:20 – 08:40', area: 'Ciências Humanas' },
    { id: 8, title: 'Aplicativo para saúde mental com IA', author: 'Pedro Henrique Souza', advisor: 'Prof.ª Juliana Reis', session: 'Sessão 2', time: '10:30 – 10:50', area: 'Ciências da Saúde' },
    { id: 9, title: 'Robótica colaborativa em ambientes industriais', author: 'Ana Clara Ribeiro', advisor: 'Prof. Carlos Mendes', session: 'Sessão 2', time: '10:50 – 11:10', area: 'Engenharias' },
    { id: 10, title: 'Uso de drones na agricultura de precisão', author: 'João Victor Lima', advisor: 'Prof.ª Camila Rocha', session: 'Sessão 3', time: '13:30 – 13:50', area: 'Ciências Agrárias' }
  ]
}

const types = [
  { key: 'PIBIC', title: 'PIBIC', desc: 'Trabalhos de iniciação científica (PIBIC).', icon: '▤' },
  { key: 'PIBIC-EM', title: 'PIBIC-EM', desc: 'Trabalhos de iniciação científica no ensino médio.', icon: '⌂' },
  { key: 'PITCH (PIBITI)', title: 'PITCH (PIBITI)', desc: 'Trabalhos de inovação tecnológica.', icon: '◉' }
]

const areaWorks = computed(() => {
  if (!selectedType.value) return []
  return (works[selectedType.value] || []).filter(w => evaluatorType.value === 'pitch' || w.area === area.value)
})

const pitchWorks = computed(() => works['PITCH (PIBITI)'])

function showToast(message) {
  toast.value = message
  setTimeout(() => toast.value = '', 2200)
}

function openType(type) {
  selectedType.value = type
  screen.value = 'list'
}

function openEvaluation(work) {
  selectedWork.value = work
  const saved = JSON.parse(localStorage.getItem(`avalia-cict-${work.id}`) || 'null')
  scores.value = saved?.scores || [0, 0, 0, 0]
  comment.value = saved?.comment || ''
  draft.value = saved?.draft || ''
  screen.value = 'evaluation'
}

function autosave() {
  if (!selectedWork.value) return
  localStorage.setItem(`avalia-cict-${selectedWork.value.id}`, JSON.stringify({
    scores: scores.value,
    comment: comment.value,
    draft: draft.value
  }))
}

function saveEvaluation() {
  autosave()
  showToast('Nota enviada com sucesso.')
  screen.value = selectedType.value ? 'list' : 'pitch'
}

function backHome() {
  screen.value = evaluatorType.value === 'pitch' ? 'pitch' : 'home'
}

function enter(kind) {
  evaluatorType.value = kind
  screen.value = kind === 'pitch' ? 'pitch' : 'home'
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div class="brand" @click="backHome">
        <div class="brand-mark">◆</div>
        <strong>Avalia CICT 2026</strong>
      </div>
      <div class="user">
        <span>{{ evaluatorType === 'pitch' ? 'Avaliador PITCH' : 'Eliel Fernandes Filho' }}</span>
        <span class="pill">{{ evaluatorType === 'pitch' ? 'PIBITI' : area }}</span>
        <button class="avatar">EF</button>
      </div>
    </header>

    <main>
      <!-- LOGIN / DEMO SWITCH -->
      <section v-if="screen === 'login'" class="login-card">
        <div class="eyebrow">MOCKUP INTERATIVO</div>
        <h1>Entrar como avaliador</h1>
        <p>Escolha o fluxo para testar a experiência.</p>
        <div class="login-options">
          <button @click="enter('area')">Avaliador de Grande Área</button>
          <button class="secondary" @click="enter('pitch')">Avaliador PITCH</button>
        </div>
      </section>

      <!-- AREA HOME -->
      <section v-else-if="screen === 'home'" class="page">
        <div class="hero-row">
          <div>
            <h1>Trabalhos para Avaliar</h1>
            <p>Selecione o tipo de trabalho ou continue uma avaliação em andamento.</p>
          </div>
          <div class="mode-toggle">
            <button :class="{active: mode === 'oral'}" @click="mode='oral'">Oral</button>
            <button :class="{active: mode === 'video'}" @click="mode='video'">Vídeos</button>
          </div>
        </div>

        <div class="summary">
          <div class="summary-icon">▤</div>
          <div><strong>8 trabalhos pendentes</strong><small>de um total de 24 atribuídos</small></div>
          <div class="summary-spacer"></div>
          <span class="mini-progress">33%</span>
        </div>

        <div class="area-selector">
          <span>Grande área do avaliador</span>
          <select v-model="area">
            <option>Ciências Exatas</option>
            <option>Ciências Humanas</option>
            <option>Ciências da Vida</option>
          </select>
        </div>

        <div class="cards-grid">
          <article v-for="t in types" :key="t.key" class="type-card">
            <div class="type-icon">{{ t.icon }}</div>
            <h2>{{ t.title }}</h2>
            <p>{{ t.desc }}</p>
            <div class="progress-line"><span :style="{width: t.key === 'PIBIC' ? '0%' : '0%'}"></span></div>
            <small>0 de {{ t.key === 'PIBIC' ? 12 : t.key === 'PIBIC-EM' ? 8 : 4 }} avaliados · 0%</small>
            <button @click="openType(t.key)">Avaliar trabalhos <span>→</span></button>
          </article>
        </div>
      </section>

      <!-- LIST -->
      <section v-else-if="screen === 'list'" class="page">
        <div class="breadcrumb" @click="screen='home'">← Voltar para a página inicial</div>
        <div class="hero-row">
          <div>
            <h1>{{ selectedType }}</h1>
            <p>Apenas trabalhos da sua grande área <b>({{ area }})</b> são exibidos.</p>
          </div>
          <div class="mode-toggle">
            <button :class="{active: mode === 'oral'}" @click="mode='oral'">Oral</button>
            <button :class="{active: mode === 'video'}" @click="mode='video'">Vídeos</button>
          </div>
        </div>

        <div class="tabs">
          <span class="active">Pendentes ({{ areaWorks.length }})</span>
          <span>Em andamento (0)</span>
          <span>Concluídos (0)</span>
        </div>

        <div class="work-list">
          <article v-for="w in areaWorks" :key="w.id" class="work-row" @click="openEvaluation(w)">
            <div class="work-main">
              <span class="status">Pendente</span>
              <h3>{{ w.title }}</h3>
              <p>Autor: {{ w.author }} · Orientador: {{ w.advisor }}</p>
              <div class="meta"><span>▣ {{ w.session }} · {{ w.time }}</span><span>⌖ {{ w.area }}</span></div>
            </div>
            <button>Avaliar ›</button>
          </article>
          <div v-if="!areaWorks.length" class="empty">Nenhum trabalho disponível para esta grande área.</div>
        </div>
      </section>

      <!-- VIDEO LIST -->
      <section v-else-if="screen === 'videos'" class="page">
        <div class="breadcrumb" @click="screen='home'">← Voltar para a página inicial</div>
        <div class="hero-row">
          <div>
            <h1>Trabalhos em Vídeo</h1>
            <p>A avaliação dos vídeos segue os mesmos critérios do oral.</p>
          </div>
          <div class="mode-toggle">
            <button @click="mode='oral'; screen='home'">Oral</button>
            <button class="active">Vídeos</button>
          </div>
        </div>
        <div class="video-list">
          <article v-for="w in areaWorks" :key="w.id" class="work-row" @click="openEvaluation(w)">
            <div>
              <span class="status">Pendente</span>
              <h3>{{ w.title }}</h3>
              <p>Autor: {{ w.author }}</p>
            </div>
            <button>Assistir e avaliar ›</button>
          </article>
        </div>
      </section>

      <!-- PITCH DASHBOARD -->
      <section v-else-if="screen === 'pitch'" class="page">
        <div class="pitch-head">
          <div>
            <h1>Avaliações de PITCH</h1>
            <p>Você avalia trabalhos PITCH (PIBITI) de todas as grandes áreas.</p>
          </div>
          <div class="pitch-stats">
            <div><b>3</b><small>avaliadores por trabalho</small></div>
            <div><b>Hoje</b><small>10 trabalhos</small></div>
          </div>
        </div>
        <div class="day-heading">Hoje <span>Todos os trabalhos PITCH</span></div>
        <div class="work-list">
          <article v-for="w in pitchWorks" :key="w.id" class="work-row" @click="openEvaluation(w)">
            <div class="work-main">
              <span class="area-badge">{{ w.area }}</span>
              <h3>{{ w.title }}</h3>
              <p>Autor: {{ w.author }}</p>
              <div class="meta"><span>▣ {{ w.session }} · {{ w.time }}</span></div>
            </div>
            <div class="pitch-action"><span class="status">Pendente</span><button>Avaliar ›</button></div>
          </article>
        </div>
      </section>

      <!-- EVALUATION -->
      <section v-else-if="screen === 'evaluation'" class="page eval-page">
        <div class="breadcrumb" @click="backHome">← Voltar</div>
        <div class="hero-row">
          <div>
            <h1>Avaliar Trabalho</h1>
            <p>{{ mode === 'video' ? 'Avaliação em vídeo' : 'Avaliação oral' }} · {{ selectedWork?.area }}</p>
          </div>
          <div class="mode-toggle">
            <button :class="{active: mode === 'oral'}" @click="mode='oral'">Oral</button>
            <button :class="{active: mode === 'video'}" @click="mode='video'">Vídeos</button>
          </div>
        </div>

        <div class="eval-grid">
          <div class="left-col">
            <div class="panel">
              <h2>Dados do Trabalho</h2>
              <dl>
                <dt>Autor</dt><dd>{{ selectedWork?.author }}</dd>
                <dt>Título do Trabalho</dt><dd>{{ selectedWork?.title }}</dd>
                <dt>Orientador</dt><dd>{{ selectedWork?.advisor }}</dd>
              </dl>
            </div>
            <div v-if="mode === 'video'" class="panel video-frame">
              <div class="fake-video"><div class="play">▶</div><span>VÍDEO DO TRABALHO</span></div>
              <div class="video-bar">0:00 / 8:45 <span>🔊 ⛶</span></div>
            </div>
          </div>

          <div class="right-col">
            <div class="panel">
              <h2>Avaliação</h2>
              <p class="hint">Atribua uma nota de 0 a 10 para cada critério.</p>
              <div v-for="(label, i) in ['Qualidade da apresentação e organização do conteúdo', 'Clareza e domínio do tema', 'Relevância e originalidade', 'Metodologia e resultados']" :key="label" class="criterion">
                <label>{{ i+1 }}. {{ label }}</label>
                <div class="score">
                  <input v-model.number="scores[i]" @change="autosave" type="range" min="0" max="10" step="0.1">
                  <output>{{ scores[i].toFixed(1) }}</output>
                </div>
              </div>
            </div>
            <div class="panel draft-panel">
              <div class="panel-heading">
                <div>
                  <h2>Rascunho do docente</h2>
                  <p class="hint">Anote impressões para retomar este trabalho depois. Este campo é salvo automaticamente.</p>
                </div>
                <span class="autosave-label">Salvo automaticamente</span>
              </div>
              <textarea v-model="draft" @input="autosave" maxlength="3000" placeholder="Ex.: lembrar de perguntar sobre a metodologia e revisar a justificativa..."></textarea>
              <small>{{ draft.length }}/3000 caracteres</small>
            </div>
            <div class="panel">
              <h2>Comentários Adicionais</h2>
              <textarea v-model="comment" @input="autosave" maxlength="2000" placeholder="Comentários que acompanharão a avaliação enviada..."></textarea>
              <small>{{ comment.length }}/2000 caracteres</small>
            </div>
            <div class="eval-footer">
              <span>Média: <b>{{ (scores.reduce((a,b)=>a+b,0)/4).toFixed(1) }}</b> / 10</span>
              <button class="primary" @click="saveEvaluation">Enviar nota</button>
            </div>
          </div>
        </div>
      </section>
    </main>

    <button v-if="screen !== 'login'" class="dev-switch" @click="screen='login'">Trocar fluxo</button>
    <div v-if="toast" class="toast">{{ toast }}</div>
  </div>
</template>