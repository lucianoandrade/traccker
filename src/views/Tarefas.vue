<template>
  <FormularioMain @aoSalvarTarefa="salvarTarefa" />
  <div class="lista">
    <div class="field">
      <p class="control has-icons-left">
        <input
          class="input"
          type="text"
          placeholder="Digite para filtrar"
          v-model="filtro"
        />
        <span class="icon is-small is-left">
          <i class="fas fa-search"></i>
        </span>
      </p>
    </div>
    <TarefaMain 
      v-for="tarefa in tarefas" 
      :key="tarefa.id" 
      :tarefa="tarefa"
      @ao-tarefa-clicada="selecionarTarefa"
    />
    <BoxMain v-if="listaEstaVazia">
      Você não está muito produtivo hoje :(
    </BoxMain>
    <Modal :mostrar="tarefaSelecionada != null">
      <template v-slot:cabecalho>
        <p class="modal-card-title">Editando uma tarefa</p>
        <button @click="fecharModal" class="delete" aria-label="close"></button>
      </template>
      <template v-slot:corpo>
        <div class="field">
          <label for="descricaoDaTarefa" class="label"> Descrição </label>
          <input
            type="text"
            class="input"
            v-model="tarefaSelecionada.descricao"
            id="descricaoDaTarefa"
          />
        </div>
      </template>
      <template v-slot:rodape>
        <button @click="alterarTarefa" class="button is-success">
          Salvar alterações
        </button>
        <button @click="fecharModal" class="button">Cancelar</button>
      </template>
    </Modal>
  </div>
</template>

<script lang="ts">
import { useStore } from "@/store";
import { computed, defineComponent, ref, watchEffect } from "vue";
import FormularioMain from "../components/FormularioMain.vue";
import TarefaMain from "../components/TarefaMain.vue";
import BoxMain from "../components/BoxMain.vue";
import Modal from "../components/Modal.vue";
import { ALTERAR_TAREFA, CADASTRAR_TAREFA, OBTER_PROJETOS, OBTER_TAREFAS } from "@/store/tipo-acoes";
import ITarefa from "@/interfaces/ITarefa";

export default defineComponent({
  name: "TarefaView",
  components: {
    FormularioMain,
    TarefaMain,
    BoxMain,
    Modal
  },
  data() {
    return {
      tarefaSelecionada: null as ITarefa | null
    }
  },
  computed: {
    listaEstaVazia(): boolean {
      return this.tarefas.length === 0;
    },
  },
  methods: {
    salvarTarefa(tarefa: ITarefa) {
        this.store.dispatch(CADASTRAR_TAREFA, tarefa)
    },
    selecionarTarefa(tarefa: ITarefa) {
      this.tarefaSelecionada = tarefa
    },
    fecharModal() {
      this.tarefaSelecionada = null
    },
    alterarTarefa() {
      this.store
        .dispatch(ALTERAR_TAREFA, this.tarefaSelecionada)
        .then(() => this.fecharModal());
    },
  },
  setup() {
    const store = useStore();
    store.dispatch(OBTER_TAREFAS);
    store.dispatch(OBTER_PROJETOS);

    const filtro = ref("");

    // const tarefas = computed(() =>
    //   store.state.tarefa.tarefas.filter(
    //     (t) => !filtro.value || t.descricao.includes(filtro.value)
    //   )
    // );
    watchEffect(() => {
      store.dispatch(OBTER_TAREFAS, filtro.value);
    });
    
    return {
      tarefas: computed(() => store.state.tarefas),
      store,
      filtro
    };
  },
});
</script>
