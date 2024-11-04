<template>
  <v-container>

    <!-- Linha para o botão de adicionar produto, alinhado à direita -->
    <v-row class="pb-1">
      <v-col class="d-flex justify-space-between">
        <h1>Listagem de Produtos</h1>

        <v-btn color="primary" @click="openDialog()">Adicionar Produto</v-btn>
      </v-col>
    </v-row>
    <!-- Linha para o campo de busca separado da tabela -->
    <v-row class="pb-3">
      <v-col class="d-flex justify-end">
        <v-text-field v-model="search" label="Buscar produtos" prepend-inner-icon="mdi-magnify" single-line
          hide-details></v-text-field>
      </v-col>
    </v-row>

    <!-- Tabela de produtos -->
    <v-data-table :headers="headers" :items="produtos" item-key="id" class="elevation-1" :search="search"
      :items-per-page="5">
      <template v-slot:item.actions="{ item }">
        <v-btn color="blue" icon="mdi-pencil" class="ma-1" size="x-small" @click="openDialog(item)"></v-btn>
        <v-btn color="red" icon="mdi-delete" class="ma-1" size="x-small" @click="deleteProduto(item.id)"></v-btn>
      </template>

      <template v-slot:item.quantidade="{ item }">
        <span :class="{ 'text--error': item.quantidade === 0 }">
          {{ item.quantidade }}
        </span>
      </template>
    </v-data-table>

    <!-- Modal para adicionar ou editar produto -->
    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">{{ editing ? "Editar Produto" : "Adicionar Produto" }}</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field v-model="produto.nome" label="Nome" required></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="produto.descricao" label="Descrição"></v-text-field>
              </v-col>
              <v-col cols="12" sm="6">
  <v-text-field
    v-model="produto.quantidade"
    label="Quantidade"
    type="number"
    required
    @input="produto.quantidade = produto.quantidade < 0 ? 0 : produto.quantidade"
  ></v-text-field>
</v-col>

<v-col cols="12" sm="6">
  <v-text-field
    v-model="produto.preco_unitario"
    label="Preço Unitário"
    type="number"
    required
    @input="produto.preco_unitario = produto.preco_unitario < 0 ? 0 : produto.preco_unitario"
  ></v-text-field>
</v-col>

            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="closeDialog">Cancelar</v-btn>
          <v-btn color="blue darken-1" text @click="saveProduto">Salvar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      search: "",
      headers: [
        { title: "ID", key: "id" },
        { title: "Nome", key: "nome" },
        { title: "Descrição", key: "descricao" },
        { title: "Quantidade", key: "quantidade" },
        { title: "Preço Unitário", key: "preco_unitario", align: "right" },
        { title: "Ações", key: "actions", sortable: false }
      ],
      produtos: [],
      dialog: false,
      editing: false,
      produto: {
        id: null,
        nome: "",
        descricao: "",
        quantidade: 0,
        preco_unitario: 0.0
      }
    };
  },
  mounted() {
    this.fetchProdutos();
  },
  methods: {
    async fetchProdutos() {
      try {
        const response = await fetch('http://localhost:4000/produtos'); // Substitua pela URL da sua API
        if (!response.ok) {
          throw new Error("Erro ao buscar os produtos");
        }
        const data = await response.json();
        this.produtos = data;
      } catch (error) {
        console.error("Erro ao carregar produtos:", error);
      }
    },
    openDialog(item = null) {
      this.editing = !!item;
      this.dialog = true;
      if (item) {
        this.produto = { ...item };
      } else {
        this.resetProduto();
      }
    },
    closeDialog() {
      this.dialog = false;
    },
    resetProduto() {
      this.produto = {
        id: null,
        nome: "",
        descricao: "",
        quantidade: 0,
        preco_unitario: 0.0
      };
    },
    async saveProduto() {
      if (this.editing) {
        // Editar produto existente
        try {
          const response = await fetch(`http://localhost:4000/produtos/${this.produto.id}`, {
            method: 'PUT',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.produto)
          });
          if (!response.ok) {
            throw new Error("Erro ao atualizar o produto");
          }
          this.fetchProdutos();
          this.closeDialog();
        } catch (error) {
          console.error("Erro ao salvar produto:", error);
        }
      } else {
        // Adicionar novo produto
        try {
          const response = await fetch('http://localhost:4000/produtos', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.produto)
          });
          if (!response.ok) {
            throw new Error("Erro ao adicionar o produto");
          }
          this.fetchProdutos();
          this.closeDialog();
        } catch (error) {
          console.error("Erro ao salvar produto:", error);
        }
      }
    },
    async deleteProduto(id) {
      try {
        const response = await fetch(`http://localhost:4000/produtos/${id}`, {
          method: 'DELETE'
        });
        if (!response.ok) {
          throw new Error("Erro ao excluir o produto");
        }
        this.fetchProdutos();
      } catch (error) {
        console.error("Erro ao excluir produto:", error);
      }
    }
  }
};
</script>

<style scoped>
.text--error {
  color: red;
}
</style>
