<template>
  <div class="container">
    <h1>Gestão de Peças de Motos</h1>

    <!-- Abas -->
    <div class="tabs">
      <button :class="{ active: abaAtiva === 'cadastro' }" @click="abaAtiva = 'cadastro'">
        Cadastro
      </button>
      <button :class="{ active: abaAtiva === 'consulta' }" @click="abaAtiva = 'consulta'">
        Consulta
      </button>
    </div>

    <!-- Conteúdo das abas -->
    <div v-if="abaAtiva === 'cadastro'">
      <form @submit.prevent="gravarPeca">
        <div class="form-group">
          <label>Número da Peça:</label>
          <input v-model="peca.NumeroPeca" type="number" required />
        </div>

        <div class="form-group">
          <label>Nome da Peça:</label>
          <input v-model="peca.NomePeca" type="text" required />
        </div>

        <div class="form-group">
          <label>Categoria:</label>
          <input v-model="peca.CategoriaPeca" type="text" required />
        </div>

        <div class="form-group">
          <label>Fabricante:</label>
          <input v-model="peca.Fabricante" type="text" />
        </div>

        <div class="form-group">
          <label>Preço:</label>
          <input v-model="peca.Preco" type="number" step="0.01" required />
        </div>

        <button type="submit">Salvar</button>
      </form>

      <p v-if="mensagem" class="msg">{{ mensagem }}</p>
    </div>

    <div v-else>
      <button @click="carregarPecas">Atualizar Lista</button>
      <table v-if="pecas.length" class="grid">
        <thead>
          <tr>
            <th>Número</th>
            <th>Nome</th>
            <th>Categoria</th>
            <th>Fabricante</th>
            <th>Preço</th>
          </tr>
        </thead>
        <tbody>
     <tr v-for="p in pecas" :key="p.NumeroPeca">
          <td>{{ p.NumeroPeca }}</td>
          <td>{{ p.NomePeca }}</td>
          <td>{{ p.CategoriaPeca }}</td>
          <td>{{ p.Fabricante }}</td>
          <td>{{ p.Preco }}</td>
          <td>
            <button class="delete-btn" @click="deletarPeca(p.NumeroPeca)">
              🗑️
            </button>
          </td>
        </tr>
        </tbody>
      </table>

      <p v-else>Nenhuma peça cadastrada.</p>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from "vue";

const abaAtiva = ref("cadastro"); // aba inicial

// Dados do formulário
const peca = reactive({
  NumeroPeca: 0,
  NomePeca: "",
  CategoriaPeca: "",
  Fabricante: "",
  Preco: 0.0,
});

const mensagem = ref("");

const pecas = ref([]);

async function gravarPeca() {
  try {
    console.log(peca)
    const response = await fetch("https://localhost:44385/Gravar", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(peca),
    });

    const result = await response.text();

    if (response.ok) {
      mensagem.value = result;
      Object.keys(peca).forEach((k) => (peca[k] = ""));
      if (abaAtiva.value === "consulta") {
        await carregarPecas();
      }
    } else {
      mensagem.value = `Erro: ${result}`;
    }
  } catch (error) {
    console.error(error);
    mensagem.value = "Erro de conexão com o servidor.";
  }
}

async function carregarPecas() {
  try {
    const response = await fetch("https://localhost:44385/Listar"); // GET /Listar
    if (response.ok) {
      const data = await response.json();
      pecas.value = data.map(p => ({
        NumeroPeca: p.numeroPeca,
        NomePeca: p.nomePeca,
        CategoriaPeca: p.categoriaPeca,
        Fabricante: p.fabricante,
        Preco: p.preco,
      }));
    } else {
      pecas.value = [];
      alert("Erro ao carregar peças");
    }
  } catch (error) {
    console.error(error);
    pecas.value = [];
    alert("Erro de conexão com o servidor.");
  }
}

watch(abaAtiva, (novaAba) => {
  if (novaAba === "consulta") {
    carregarPecas();
  }
});

// async function deletarPeca(numeroPeca) {
//   if (!confirm("Deseja realmente deletar esta peça?")) return;

//   try {
//     const response = await fetch(`https://localhost:44385/Delete/${numeroPeca}`, {
//       method: "DELETE",
//     });

//     const result = await response.text();

//     if (response.ok) {
//       alert(result); // mostra mensagem do backend
//       carregarPecas(); // atualiza o grid
//     } else {
//       alert(`Erro: ${result}`);
//     }
//   } catch (error) {
//     console.error(error);
//     alert("Erro de conexão com o servidor.");
//   }
// }
</script>


<style>
.container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 1rem;
  border-radius: 8px;
  background: #f9f9f9;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  margin-bottom: 1rem;
}

.tabs {
  display: flex;
  justify-content: center;
  margin-bottom: 1rem;
}

.tabs button {
  padding: 0.5rem 1rem;
  border: none;
  background: #eee;
  cursor: pointer;
  margin: 0 0.2rem;
  border-radius: 6px;
}

.tabs button.active {
  background: #42b883;
  color: white;
}

.form-group {
  margin-bottom: 1rem;
  display: flex;
  flex-direction: column;
}

input {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
}

button {
  padding: 0.7rem;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background: #369870;
}

.msg {
  margin-top: 1rem;
  text-align: center;
  font-weight: bold;
}

.grid {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.grid th,
.grid td {
  border: 1px solid #ccc;
  padding: 0.5rem;
  text-align: left;
}

.grid th {
  background: #42b883;
  color: white;
}

.delete-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: 1.2rem;
}

.delete-btn:hover {
  color: red;
}
</style>
