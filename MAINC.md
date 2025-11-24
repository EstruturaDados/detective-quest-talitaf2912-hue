/*
 Detective Quest - Nível Aventureiro (versão para entrega)
 Implementa:
  - Mapa da mansão como árvore binária (struct Sala)
  - BST para armazenar pistas encontradas (struct Pista)
  - Navegação interativa (e = esquerda, d = direita, s = sair)
  - Ao sair, exibe todas as pistas coletadas em ordem alfabética
 Compilar: gcc main.c -o detective
 Executar: ./detective
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_NOME 100

/* Estrutura para a sala (nó da árvore binária do mapa) */
typedef struct Sala {
    char nome[MAX_NOME];
    char pista[MAX_NOME]; // string vazia se não houver pista
    struct Sala* esquerda;
    struct Sala* direita;
} Sala;

/* Nó da árvore BST para pistas */
typedef struct Pista {
    char conteudo[MAX_NOME];
    struct Pista* esquerda;
    struct Pista* direita;
} Pista;

/* --- Funções para Sala (mapa) --- */
Sala* criarSala(const char* nome, const char* pista) {
    Sala* s = (Sala*) malloc(sizeof(Sala));
    if (!s) { perror("malloc"); exit(EXIT_FAILURE); }
    strncpy(s->nome, nome, MAX_NOME-1);
    s->nome[MAX_NOME-1] = '\0';
    if (pista) {
        strncpy(s->pista, pista, MAX_NOME-1);
        s->pista[MAX_NOME-1] = '\0';
    } else {
        s->pista[0] = '\0';
    }
    s->esquerda = s->direita = NULL;
    return s;
}

void liberarMapa(Sala* raiz) {
    if (!raiz) return;
    liberarMapa(raiz->esquerda);
    liberarMapa(raiz->direita);
    free(raiz);
}

/* --- Funções para BST de pistas --- */
Pista* criarPista(const char* texto) {
    Pista* p = (Pista*) malloc(sizeof(Pista));
    if (!p) { perror("malloc"); exit(EXIT_FAILURE); }
    strncpy(p->conteudo, texto, MAX_NOME-1);
    p->conteudo[MAX_NOME-1] = '\0';
    p->esquerda = p->direita = NULL;
    return p;
}

int comparar(const char* a, const char* b) {
    return strcmp(a, b);
}

Pista* inserirPista(Pista* raiz, const char* texto) {
    if (!raiz) return criarPista(texto);
    int cmp = comparar(texto, raiz->conteudo);
    if (cmp < 0) raiz->esquerda = inserirPista(raiz->esquerda, texto);
    else if (cmp > 0) raiz->direita = inserirPista(raiz->direita, texto);
    // se igual, não insere duplicata
    return raiz;
}

void emOrdemPistas(Pista* raiz) {
    if (!raiz) return;
    emOrdemPistas(raiz->esquerda);
    printf(" - %s\n", raiz->conteudo);
    emOrdemPistas(raiz->direita);
}

void liberarPistas(Pista* raiz) {
    if (!raiz) return;
    liberarPistas(raiz->esquerda);
    liberarPistas(raiz->direita);
    free(raiz);
}

/* --- Exploração interativa --- */
void explorarSalasComPistas(Sala* atual, Pista** arvorePistas) {
    char opcao[10];
    while (atual != NULL) {
        printf("\nVocê está em: %s\n", atual->nome);
        if (strlen(atual->pista) > 0) {
            printf("Você encontrou uma pista: \"%s\"\n", atual->pista);
            *arvorePistas = inserirPista(*arvorePistas, atual->pista);
        } else {
            printf("Esta sala não contém pista.\n");
        }
        printf("Escolha: (e) esquerda, (d) direita, (s) sair: ");
        if (!fgets(opcao, sizeof(opcao), stdin)) return;
        // remover newline
        opcao[strcspn(opcao, "\n")] = 0;
        if (strcmp(opcao, "e") == 0) {
            if (atual->esquerda) atual = atual->esquerda;
            else {
                printf("Não há caminho à esquerda. Você permanece na mesma sala.\n");
            }
        } else if (strcmp(opcao, "d") == 0) {
            if (atual->direita) atual = atual->direita;
            else {
                printf("Não há caminho à direita. Você permanece na mesma sala.\n");
            }
        } else if (strcmp(opcao, "s") == 0) {
            printf("Saindo da exploração.\n");
            return;
        } else {
            printf("Opção inválida. Use 'e', 'd' ou 's'.\n");
        }
    }
}

/* Monta um mapa fixo - você pode adaptar este mapa conforme desejar */
Sala* montarMapa() {
    Sala* hall = criarSala("Hall de Entrada", "Pegadas de Lama");
    Sala* salaEstar = criarSala("Sala de Estar", "Chave perdida");
    Sala* biblioteca = criarSala("Biblioteca", "Livro com página faltando");
    Sala* quarto = criarSala("Quarto", "Lençol manchado");
    Sala* cozinha = criarSala("Cozinha", "Gaveta perdida");
    Sala* jardim = criarSala("Jardim", "Pegada misteriosa");
    // ligar nós
    hall->esquerda = salaEstar;
    hall->direita = biblioteca;
    salaEstar->esquerda = quarto;
    salaEstar->direita = cozinha;
    biblioteca->direita = jardim;
    return hall;
}

int main() {
    printf("=== Detective Quest - Coleta de Pistas ===\n");
    printf("Navegue pela mansão e colete pistas. Ao sair, veremos todas as pistas coletadas.\n");
    printf("Instruções: digite 'e' para esquerda, 'd' para direita, 's' para sair e ENTER.\n");

    Sala* mapa = montarMapa();
    Pista* arvorePistas = NULL;

    explorarSalasComPistas(mapa, &arvorePistas);

    printf("\nPistas coletadas em ordem alfabética:\n");
    if (arvorePistas) emOrdemPistas(arvorePistas);
    else printf("Nenhuma pista coletada.\n");

    liberarPistas(arvorePistas);
    liberarMapa(mapa);
    return 0;
}
