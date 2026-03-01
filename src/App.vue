<script setup>
  import { reactive, onMounted, ref } from 'vue';
  const estado = reactive({
    filtro: 'todos',
    tarefaTemp: '',
    tema: 'coral',
    tarefas: [
      {
        titulo: 'Estudar ES6',
        finalizada: false,
        editando: false,
      },
      {
        titulo: 'Estudar SASS',
        finalizada: false,
        editando: false,
      },
      {
        titulo: 'Malhar',
        finalizada: true,
        editando: false,
      }
    ]
  })
  const getTarefasPendentes = () => {
    return estado.tarefas.filter(tarefa => !tarefa.finalizada)
  }
  const getTarefasFinalizadas = () => {
    return estado.tarefas.filter(tarefa => tarefa.finalizada)
  }
  const getTarefasFiltradas = () => {
    const { filtro } = estado;

    switch (filtro) {
      case 'pendentes':
        return getTarefasPendentes();
      case 'finalizadas':
        return getTarefasFinalizadas();
      default:
        return estado.tarefas;
    }
  }

  const cadastraTarefa = () => {
    const tarefaNova = {
      titulo: estado.tarefaTemp,
      finalizada: false,
      editando: false,
    }
    estado.tarefas.push(tarefaNova)
    estado.tarefaTemp = '';
  }

  const iniciarEdicao = (tarefa) => {
    // mark the task as being edited and give focus via template
    tarefa.editando = true;
  }

  const finalizarEdicao = (tarefa) => {
    tarefa.editando = false;
  }

  const limparFinalizadas = () => {
    estado.tarefas = estado.tarefas.filter(t => !t.finalizada)
  }

  const marcarTodas = () => {
    const todas = estado.tarefas.every(t => t.finalizada)
    estado.tarefas.forEach(t => t.finalizada = !todas)
  }

  const getProgresso = () => {
    if (!estado.tarefas.length) return 0
    const feitas = getTarefasFinalizadas().length
    return Math.round((feitas / estado.tarefas.length) * 100)
  }

  const aplicarTema = (tema) => {
    estado.tema = tema
    try { localStorage.setItem('todo-tema', tema) } catch(e){}
    document.documentElement.setAttribute('data-theme', tema)
  }

  const tarefaInput = ref(null)
  const listaRef = ref(null)

  onMounted(() => {
    const temaSalvo = (() => { try { return localStorage.getItem('todo-tema') } catch(e){ return null } })()
    aplicarTema(temaSalvo || estado.tema)
    // focar no input de nova tarefa e rolar para a lista
    setTimeout(() => {
      tarefaInput.value?.focus()
      listaRef.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
    }, 120)
  })
</script>

