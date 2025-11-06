# 🛒 Sistema de Gerenciamento de Produtos em C

Este é um **sistema completo de gerenciamento de produtos**, feito inteiramente em **C**, utilizando **arquivos de texto** (`.txt`) para armazenar as informações dos produtos.  
Com ele, é possível **cadastrar, listar, editar e excluir produtos**, além de **ordenar os produtos por nome ou preço** com base em sua preferência.

---

## 🚀 Funcionalidades

- ✅ **Cadastrar produtos**
- ✅ **Listar produtos com opções de ordenação**
  - Nome (A–Z)
  - Nome (Z–A)
  - Preço (menor → maior)
  - Preço (maior → menor)
- ✅ **Editar produtos por ID**
- ✅ **Excluir produtos por ID**
- ✅ **Armazenamento permanente** no arquivo `produtos.txt`

---

## ⚙️ Como compilar e executar

### Compilação
```bash
gcc sistema_produtos.c -o sistema

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void bubbleSort(char v[], int n) {
    int i, j, aux;
    // Percorre o vetor várias vezes
    for (i = 0; i < n - 1; i++) {
        for (j = 0; j < n - i - 1; j++) {
            // Se o elemento atual for maior que o próximo, troca
            if (v[j] > v[j + 1]) {
                aux = v[j];
                v[j] = v[j + 1];
                v[j + 1] = aux;
            }
        }
    }
}

// Função para obter o último ID usado no arquivo
int obterUltimoID(const char *arquivo) {
    FILE *fp = fopen(arquivo, "r");
    if (!fp) return 0;

    int id, maiorID = 0;
    char linha[200];

    while (fgets(linha, sizeof(linha), fp)) {
        sscanf(linha, "|%d", &id);
        if (id > maiorID) maiorID = id;
    }

    fclose(fp);
    return maiorID;
}

// Função para gravar produtos
void gravarProdutos(const char *arquivo) {
    FILE *fp = fopen(arquivo, "a");
    if (!fp) {
        printf("Erro ao abrir o arquivo!\n");
        return;
    }

    int n;
    char nome[50], categoria[30], fornecedor[50];
    float preco;
    int quantidade;

    printf("Quantos produtos deseja cadastrar? ");
    scanf("%d", &n);

    int ultimoID = obterUltimoID(arquivo);

    for (int i = 0; i < n; i++) {
        printf("\nProduto %d:\n", i + 1);

        printf("Nome: ");
        scanf(" %[^\n]", nome);
        printf("Categoria: ");
        scanf(" %[^\n]", categoria);
        printf("Preço: ");
        scanf("%f", &preco);
        printf("Quantidade: ");
        scanf("%d", &quantidade);
        printf("Fornecedor: ");
        scanf(" %[^\n]", fornecedor);

        int id = ultimoID + i + 1;

        fprintf(fp, "|%d | %s | %s | %.2f | %d | %s |\n",
                id, nome, categoria, preco, quantidade, fornecedor);
    }

    fclose(fp);
    printf("\nProdutos gravados com sucesso!\n");
}

// Função para excluir produto por ID
void excluirProdutos(const char *arquivo, int idExcluir) {
    FILE *fpOriginal = fopen(arquivo, "r");
    FILE *fpTemp = fopen("temp.txt", "w");

    if (!fpOriginal || !fpTemp) {
        printf("Erro ao abrir os arquivos.\n");
        return;
    }

    char linha[200];
    int idAtual;

    while (fgets(linha, sizeof(linha), fpOriginal)) {
        sscanf(linha, "|%d", &idAtual);

        if (idAtual != idExcluir) {
            fputs(linha, fpTemp);
        }
    }

    fclose(fpOriginal);
    fclose(fpTemp);

    remove(arquivo);
    rename("temp.txt", arquivo);

    printf("Produto com ID %d excluído com sucesso!\n", idExcluir);
}

// Função para editar produto por ID
void editarProdutoPorID(const char *arquivo, int idEditar) {
    FILE *fpOriginal = fopen(arquivo, "r");
    FILE *fpTemp = fopen("temp.txt", "w");

    if (!fpOriginal || !fpTemp) {
        printf("Erro ao abrir os arquivos.\n");
        return;
    }

    char linha[200];
    int idAtual;
    int encontrado = 0;

    while (fgets(linha, sizeof(linha), fpOriginal)) {
        sscanf(linha, "|%d", &idAtual);

        if (idAtual == idEditar) {
            encontrado = 1;
            char nome[50], categoria[30], fornecedor[50];
            float preco;
            int quantidade;

            printf("Novo nome: ");
            scanf(" %[^\n]", nome);
            printf("Nova categoria: ");
            scanf(" %[^\n]", categoria);
            printf("Novo preço: ");
            scanf("%f", &preco);
            printf("Nova quantidade: ");
            scanf("%d", &quantidade);
            printf("Novo fornecedor: ");
            scanf(" %[^\n]", fornecedor);

            fprintf(fpTemp, "|%d | %s | %s | %.2f | %d | %s |\n",
                    idEditar, nome, categoria, preco, quantidade, fornecedor);
        } else {
            fputs(linha, fpTemp);
        }
    }

    fclose(fpOriginal);
    fclose(fpTemp);

    remove(arquivo);
    rename("temp.txt", arquivo);

    if (encontrado)
        printf("Produto com ID %d editado com sucesso!\n", idEditar);
    else
        printf("Produto com ID %d não encontrado.\n", idEditar);
}

// ----------------- LISTAR PRODUTOS COM ORDENAÇÃO -----------------
void listarProdutos(const char *arquivo) {
    FILE *fp = fopen(arquivo, "r");
    if (!fp) {
        printf("Erro ao abrir o arquivo para leitura.\n");
        return;
    }

    char linhas[200][200];
    char nomes[200][100];
    float precos[200];
    int total = 0;

    while (fgets(linhas[total], sizeof(linhas[total]), fp)) {
        char nome[100], categoria[50], fornecedor[50];
        int id, quantidade;
        float preco;

        sscanf(linhas[total], "|%d | %[^|] | %[^|] | %f | %d | %[^|] |",
               &id, nome, categoria, &preco, &quantidade, fornecedor);

        strcpy(nomes[total], nome);
        precos[total] = preco;
        total++;
    }
    fclose(fp);

    if (total == 0) {
        printf("Nenhum produto encontrado.\n");
        return;
    }

    int modo, i, j;
    char auxLinha[200], auxNome[100];
    float auxPreco;

    printf("\nComo deseja listar os produtos?\n");
    printf("1 - Nome (A-Z)\n");
    printf("2 - Nome (Z-A)\n");
    printf("3 - Preço (menor -> maior)\n");
    printf("4 - Preço (maior -> menor)\n");
    printf("Escolha: ");
    scanf("%d", &modo);

    // BUBBLE SORT (sua lógica)
    for (i = 0; i < total - 1; i++) {
        for (j = 0; j < total - i - 1; j++) {
            int troca = 0;
            if (modo == 1 && strcmp(nomes[j], nomes[j + 1]) > 0) troca = 1;
            if (modo == 2 && strcmp(nomes[j], nomes[j + 1]) < 0) troca = 1;
            if (modo == 3 && precos[j] > precos[j + 1]) troca = 1;
            if (modo == 4 && precos[j] < precos[j + 1]) troca = 1;

            if (troca) {
                strcpy(auxLinha, linhas[j]);
                strcpy(linhas[j], linhas[j + 1]);
                strcpy(linhas[j + 1], auxLinha);

                strcpy(auxNome, nomes[j]);
                strcpy(nomes[j], nomes[j + 1]);
                strcpy(nomes[j + 1], auxNome);

                auxPreco = precos[j];
                precos[j] = precos[j + 1];
                precos[j + 1] = auxPreco;
            }
        }
    }

    // Exibir o resultado ordenado
    printf("\n|--------------------------------------------------------|\n");
    printf("| ID | Nome | Categoria | Preço | Quantidade | Fornecedor |\n");
    for (i = 0; i < total; i++) {
        printf("%s", linhas[i]);
    }
}

int main() {
    const char *arquivo = "produtos.txt";
    int opcao, id;

    do {
        printf("\n1 - Cadastrar Produto\n2 - Listar Produtos\n3 - Excluir Produto\n4 - Editar Produto\n0 - Sair\nEscolha: ");
        scanf("%d", &opcao);

        switch (opcao) {
            case 1:
                gravarProdutos(arquivo);
                break;
            case 2:
                listarProdutos(arquivo);
                break;
            case 3:
                printf("Digite o ID do produto que deseja excluir: ");
                scanf("%d", &id);
                excluirProdutos(arquivo, id);
                break;
            case 4:
                printf("Digite o ID do produto que deseja editar: ");
                scanf("%d", &id);
                editarProdutoPorID(arquivo, id);
                break;
            case 0:
                printf("Saindo...\n");
                break;
            default:
                printf("Opção inválida!\n");
        }
    } while (opcao != 0);

    return 0;
}
