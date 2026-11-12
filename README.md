# Mineração de Dados – Acidentes em Rodovias Federais (PRF)

Projeto da disciplina de Mineração de Dados.

**Integrantes:** Ulisses Vedovello Macedo, Vitor Rafael, Ricardo

## Descrição
Análise de dados abertos da Polícia Rodoviária Federal (PRF) sobre acidentes em rodovias federais brasileiras, aplicando:
- **Agrupamento (clustering)** para identificar perfis de acidentes;
- **Regras de associação (Apriori)** para encontrar combinações de fatores ligadas à gravidade.

**Fonte dos dados:** https://www.gov.br/prf/pt-br/acesso-a-informacao/dados-abertos/dados-abertos-da-prf

## Estrutura
- `data/raw/` – CSVs originais da PRF (não versionados)
- `data/processed/` – dados tratados
- `notebooks/` – análises (exploração, clustering, associação)
- `src/` – scripts auxiliares
- `relatorio/` – relatório final
- `apresentacao/` – slides