<template>
  <div class="container py-4">
    <header class="hero p-4 mb-4 rounded-3 text-white">
      <div class="d-flex justify-content-between align-items-center">
        <div>
          <h1 class="mb-1">Minhas Tarefas</h1>
          <p class="mb-0 opacity-75">Você possui <strong>{{ getTarefasPendentes().length }}</strong> pendentes</p>
        </div>
        <div class="text-end d-flex align-items-center">
          <div class="me-2">
            <button class="btn btn-light btn-sm me-2" @click="marcarTodas">Marcar/Desmarcar tudo</button>
            <button class="btn btn-outline-light btn-sm" @click="limparFinalizadas">Limpar finalizadas</button>
          </div>
          <div class="d-flex align-items-center">
            <button class="btn btn-sm btn-outline-light me-2" @click.prevent="listaRef.scrollIntoView({ behavior: 'smooth' })">Entrar na lista</button>
            <select class="form-select form-select-sm" :value="estado.tema" @change="evento => aplicarTema(evento.target.value)" style="width:150px">
              <option value="coral">Tema: Coral</option>
              <option value="teal">Tema: Teal</option>
              <option value="purple">Tema: Purple</option>
            </select>
          </div>
        </div>
      </div>
      <div class="mt-3">
        <div class="progress" style="height:8px">
          <div class="progress-bar bg-light" role="progressbar" :style="`width: ${getProgresso()}%`" :aria-valuenow="getProgresso()" aria-valuemin="0" aria-valuemax="100"></div>
        </div>
        <small class="d-block mt-1 opacity-75">Progresso: {{ getProgresso() }}%</small>
      </div>
    </header>
    <form @submit.prevent="cadastraTarefa">
        <div class="input-group mb-3">
        <input
          class="form-control"
          v-model="estado.tarefaTemp"
          ref="tarefaInput"
          required
          type="text"
          placeholder="Adicionar nova tarefa e pressione Enter"
        />
        <button class="btn btn-primary" type="submit">Inserir</button>
        <select @change="evento => estado.filtro = evento.target.value" class="form-select ms-2" style="max-width:180px">
          <option value="todos">Todas</option>
          <option value="pendentes">Pendentes</option>
          <option value="finalizadas">Finalizadas</option>
        </select>
      </div>
    </form>
    
    <div class="card" ref="listaRef">
      <div class="card-body">
        <ul class="list-group">
          <li class="list-group-item d-flex align-items-center" v-for="tarefa in getTarefasFiltradas()" :key="tarefa.titulo">
            <input @change="evento => tarefa.finalizada = evento.target.checked" :checked="tarefa.finalizada" :id="tarefa.titulo" type="checkbox">
            <div class="ms-3 flex-grow-1">
              <template v-if="!tarefa.editando">
                <div class="d-flex justify-content-between align-items-center">
                  <div>
                    <label :class="{ done: tarefa.finalizada } mb-0" @dblclick="iniciarEdicao(tarefa)">{{ tarefa.titulo }}</label>
                    <div><small class="text-muted">{{ tarefa.finalizada ? 'Finalizada' : 'Pendente' }}</small></div>
                  </div>
                  <div>
                    <span class="badge" :class="tarefa.finalizada ? 'bg-success' : 'bg-secondary'">{{ tarefa.finalizada ? '✓' : '…' }}</span>
                  </div>
                </div>
              </template>
              <template v-else>
                <input
                  class="form-control"
                  type="text"
                  v-model="tarefa.titulo"
                  @blur="finalizarEdicao(tarefa)"
                  @keyup.enter="finalizarEdicao(tarefa)"
                  autofocus
                />
              </template>
            </div>
          </li>
        </ul>

        <div v-if="!getTarefasFiltradas().length" class="text-center py-4 text-muted">
          <div style="font-size:36px">📭</div>
          <p class="mb-0">Nenhuma tarefa encontrada — adicione uma nova tarefa!</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
  :root{
    --bg-dark: #071627;
    --accent: #ff6b6b; /* coral */
    --accent-2: #ff9f43; /* orange */
    --card: rgba(255,255,255,0.02);
    --text: #f7fbff;
    --muted: #c9d6e3;
    --success: #2dd4bf;
  }

  body{
    font-family: 'Inter', system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
    background: linear-gradient(180deg,#061025 0%, #03101a 100%);
    color: var(--text);
  }

  .hero{
    background: linear-gradient(135deg,var(--accent) 0%, var(--accent-2) 100%);
    box-shadow: 0 6px 18px rgba(11,18,40,0.45);
  }

  /* temas alternativos via atributo data-theme no <html> */
  [data-theme="teal"]{
    --accent: #0ea5a4;
    --accent-2: #2dd4bf;
    --card: rgba(255,255,255,0.02);
    --text: #eafaf9;
    --muted: #9fe6de;
  }

  [data-theme="purple"]{
    --accent: #7c3aed;
    --accent-2: #06b6d4;
    --card: rgba(255,255,255,0.02);
    --text: #f7f7ff;
    --muted: #cfc7ff;
  }

  .card{
    background: var(--card);
    border: none;
    color: var(--text);
  }

  .btn-primary{
    background: var(--accent);
    border-color: transparent;
    box-shadow: 0 4px 12px rgba(108,99,255,0.12);
  }

  .btn-outline-light{
    color: var(--text);
    border-color: rgba(255,255,255,0.12);
  }

  .progress-bar{
    background: linear-gradient(90deg,var(--accent),var(--accent-2));
  }

  .badge.bg-success{
    background: var(--success) !important;
    color: #042022;
  }

  input.form-control{
    background: rgba(255,255,255,0.02);
    border: 1px solid rgba(255,255,255,0.04);
    color: var(--text);
  }

  input.form-control:focus{
    box-shadow: 0 0 0 0.2rem rgba(108,99,255,0.12);
    border-color: var(--accent);
  }

  .list-group-item{
    background: transparent;
    border: 1px solid rgba(255,255,255,0.03);
    margin-bottom: 8px;
    border-radius: 8px;
    transition: background .15s ease, transform .12s ease;
  }

  .list-group-item:hover{
    transform: translateY(-3px);
    background: rgba(255,255,255,0.01);
  }

  .done{
    text-decoration: line-through;
    color: var(--muted);
  }

  .text-muted{ color: rgba(255,255,255,0.6) !important; }

  /* pequenas melhorias responsivas */
  @media (max-width:576px){
    .hero{ padding: 1rem; }
    .input-group .form-select{ display:none; }
  }
</style>
