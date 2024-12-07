<template>
  <div id="app">
    <h1>Orçamento por Categorias</h1>

    <div class="presets">
      <button @click="ativarPreset('alice')">Preset - Alice</button>
      <button @click="ativarPreset('lucas')">Preset - Lucas</button>
    </div>

    <form @submit.prevent="calcularValores">
  <template v-if="presetAtivo === 'alice'">
    <div v-for="(categoria, index) in categoriasAlice" :key="index">
      <label :for="categoria.nome">{{ categoria.nome }} (%):</label>
      <input
        type="number"
        v-model.number="categoria.percentual"
        min="0"
        max="100"
        step="0.01"
        required
      />
    </div>
  </template>

  <template v-else-if="presetAtivo === 'lucas'">
    <div v-for="(categoria, index) in categoriasLucas" :key="index">
      <label :for="categoria.nome">{{ categoria.nome }} (%):</label>
      <input
        type="number"
        v-model.number="categoria.percentual"
        min="0"
        max="100"
        step="0.01"
        required
      />
    </div>
  </template>

  <div>
    <label for="receita">Receita Total:</label>
    <input
      type="number"
      v-model.number="receita"
      min="0"
      step="0.01"
      placeholder="Informe o valor da receita"
      required
    />
  </div>

  <button type="submit">Calcular Valores</button>
</form>

    <div v-if="dizimo !== null" class="card-resultados">
      <div class="resultado-item">
        <p><strong>Dízimo (10%):</strong></p>
        <p class="valor">R$ {{ dizimo.toFixed(2) }}</p>
      </div>
      <div class="resultado-item">
        <p><strong>Receita após Dízimo:</strong></p>
        <p class="valor">R$ {{ receitaAposDizimo.toFixed(2) }}</p>
      </div>
    </div>

    <table v-if="valoresCalculados.length" id="tabela">
      <thead>
        <tr>
          <th>Categoria</th>
          <th>Porcentagem (%)</th>
          <th>Valor (R$)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(valor, index) in valoresCalculados" :key="index">
          <td>{{ valor.nome }}</td>
          <td>{{ valor.percentual }}</td>
          <td>{{ valor.valor.toFixed(2) }}</td>
        </tr>
      </tbody>
    </table>

    <button v-if="valoresCalculados.length" @click="exportarTabela">Exportar Tabela como PNG</button>
  </div>
</template>

<script>
import domtoimage from "dom-to-image";

export default {
  data() {
    return {
      presetAtivo: "alice",
      categoriasAlice: [
        { nome: "Investimentos", percentual: 0 },
        { nome: "Custos fixos", percentual: 0 },
        { nome: "Conforto", percentual: 0 },
        { nome: "Prazeres", percentual: 0 },
        { nome: "Aprendizado", percentual: 0 },
      ],
      categoriasLucas: [
        { nome: "Custo", percentual: 0 },
        { nome: "Investimento", percentual: 0 },
        { nome: "Se precisar", percentual: 0 },
        { nome: "Lazer", percentual: 0 },
      ],
      receita: 0,
      dizimo: null,
      receitaAposDizimo: null,
      valoresCalculados: [],
    };
  },
  methods: {
    ativarPreset(preset) {
      this.presetAtivo = preset;
      if (preset === "alice") {
        this.categoriasAlice = [
          { nome: "Investimentos", percentual: 46.3 },
          { nome: "Custos fixos", percentual: 28.7 },
          { nome: "Conforto", percentual: 10.0 },
          { nome: "Prazeres", percentual: 10.0 },
          { nome: "Aprendizado", percentual: 5.0 },
        ];
      } else if (preset === "lucas") {
        this.categoriasLucas = [
          { nome: "Custo", percentual: 42.00 },
          { nome: "Investimento", percentual: 52.00 },
          { nome: "Se precisar", percentual: 3.00 },
          { nome: "Lazer", percentual: 3.00 },
        ];
      }
    },
    calcularValores() {
      this.dizimo = this.receita * 0.1;
      this.receitaAposDizimo = this.receita - this.dizimo;

      const categoriasAtuais = this.presetAtivo === "alice" ? this.categoriasAlice : this.categoriasLucas;

      this.valoresCalculados = categoriasAtuais.map((categoria) => {
        return {
          nome: categoria.nome,
          percentual: categoria.percentual,
          valor: (this.receitaAposDizimo * categoria.percentual) / 100,
        };
      });
    },
    exportarTabela() {
      const tabela = document.getElementById("tabela");
      domtoimage
        .toPng(tabela)
        .then((dataUrl) => {
          const link = document.createElement("a");
          link.download = "tabela_orcamento.png";
          link.href = dataUrl;
          link.click();
        })
        .catch((error) => {
          console.error("Erro ao exportar tabela como PNG:", error);
        });
    },
  },
};
</script>

<style>
#app {
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
}

.presets {
  display: flex;
  justify-content: space-around;
  margin-bottom: 1rem;
}

form div {
  margin-bottom: 1rem;
}

.card-resultados {
  display: flex;
  justify-content: space-around;
  margin-top: 1.5rem;
  padding: 1rem;
  background-color: #f5f5f5;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.resultado-item {
  text-align: center;
}

.resultado-item p {
  margin: 0.5rem 0;
}

.valor {
  font-size: 1.2em;
  color: #4CAF50;
  font-weight: bold;
}

table {
  width: 100%;
  margin-top: 1.5rem;
  border-collapse: collapse;
}

th,
td {
  padding: 8px;
  border: 1px solid #ddd;
}

button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

button:hover {
  background-color: #45a049;
}
</style>
