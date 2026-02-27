# README — Segmentação de Tumores Cerebrais (BraTS 2020/2024)

🇺🇸 [Read in English](README.md) | 🇧🇷 Português

---

## 1) Visão geral do trabalho
Este repositório organiza todos os códigos e artefatos usados para avaliar o impacto das diferentes sequências de RM (T1n, T1c, T2w, T2f/FLAIR) em cenários cross-dataset entre BraTS 2020 e BraTS 2024.  

A estrutura foi pensada para:  
- **Criar e visualizar** os splits dos datasets (train/val/test).  
- **Treinar** modelos (baseline e mixdataset) por sequência.  
- **Inferir/avaliar cross-dataset** (treina em A → testa em B e vice-versa) por sequência.  
- **Consolidar resultados** (métricas globais e por quartil de tamanho do tumor) e mostrar exemplos visuais por sequência.

> Em resumo, o repositório guia o fluxo completo: preparação de dados → treinamento → inferência → análise e tabela final de resultados.

---
## 2) Bases de dados
- **BraTS 2020**: Base do desafio BraTS edição 2020 amplamente utilizada na literatura, pode ser encontrada no link: 
https://www.med.upenn.edu/cbica/brats2020/data.html ou https://www.kaggle.com/datasets/awsaf49/brats2020-training-data
- **BraTS 2024**: Base do desafio BraTS edição 2024 amplamente utilizada na literatura, pode ser encontrada no link: 
https://www.synapse.org/Synapse:syn53708249/wiki/626323
> **Observação:** Baixe os dados brutos, mantenha a organização por paciente/sequência/máscara, e a nomenclatura original para reprodutibilidade.

> **Equivalência dos nomes das sequências:** Entre as bases existem divergências entre os nomes das sequencias, para isso criamos um mapa de equivalência para melhor compreensão:
```python
SEQ_KEYS_2024 = ["t1n","t1c","t2w","t2f","seg"]
SEQ_KEYS_2020 = ["t1","t1ce","t2","flair","seg"]
```
## 3) Estrutura do projeto

