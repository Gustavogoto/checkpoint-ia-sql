# Checkpoint de IA: Assistente Texto → SQL com Qwen3-0.6B

**Grupo:** Nome 1 (RM) · Nome 2 (RM) · Nome 3 (RM) · Nome 4 (RM) · Nome 5 (RM)

## O problema
Quem não sabe SQL não consegue tirar informações de um banco de dados sozinho. Treinamos uma LLM pequena para **transformar perguntas em português em consultas SQL** sobre o banco fixo de uma loja online. A consulta é **executada** e o resultado volta para o usuário. O tema é o "Texto → SQL em um schema fixo", da lista de sugestões.

```
Pergunta:  "Quais são os 3 produtos mais vendidos?"
SQL:       SELECT p.nome, SUM(i.quantidade) AS unidades FROM itens_pedido i
           JOIN pedidos pe ON i.pedido_id = pe.id JOIN produtos p ON i.produto_id = p.id
           WHERE pe.status <> 'cancelado' GROUP BY p.id ORDER BY unidades DESC LIMIT 3;
Resultado: tabela com os 3 produtos e as unidades vendidas
```

## A LLM escolhida: Qwen3-0.6B
- **Está na lista** de modelos sugeridos pelo professor.
- **Tamanho:** tem 0,6 bilhão de parâmetros. Com LoRA, o treino cabe com folga na GPU T4 gratuita do Google Colab. Treinar todos os pesos estourou a memória da T4 (15 GB), por isso usamos LoRA.
- **Conhecimento prévio:** já viu bastante código, inclusive SQL, então aprende rápido o padrão do nosso banco.

## Como foi feito
Tudo está em **um único notebook**, [`checkpoint_ia_texto_sql.ipynb`](checkpoint_ia_texto_sql.ipynb), no mesmo formato da Aula 8:

| Etapa | O que faz |
|---|---|
| 1 | Verifica a GPU (`nvidia-smi`) |
| 3 | Prompt padronizado (`### Banco / ### Pergunta / ### SQL`) e funções `carregar` e `gerar` |
| 4 | Cria o banco SQLite da loja e executa as consultas em modo somente leitura |
| 5 | Monta o dataset de treino, validação, teste e teste manual |
| 6–7 | Avalia o modelo base, **antes** do treino |
| 8 | Fine-tuning com **LoRA** na GPU (forward → loss → backward → step), com validação a cada época |
| 9 | Avalia **depois** do treino e compara |
| 10 | Testa parâmetros de geração (temperature, top_p) e de treino (taxa de aprendizado, épocas, rank do LoRA) |
| 11 | Mostra onde o modelo erra |
| 13 | Frontend web (Gradio) para o usuário fazer perguntas |

**Banco:** 5 tabelas, com 300 clientes, 8 categorias, 56 produtos, 2.000 pedidos e cerca de 5.000 itens. Existe uma regra de negócio que **não está no prompt** e o modelo precisa aprender com os exemplos: *faturamento e vendas não contam pedidos cancelados*.

**Dataset:**
- São 26 tipos de pergunta, cada um com 4 formas de perguntar.
- O **treino** (1.155 exemplos) usa 3 dessas formas.
- **Validação e teste** usam a 4ª forma, que o modelo nunca viu no treino.
- O **teste manual** tem 30 perguntas escritas à mão, de jeitos bem diferentes.

**Avaliação:** o SQL gerado é **executado** e o resultado é comparado com o da consulta correta. Uma consulta escrita de outro jeito, mas que devolve o mesmo resultado, conta como acerto. Também medimos a % de SQL válido e o tempo por pergunta.

## Resultados
> Cole aqui a tabela que a Etapa 10.3 imprime.

| Experimento | Acurácia no teste | Acurácia no teste manual | SQL válido |
|---|---|---|---|
| Modelo base (antes) | | | |
| Modelo treinado (depois) | | | |

![Antes x depois](prints/antes_depois.png)

## GPU e frontend
![GPU](prints/gpu.png)
![Frontend](prints/frontend.png)

## Como rodar
1. Abra o notebook no Google Colab.
2. Vá em **Ambiente de execução → Alterar o tipo → GPU T4**.
3. Rode as células de cima para baixo. O treino leva cerca de 10 a 20 minutos.

## Referências
- Qwen Team. *Qwen3 Technical Report*. 2025.
- Allal, L. B. et al. *SmolLM2: When Smol Goes Big*. 2025 (base da Aula 8).
- Yu, T. et al. *Spider: A Large-Scale Human-Labeled Dataset for Text-to-SQL*. 2018.
- Hu, E. et al. *LoRA: Low-Rank Adaptation of Large Language Models*. 2021.
- Hugging Face Transformers e PEFT: documentação oficial.
