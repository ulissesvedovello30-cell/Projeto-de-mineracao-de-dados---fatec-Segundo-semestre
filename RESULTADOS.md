# Resultados da análise (para o relatório e os slides)

Base: acidentes em rodovias federais da PRF, 2025 (`datatran2025.csv`, agrupados por ocorrência).
Código: `src/analise_completa.py`. Tabelas e gráficos: pasta `resultados/`.

## 1. Base de dados
- 72.529 registros e 30 atributos originais. Removida 1 linha sem `classificacao_acidente` → **72.528 acidentes**.
- 0 ids duplicados. Nulos restantes só em `regional`, `delegacia` e `uop` (colunas administrativas, não usadas).
- Atributos criados: `mes` (de `data_inversa`) e `horas` (de `horario`).
- Gravidade: Com Vítimas Feridas 56.181 (77,5%) · Sem Vítimas 11.138 (15,4%) · Com Vítimas Fatais 5.209 (7,2%) (n = 72.528 após a limpeza; a base original tinha 1 nulo).
- Causas mais comuns: ausência de reação do condutor (11.469), reação tardia ou ineficiente (10.799), acessar a via sem observar outros veículos (7.097).
- Fase do dia: pleno dia 40.375 · plena noite 24.781 · anoitecer 3.926 · amanhecer 3.447.

## 2. Agrupamento (K-Means)
- Atributos: `pessoas`, `veiculos`, `feridos`, `mortos`, padronizados com `StandardScaler` (as escalas diferem e o K-Means usa distância).
- **Cotovelo (Elbow):** inércia cai de 290 mil (k=1) para 144 mil (k=4) e 123 mil (k=5); a queda vira quase metade depois de k=4 → dobra em **k=4**.
- **Silhouette** (amostra de 10 mil, k=2..8): k=2 → 0,632 · k=3 → 0,549 · k=4 → 0,485 · k=5 → 0,495 · k=6 → 0,437 · k=7 → 0,445 · k=8 → 0,453.
- **Escolha: k = 4.** Com k=2 o melhor Silhouette (0,632) só separa "tem morte" de "não tem morte" (grupo 1 = 98% fatais), o que é óbvio porque `mortos` é um dos atributos. O k=4 combina o cotovelo com Silhouette ainda aceitável (0,485) e dá grupos interpretáveis.

| Grupo | Acidentes | % | Pessoas | Veículos | Feridos | Mortos | % fatais | Interpretação |
|---|---|---|---|---|---|---|---|---|
| 0 | 52.635 | 72,6% | 1,95 | 1,60 | 0,99 | 0,00 | 0% | Acidentes leves/comuns, sem morte |
| 1 | 5.078 | 7,0% | 3,10 | 2,43 | 0,58 | 1,15 | 100% | Acidentes fatais (≈1 morte) |
| 2 | 131 | 0,2% | 38,67 | 2,54 | 11,17 | 0,66 | 26% | Muitas pessoas envolvidas (ônibus/tragédias) |
| 3 | 14.684 | 20,2% | 4,40 | 3,26 | 1,83 | 0,01 | 0,7% | Acidentes maiores (vários veículos e feridos), quase sem morte |

Limitações para a análise crítica: `pessoas` já inclui `mortos` e `feridos` (colunas correlacionadas); `mortos` faz o K-Means separar a gravidade de forma óbvia; o Silhouette foi calculado em amostra de 10.000 registros.

## 3. Regras de associação (Apriori)
- Itens: 5 atributos categóricos (`causa_acidente`, `classificacao_acidente`, `fase_dia`, `condicao_metereologica`, `tipo_acidente`) transformados em 102 itens (coluna=valor); cada acidente é uma "cesta".
- Parâmetros: suporte mínimo 1%, confiança mínima 30%, lift > 1, itemsets de até 4 itens → 493 itemsets frequentes e **680 regras**.
- Para acidentes **fatais** (só 7% da base) usamos suporte mínimo 0,3%, confiança ≥ 15% e lift > 1,5 → 22 regras.

Principais regras (suporte; confiança; lift):
- Animais na pista + plena noite → atropelamento de animal (1,1%; 88%; 56,4).
- Falhas mecânicas/elétricas + incêndio → sem vítimas (2,1%; 97,5%; 6,3).
- Pedestre andava na pista + atropelamento de pedestre → **fatal** (0,3%; 42,7%; 5,9).
- Transitar na contramão + colisão frontal → **fatal** (0,9%; 38,5%; 5,4). À noite: 39,2% (lift 5,5).
- Céu claro + colisão frontal → **fatal** (1,3%; 30,2%; 4,2).
- Queda de ocupante de veículo → com vítimas feridas (4,5%; 95,3%; 1,23).

Leitura: acidentes fatais são só 7,2% do total, mas a base rate sobe para 30–43% (lift 4–6) quando há colisão frontal, contramão ou pedestre na pista.

## 4. Arquivos gerados
`resultados/01_elbow.png`, `02_silhueta.png`, `03_perfil_grupos.png`, `04_regras_suporte_confianca.png`, `perfil_grupos.csv`, `gravidade_por_grupo.csv`, `regras_todas.csv`, `regras_fatais.csv`, `regras_sem_vitimas.csv`, `regras_feridas.csv`, `elbow.csv`, `silhueta.csv`.

## 5. Como reproduzir
```bash
pip install -r requirements.txt
python src/analise_completa.py 4
```
O número no final é o k do K-Means (sem ele, usa o melhor Silhouette, que é k=2).