```
.
├─ .gitignore
├─ Artigo.pdf
├─ Poster.pdf
├─ README.md
├─ environment_conda.yaml
├─ resultados.xlsx
├─ splits.pdf
├─ experimentos/
│  ├─ Dataset_viz_allslices_dataset_creation.ipynb
│  ├─ baseline2020_*.ipynb                  # 4 notebooks (um por sequência)
│  ├─ baseline2024_*.ipynb                  # 4 notebooks
│  ├─ crossdataset_20_24_*.ipynb            # 4 notebooks
│  ├─ crossdataset_24_20_*.ipynb            # 4 notebooks
│  ├─ mixdataset_full_2020_2024_*.ipynb     # 4 notebooks 
│  ├─ mixdataset2020_*.ipynb                # 4 notebooks 
│  ├─ mixdataset2024_*.ipynb                # 4 notebooks 
│  └─ modelos_treinados/                    # pesos (.pt) salvos por experimento

```
**Resumo do notebook de separação / visualizaçãodos e dos 28 notebooks de experimentos:**
- **Separação / viz** — 1 notebook (Separa todas as sequencias e todos os experimentos).  
- **Baseline** — 8 notebooks (4 seq. Treino e teste no 2020; 4 seq. Treino e teste no 2024).  
- **Crossdataset** — 8 notebooks (4 seq. Treino no 2020 e teste no 2024; 4 seq. Treino no 2024 e teste no 2020).  
- **Mixdataset** — 12 notebooks  (4 seq. Treino misto no 2020 + 2024 e testes no 2020 + 2024 (Full) e separados no 2020 e 2024.

Todos os experimentos tem nomes autoexplicativos indicando cenário e sequência com a estrutura: cenário - dataset - sequencia.  
Os pesos treinados ficam em `experimentos/modelos_treinados/`.

---
## 4) O que faz cada notebook
> **Observação:** Todos os notebooks estão rodados e todas as etapas estão demonstradas comprovando os resultados obtidos.
---

### 4.1) Notebook de **separação e visualização**
**Arquivo:** `Dataset_viz_allslices_dataset_creation.ipynb` 

**Parâmetros típicos para carregamento das bases:** 
```python
base_path_20 = r"C:\Users\dados\Downloads\path_brats2020"
base_path_24 = r"C:\Users\dados\Downloads\path_brats2024"

```

**Parâmetros típicos para a saída dos splits:** 
```python
VAR_CENARIO = r"D:\all_slices_dataset\split_a_ser_utilizado"
```
**Etapas:**
- **Carregamento** das bases BraTS 2020/2024.  
- **Visualização** de imagens (amostras por sequência) e verificação de shapes.  
- **Separação em splits** (70-15-15) por pacientes, gravando em pastas separadas por split e sequencia.  
- **Criação de metadata** (CSV) por split e cenário.  
- **Contagem de imagens** por split e por cenário de testes.  
- **Verificação de duplicatas** de pacientes dentro de outro Split evitando testar com imagens utilizadas no treino.

### 4.2) Notebooks de **treino**  
**Cenários:** Baseline (2020, 2024) e Mixdataset_full  
**Parâmetros típicos:**
```python
VAR_CENARIO = r"D:\all_slices_dataset\split_a_ser_utilizado"
SEQ = "sequencia"    
MASK = "mascara"
```
**Etapas:**
- **Carregamento** dos splits das imagens (treino, validação e teste)
- **CUDA**: conexão/verificação de GPU.  
- **Resumo** dos tamanhos dos splits e shapes (imagens e máscaras).  
- **Plots** de imagem + máscara + overlay.  
- **Checagem de paths** carregados.  
- **Modelo**: definição da **Res-UNet**.  
- **Métricas & Loss**: definição das métricas de avaliação e função LOSS.  
- **Configuração de treino** (épocas, LR e early-stopping).  
- **Loop de treino**: salvamento de pesos e logs de treino.  
- **Teste unitário** + plot de um exemplo.  
- **Exemplos visuais**: 2 melhores, 4 aleatórios, 2 piores.  
- **Análise por quartil** do tamanho do tumor (métricas por quartil).  
- **Avaliação completa** do teste cáuculo das métricas relevantes.

### 4.3) Notebooks de **inferência**  
**Cenários:** Crossdataset (2020→2024; 2024→2020) e Mixdataset (testes 2020 e 2024)  
**Parâmetros típicos:**
```python
VAR_CENARIO = r"D:\all_slices_dataset\split_a_ser_utilizado"
SEQ = "sequencia"       
MASK = "mascara"
```
**Etapas:**
- **Carregamento** do split das imagens (apenas teste)
- **CUDA**: conexão/verificação de GPU.  
- **Resumo** do split de teste e shapes (imagens e máscaras.  
- **Plots** de imagem + máscara + overlay.    
- **Checagem de paths** carregados.  
- **Modelo**: definição da **Res-UNet**. 
- **Métricas**: definição das métricas de avaliação.    
- **Carregamento de pesos** treinados.  
- **Teste unitário** + plot de um exemplo.  
- **Exemplos visuais** (2 melhores, 4 aleatórios, 2 piores).  
- **Análise por quartil** do tamanho do tumor (métricas por quartil).  
- **Avaliação completa** do teste cáuculo das métricas relevantes.

---
## 4.4) Variáveis-chave (paths e sequência)
Nos notebooks, as variaveis mudam conforme o experimento. As variáveis padrão são:
- **`VAR_CENARIO`**: `BASELINE_2020`, `BASELINE_2024`, `CROSS_2024_2020`, `CROSS_2020_2024`, `MIXDATASET_FULL`, `MIXDATASET_2020`, `MIXDATASET_2024` 
- **`SEQ`**: `"t1n"-"t1" | "t1c"-"t1ce" | "t2w"-"t2" | "t2f-"flair"`  
- **`MASK`**: `"seg"`  nome padrão para todos

---
## 5) Tabela de referência dos notebooks

| Cenário        | Treino em | Teste em | Sequências (4)                  | Nº notebooks |
|----------------|-----------|----------|---------------------------------|--------------|
| Baseline_2020  | 2020      | 2020     | T1n, T1c, T2w, T2f              | 4            |
| Baseline_2024  | 2024      | 2024     | T1n, T1c, T2w, T2f              | 4            |
| Cross_20→24    | 2020      | 2024     | T1n, T1c, T2w, T2f              | 4            |
| Cross_24→20    | 2024      | 2020     | T1n, T1c, T2w, T2f              | 4            |
| Mix_full       | 2020+2024 | 2020+2024| T1n, T1c, T2w, T2f              | 4            |
| Mix_2020       | 2020+2024 | 2020     | T1n, T1c, T2w, T2f              | 4            |
| Mix_2024       | 2020+2024 | 2024     | T1n, T1c, T2w, T2f              | 4            |

---
## 6) Modelos treinados
Os pesos (checkpoints) são salvos em `experimentos/modelos_treinados/`, seguindo a convenção de nome usada nos notebooks com um best na frente.

Exemplo: best_BASELINE2020_T1c_T1ce.pt

---
## 7) Tabela de resultados e imagem ilustrativa dos splits
O arquivo resultado.xlsx contém uma síntese de todos os resultados das métricas globais coletados nos experimentos.

O arquivo splits.pdf apresenta uma ilustração explicativa dos cenários testados no projeto.

---
## 8) Artigo e Poster de apresentação da pesquisa
O arquivo Artigo.pdf contém as etapas de desenvolvimento e as conclusões do trabalho, apresentadas de forma acadêmica e detalhada.

O arquivo Poster.pdf apresenta os principais pontos do projeto de maneira resumida e de rápida compreensão.

---
## 9) Reprodutibilidade
-  Configurar o ambiente com o arquivo .yaml
-  Baixar as bases BraTS 2020/2024.  
-  Ajustar variáveis de Path nos notebooks.  
-  Executar `Dataset_viz_allslices_dataset_creation.ipynb` para criar splits/metadata.  
-  Rodar os notebooks de treino para gerar os pesos e ja testar os cenarios respectivos.  
-  Rodar os notebooks de inferencia que usam os pesos dos notebooks de treino para testar os cenarios.
