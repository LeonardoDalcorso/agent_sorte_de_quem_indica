# Campanha Sorte de Quem Indica
## Sistema de Análise de Cupom Fiscal

---

## 📊 Resultados até 18/11/2025

- **Total de Cupons Analisados:** 132.354
- **Valor em Produtos SpecialDog:** R$ 14.936.852,30

---

## 🎯 REGRAS DE PONTUAÇÃO DA CAMPANHA

### Tabela de Multiplicadores

| Linha do Produto | Multiplicador | Exemplo (10 unidades) |
|------------------|---------------|----------------------|
| 🔴 **PREMIUM** (Padrão) | **1x** | 10 × 1 = **10 pontos** |
| 🟡 **ULTRALIFE** | **2x** | 10 × 2 = **20 pontos** |
| 🟢 **BIONATURAL** | **3x** | 10 × 3 = **30 pontos** |
| 🔵 **BIONATURAL SENSITIVE** | **4x** | 10 × 4 = **40 pontos** |

**Fórmula:** `PONTOS = QUANTIDADE × MULTIPLICADOR`

---

## 🏗️ Stack Tecnológica

| Tecnologia | Finalidade |
|------------|-----------|
| **ASP.NET** | Backend e API principal |
| **OpenCvSharp** | Processamento e tratamento de imagens |
| **Google Cloud Vision OCR** | OCR primário (prioridade 1) |
| **AWS OCR** | OCR secundário (fallback) |
| **Google Document AI OCR** | OCR terciário (fallback) |
| **OpenAI API Assistant** | Conversão para JSON e validação de produtos |
| **Hangfire** | Orquestração dos robôs e agentes |

---

## 🔄 Fluxo Geral do Sistema

```mermaid
graph TB
    A[📱 Usuário envia foto do cupom] --> B[🎯 Hangfire - Agente Orquestrador]
    B --> C[📦 Distribui para 10 Agentes de Processamento]
    C --> D[🖼️ Pré-processamento da Imagem]
    
    D --> E[📝 Tentativa 1: Google Cloud Vision]
    E -->|✅ Sucesso| F[🤖 OpenAI Assistant - Conversão JSON]
    E -->|❌ Erro OCR ou<br/>Validação Falhou| G[📝 Tentativa 2: AWS OCR]
    
    G -->|✅ Sucesso| F
    G -->|❌ Erro OCR ou<br/>Validação Falhou| H[📝 Tentativa 3: Document AI]
    
    H -->|✅ Sucesso| F
    H -->|❌ Falha Total| Z[⚠️ Cupom Não Processado]
    
    F --> I[✅ Validação de Dados<br/>CNPJ + Chave + Produtos]
    I -->|❌ Produtos Inválidos ou<br/>Dados Incorretos| G
    I -->|✅ Validação OK| J[💰 Cálculo de Pontuação]
    
    J --> K[📊 Lançamento no Sistema de Sorteio]
    
    style A fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
    style B fill:#fff3cd,stroke:#856404,stroke-width:2px
    style K fill:#d4edda,stroke:#155724,stroke-width:2px
    style E fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style G fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style H fill:#dc3545,stroke:#721c24,stroke-width:3px,color:#fff
    style Z fill:#f8d7da,stroke:#721c24,stroke-width:2px
```

### 🔄 Estratégia de Retry com 3 OCRs

**IMPORTANTE:** O sistema tenta os 3 OCRs em sequência quando encontra:
- ❌ Erro na leitura do OCR
- ❌ Texto extraído incompleto ou ilegível
- ❌ Produtos não identificados
- ❌ CNPJ ou Chave inválidos
- ❌ Falha na conversão JSON
- ❌ Validação de dados falhou

**Fluxo de Retry:**
```
1ª Tentativa → Google Cloud Vision
    ↓ (erro em qualquer etapa)
2ª Tentativa → AWS OCR (processa do zero)
    ↓ (erro em qualquer etapa)
3ª Tentativa → Document AI (processa do zero)
    ↓ (erro em qualquer etapa)
❌ Cupom marcado como não processado
```

---

## 🎯 Arquitetura de Agentes

```mermaid
graph TB
    A[Hangfire - Orquestrador Principal] --> B[Agente 1]
    A --> C[Agente 2]
    A --> D[Agente 3]
    A --> E[Agente 4]
    A --> F[Agente 5]
    A --> G[Agente 6]
    A --> H[Agente 7]
    A --> I[Agente 8]
    A --> J[Agente 9]
    A --> K[Agente 10]
    
    B --> L[Processamento Paralelo]
    C --> L
    D --> L
    E --> L
    F --> L
    G --> L
    H --> L
    I --> L
    J --> L
    K --> L
    
    style A fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style B fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style C fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style D fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style E fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style F fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style G fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style H fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style I fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style J fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style K fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style L fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
```

**Função dos Agentes:**
- ⚡ **10 agentes** trabalhando em paralelo
- 📦 **Cada agente** processa cupons de forma independente
- 🔄 **Distribuição automática** de carga pelo Hangfire
- 📊 **Processamento escalável** - pode aumentar número de agentes conforme demanda

---

## 🖼️ Etapa 1: Pré-processamento de Imagem

```mermaid
graph TB
    A[📸 Foto Original do Cupom] --> B{Análise de Qualidade}
    B --> C[🗜️ Compressão da Imagem<br/>OpenCvSharp]
    C --> D{Necessita<br/>Inversão de Cores?}
    D -->|Sim| E[🔄 Inversão de Cores<br/>para facilitar OCR]
    D -->|Não| F[✅ Imagem Pronta]
    E --> F
    F --> G[📤 Envio para OCR]
    
    style A fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
    style F fill:#d4edda,stroke:#155724,stroke-width:2px
    style G fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
```

**Objetivos do Pré-processamento:**
- 🗜️ Reduzir tamanho do arquivo mantendo qualidade
- 🔍 Melhorar contraste para leitura do OCR
- 💰 Otimizar custos de processamento
- ⚡ Aumentar taxa de sucesso do OCR

---

## 📝 Etapa 2: OCR em Cascata com Retry Completo

```mermaid
graph TB
    A[🖼️ Imagem Processada] --> B[📝 TENTATIVA 1<br/>Google Cloud Vision<br/>💰 Custo Baixo]
    
    B -->|✅ Texto OK| C[🤖 OpenAI - Conversão JSON]
    B -->|❌ Erro ou texto ruim| D[📝 TENTATIVA 2<br/>AWS OCR<br/>💰💰 Custo Médio]
    
    C -->|✅ JSON OK| E[✅ Validação CNPJ/Chave/Produtos]
    C -->|❌ Falha conversão| D
    
    D -->|✅ Texto OK| F[🤖 OpenAI - Conversão JSON]
    D -->|❌ Erro ou texto ruim| G[📝 TENTATIVA 3<br/>Google Document AI<br/>💰💰💰 Custo Alto]
    
    F -->|✅ JSON OK| E
    F -->|❌ Falha conversão| G
    
    E -->|❌ Produtos Inválidos<br/>ou Dados Incorretos| D
    E -->|✅ Tudo OK| H[✅ Prosseguir para<br/>Cálculo de Pontos]
    
    G -->|✅ Texto OK| I[🤖 OpenAI - Conversão JSON]
    G -->|❌ Erro| J[⚠️ FALHA TOTAL<br/>Cupom Não Processado]
    
    I -->|✅ JSON OK| E
    I -->|❌ Falha conversão| J
    
    style B fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style D fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style G fill:#dc3545,stroke:#721c24,stroke-width:3px,color:#fff
    style H fill:#d4edda,stroke:#155724,stroke-width:2px
    style J fill:#f8d7da,stroke:#721c24,stroke-width:2px
```

### 🔄 Sistema de Retry Inteligente

**O sistema SEMPRE tenta os 3 OCRs quando encontra:**

| Tipo de Erro | Causa o Retry? | Próximo OCR |
|--------------|----------------|-------------|
| ❌ OCR não conseguiu ler | ✅ Sim | Próximo da fila |
| ❌ Texto extraído ilegível | ✅ Sim | Próximo da fila |
| ❌ OpenAI falhou converter JSON | ✅ Sim | Próximo da fila |
| ❌ CNPJ inválido no JSON | ✅ Sim | Próximo da fila |
| ❌ Chave NF-e inválida | ✅ Sim | Próximo da fila |
| ❌ Nenhum produto válido encontrado | ✅ Sim | Próximo da fila |
| ❌ Produtos não reconhecidos | ✅ Sim | Próximo da fila |
| ❌ JSON malformado | ✅ Sim | Próximo da fila |

### 📊 Estratégia de Fallback

| Tentativa | OCR | Custo | Quando Usa | Taxa Sucesso Final |
|-----------|-----|-------|------------|-------------------|
| **1ª** | Google Cloud Vision | 💰 Baixo | Sempre | ~85% |
| **2ª** | AWS OCR | 💰💰 Médio | Se 1ª falhar em qualquer etapa | ~12% |
| **3ª** | Google Document AI | 💰💰💰 Alto | Se 2ª falhar em qualquer etapa | ~3% |

**Taxa de Sucesso Acumulada:** ~98% (após 3 tentativas)

### 💡 Exemplo de Fluxo Real

```
┌─────────────────────────────────────────────────────────┐
│ CUPOM #12345 - Tentando processar                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ 1ª TENTATIVA: Google Cloud Vision                      │
│   ├─ OCR extraiu texto: ✅ OK                          │
│   ├─ OpenAI converteu JSON: ✅ OK                      │
│   └─ Validação CNPJ: ❌ CNPJ inválido detectado       │
│                                                         │
│ 🔄 RETRYING COM 2º OCR...                              │
│                                                         │
│ 2ª TENTATIVA: AWS OCR                                  │
│   ├─ OCR extraiu texto: ✅ OK (melhor qualidade)      │
│   ├─ OpenAI converteu JSON: ✅ OK                      │
│   ├─ Validação CNPJ: ✅ OK                             │
│   ├─ Validação Chave: ✅ OK                            │
│   └─ Validação Produtos: ✅ 3 produtos elegíveis      │
│                                                         │
│ ✅ CUPOM APROVADO - 28 pontos gerados                  │
└─────────────────────────────────────────────────────────┘
```

**Vantagens de Cada OCR:**
- **Google Cloud Vision:** Rápido, econômico, ótimo para cupons limpos
- **AWS:** Melhor com textos complexos, cupons amassados, múltiplas fontes
- **Document AI:** Máxima precisão, entende layouts complexos, reconhece tabelas e estruturas

---

## 🤖 Etapa 3: Conversão para JSON com OpenAI

```mermaid
graph TB
    A[📄 Texto do OCR] --> B[🤖 OpenAI Assistant]
    B --> C[Conversão para Padrão JSON]
    C --> D[📋 Estrutura de Dados]
    D --> E[CNPJ do Estabelecimento]
    D --> F[Chave da Nota Fiscal]
    D --> G[Lista de Produtos]
    D --> H[Valores e Quantidades]
    D --> I[Pontos Gerados]
    
    E --> J[✅ Dados Estruturados]
    F --> J
    G --> J
    H --> J
    I --> J
    
    style B fill:#d4edda,stroke:#155724,stroke-width:2px
    style J fill:#d4edda,stroke:#155724,stroke-width:2px
```

### 📋 Estrutura JSON Gerada

```json
{
  "Cnpj": "XXXXXXXX",
  "Chave": "3525110330XXX00011765001XXXXXX1000261605",
  "Url": "https://www.nfce.fazenda.sp.gov.br/consulta?p=...",
  "DataEmissao": "2025-11-08T11:40:18",
  "ValorTotalCupom": 141.7,
  "Produtos": [
    {
      "Nome": "SACHE DOG CHOW FL. CARNE 100GR",
      "Quantidade": 4.0,
      "ValorUnitario": 3.5,
      "ValorTotal": 14.0,
      "DescontoUnitario": 0.0,
      "DescontoTotal": 0.0,
      "ValorUnitarioMenosDesconto": 3.5,
      "ValorTotalMenosDesconto": 14.0,
      "PontosGerados": 4.0,
      "Duvida": false,
      "ProdutoSpecialDog": false
    },
    {
      "Nome": "SPECIAL DOG ULTRALIFE CORDEIRO 100GR",
      "Quantidade": 4.0,
      "ValorUnitario": 3.2,
      "ValorTotal": 12.8,
      "DescontoUnitario": 0.0,
      "DescontoTotal": 0.0,
      "ValorUnitarioMenosDesconto": 3.2,
      "ValorTotalMenosDesconto": 12.8,
      "PontosGerados": 8.0,
      "Duvida": false,
      "ProdutoSpecialDog": true
    }
  ],
  "TextoCompleto": "TEXTO COMPLETO EXTRAÍDO DO OCR..."
}
```

### 📋 Detalhamento dos Campos JSON

#### **Campos do Cupom**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| `Cnpj` | string | CNPJ do estabelecimento (apenas números) |
| `Chave` | string | Chave de acesso da NFC-e (44 dígitos) |
| `Url` | string | URL para consulta da nota fiscal |
| `DataEmissao` | datetime | Data e hora de emissão do cupom |
| `ValorTotalCupom` | decimal | Valor total do cupom fiscal |
| `Produtos` | array | Lista de produtos identificados no cupom |
| `TextoCompleto` | string | Texto bruto extraído pelo OCR |

#### **Campos de Cada Produto**
| Campo | Tipo | Descrição |
|-------|------|-----------|
| `Nome` | string | Descrição do produto conforme cupom |
| `Quantidade` | decimal | Quantidade comprada |
| `ValorUnitario` | decimal | Valor unitário original |
| `ValorTotal` | decimal | Valor total do item (Qtd × Valor Unit.) |
| `DescontoUnitario` | decimal | Desconto aplicado por unidade |
| `DescontoTotal` | decimal | Desconto total no item |
| `ValorUnitarioMenosDesconto` | decimal | Valor unitário após desconto |
| `ValorTotalMenosDesconto` | decimal | Valor total após desconto |
| `PontosGerados` | decimal | **Pontos calculados para o produto** |
| `Duvida` | boolean | Flag indicando se há dúvida na identificação |
| `ProdutoSpecialDog` | boolean | **Indica se é produto elegível SpecialDog** |

---

## 🔍 Processamento Detalhado do JSON

```mermaid
graph TB
    A[📄 Texto do OCR] --> B[🤖 OpenAI Assistant]
    B --> C[📦 JSON Estruturado]
    
    C --> D[Análise de Cada Produto]
    
    D --> E[🔍 Identificação do Produto]
    E --> F{É SpecialDog?}
    
    F -->|✅ Sim| G[ProdutoSpecialDog = true]
    F -->|❌ Não| H[ProdutoSpecialDog = false]
    
    G --> I[💰 Cálculo de Pontos]
    H --> J[PontosGerados = 0]
    
    I --> K{Identificar Linha}
    K -->|Premium| L[Multiplicador 1x]
    K -->|UltraLife| M[Multiplicador 2x]
    K -->|BioNatural| N[Multiplicador 3x]
    K -->|BioNat Sensitive| O[Multiplicador 4x]
    
    L --> P[📊 PontosGerados<br/>Quantidade × Multiplicador]
    M --> P
    N --> P
    O --> P
    J --> P
    
    P --> Q[✅ Produto Processado]
    
    style G fill:#d4edda,stroke:#155724,stroke-width:2px
    style H fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style P fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
```

### 🎯 Regras de Negócio Aplicadas

#### **1. Identificação de Produtos SpecialDog**

```mermaid
graph LR
    A[Nome do Produto] --> B{Contém<br/>SPECIAL DOG<br/>ou DOG CHOW?}
    B -->|✅ Sim| C[SpecialDog = true]
    B -->|❌ Não| D[SpecialDog = false]
    
    C --> E{Identificar Linha}
    E -->|ULTRALIFE| F[2x]
    E -->|BIONATURAL<br/>SENSITIVE| G[4x]
    E -->|BIONATURAL| H[3x]
    E -->|Outros| I[1x]
    
    style C fill:#d4edda,stroke:#155724,stroke-width:2px
    style D fill:#f8d7da,stroke:#721c24,stroke-width:2px
```

#### **2. Cálculo de Pontuação por Linha de Produto**

**Tabela de Identificação:**

| Palavras-chave no Nome | Linha | Multiplicador | Pontos p/ 10 unid. |
|------------------------|-------|---------------|--------------------|
| "PREMIUM", "SACHÊ", "CARNE" | Premium | 1x | 10 |
| "ULTRALIFE" | UltraLife | 2x | 20 |
| "BIONATURAL" (sem "SENSITIVE") | BioNatural | 3x | 30 |
| "BIONATURAL" + "SENSITIVE" | BioNat. Sensitive | 4x | 40 |
| Nenhuma das acima | Não elegível | 0x | 0 |

**Exemplos de Cálculo Real:**

| Exemplo | Produto | Qtd | Linha | Mult. | Cálculo | Pontos |
|---------|---------|-----|-------|-------|---------|--------|
| 1 | SPECIAL DOG SACHÊ CARNE 100G | 5 | Premium | 1x | 5 × 1 | **5** |
| 2 | SPECIAL DOG ULTRALIFE CORDEIRO | 5 | UltraLife | 2x | 5 × 2 | **10** |
| 3 | SPECIAL DOG BIONATURAL ADULTO | 5 | BioNatural | 3x | 5 × 3 | **15** |
| 4 | SPECIAL DOG BIONATURAL SENSITIVE | 5 | BioNat. Sensit. | 4x | 5 × 4 | **20** |

#### **3. Tratamento de Descontos**

```mermaid
graph TB
    A[Valor Unitário] --> B{Tem Desconto?}
    B -->|✅ Sim| C[Calcula Desconto]
    B -->|❌ Não| D[ValorMenosDesconto<br/>= ValorOriginal]
    
    C --> E[DescontoUnitario]
    C --> F[DescontoTotal]
    E --> G[ValorUnitarioMenosDesconto]
    F --> H[ValorTotalMenosDesconto]
    
    G --> I[💰 Base para Pontuação]
    H --> I
    D --> I
    
    style I fill:#fff3cd,stroke:#856404,stroke-width:2px
```

**IMPORTANTE:** Os pontos são calculados sempre sobre o valor APÓS descontos.

#### **4. Flag de Dúvida**

O campo `Duvida` é marcado como `true` quando:
- Nome do produto está incompleto ou truncado
- OCR teve baixa confiança na leitura
- Produto com nome ambíguo
- Necessita validação manual

```json
{
  "Nome": "SPECIAL DOG S...",
  "Duvida": true,
  "ProdutoSpecialDog": true
}
```

---

## 📊 Análise de Exemplo Real

### Cupom Processado

```json
{
  "Cnpj": "03302910000117",
  "Chave": "35251103302910000117650010000261591000261605",
  "DataEmissao": "2025-11-08T11:40:18",
  "ValorTotalCupom": 141.70
}
```

### Breakdown de Pontos (Exemplo Real)

| Produto | Qtd | Valor | Linha | Mult. | Pontos |
|---------|-----|-------|-------|-------|--------|
| SACHE DOG CHOW FL. CARNE | 4 | 14.00 | Premium | 1x | 4 |
| SACHE DOG CHOW AD. CARNE | 8 | 28.00 | Premium | 1x | 8 |
| SACHE DOG CHOW FL. CARNE | 8 | 28.00 | Premium | 1x | 8 |
| SPECIAL DOG ULTRALIFE CORDEIRO | 4 | 12.80 | UltraLife | 2x | 8 |
| AREIA PIPICAT FLORAL | 1 | 58.90 | N/A | 0x | 0 |
| **TOTAL** | **25** | **141.70** | - | - | **28** |

**Resumo da Pontuação:**
- ✅ 4 produtos SpecialDog elegíveis
- ❌ 1 produto não elegível (areia para gatos)
- 💰 **28 pontos** gerados no total
- 📊 58.5% do valor do cupom em produtos SpecialDog (R$ 82,80)

**Detalhamento por Linha:**
- 🔵 **Premium (1x):** 20 unidades = 20 pontos
- 🟢 **UltraLife (2x):** 4 unidades × 2 = 8 pontos
- ⚪ **Não elegível:** 0 pontos

---

## ✅ Etapa 4: Validação de Dados

```mermaid
graph TB
    A[📦 JSON Recebido] --> B{CNPJ<br/>Válido?}
    B -->|❌ Não| Z1[❌ Rejeitar: CNPJ Inválido]
    B -->|✅ Sim| C{Chave NF-e<br/>Válida?}
    C -->|❌ Não| Z2[❌ Rejeitar: Chave Inválida]
    C -->|✅ Sim| D{Data de Emissão<br/>no Período?}
    D -->|❌ Não| Z3[❌ Rejeitar: Fora do Período]
    D -->|✅ Sim| E[🔍 Validar Produtos]
    
    E --> F[Para cada produto no array]
    F --> G{ProdutoSpecialDog<br/>= true?}
    G -->|❌ Não| H[Ignorar produto]
    G -->|✅ Sim| I[🗄️ Buscar em Base<br/>de Descritivos]
    
    I --> J{Encontrado<br/>Exato?}
    J -->|✅ Sim| K[✅ Produto Validado]
    J -->|❌ Não| L[🔍 Fuzzy Match<br/>Similaridade]
    
    L --> M{Similaridade<br/>&gt; 80%?}
    M -->|✅ Sim| K
    M -->|❌ Não| N{Duvida<br/>= true?}
    N -->|✅ Sim| O[⏳ Análise Manual]
    N -->|❌ Não| P[❌ Produto Rejeitado]
    
    K --> Q[💰 Validar Pontos]
    Q --> R[✅ Cupom Aprovado]
    
    style Z1 fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style Z2 fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style Z3 fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style R fill:#d4edda,stroke:#155724,stroke-width:2px
    style O fill:#fff3cd,stroke:#856404,stroke-width:2px
```

### 🗄️ Base de Descritivos de Produtos

**Exemplos de Variações Cadastradas**

Produto Padrão: **SPECIAL DOG SACHÊ CARNE 100G**

| Variação Encontrada no Cupom | Logista | Status |
|------------------------------|---------|--------|
| SACHE SPECIAL DOG CARNE 100GR | Rede A | ✅ Mapeado |
| SP DOG SACHE CARNE 100G | Rede B | ✅ Mapeado |
| SPECIAL DOG ADULTO CARNE SACHE | Rede C | ✅ Mapeado |
| SPL DOG SACHE CRN 100 | Rede D | ✅ Mapeado |
| SPECIALDOG SACHET BEEF 100G | Rede E | ✅ Mapeado |

**Processo de Cadastro:**
1. 📝 Equipe identifica novas variações de nome
2. ✍️ Cadastro manual no sistema
3. 🔗 Vinculação ao produto padrão
4. ✅ Validação e ativação

### Algoritmo de Fuzzy Match

```python
def validar_produto(nome_cupom, base_descritivos):
    # 1. Normalização do texto
    nome_normalizado = normalize(nome_cupom)
    # Remove acentos, converte para maiúsculas, 
    # remove caracteres especiais
    
    # 2. Busca exata
    if nome_normalizado in base_descritivos:
        return True, 100  # Match exato
    
    # 3. Fuzzy matching (Levenshtein Distance)
    for descritivo in base_descritivos:
        similaridade = calcular_similaridade(
            nome_normalizado, 
            descritivo
        )
        
        if similaridade >= 80:  # Threshold de 80%
            return True, similaridade
    
    # 4. Não encontrado
    return False, 0
```

**Técnicas Utilizadas:**
- **Levenshtein Distance:** Mede diferença entre strings
- **Jaro-Winkler:** Otimizado para nomes curtos
- **Token Sort Ratio:** Ignora ordem das palavras

**Exemplo de Similaridade:**

| Texto Cupom | Produto Base | Similaridade | Match? |
|-------------|--------------|--------------|--------|
| "SACHE SPECIAL DOG" | "SPECIAL DOG SACHÊ" | 92% | ✅ |
| "SP DOG CARNE" | "SPECIAL DOG CARNE" | 87% | ✅ |
| "SPECIAL CAT" | "SPECIAL DOG" | 45% | ❌ |
| "SPECIALDOG ADULTO" | "SPECIAL DOG ADULTO" | 95% | ✅ |

---

## 💰 Etapa 5: Cálculo de Pontuação

```mermaid
graph TB
    A[✅ Produtos Validados] --> B[🤖 Primeira Validação<br/>OpenAI Assistant]
    B --> C[💯 Pontos Calculados<br/>por Tipo de Produto]
    C --> D[🔍 Segunda Validação<br/>Lógica Interna]
    D -->|✅ Confirmado| E[✅ Pontuação Final]
    D -->|❌ Divergência| F[⚠️ Recálculo Manual]
    E --> G[📊 Lançamento no Sistema]
    F --> G
    
    style E fill:#d4edda,stroke:#155724,stroke-width:2px
    style F fill:#fff3cd,stroke:#856404,stroke-width:2px
```

### 🔄 Processo de Validação Dupla

**Otimização de Custos Implementada:**

| Versão | Método | Custo Tokens | Status |
|--------|--------|--------------|--------|
| **1.0** (Antiga) | OpenAI calcula tudo | 💰💰💰 Alto | ❌ Descontinuada |
| **2.0** (Atual) | OpenAI + Lógica | 💰 Médio | ✅ Em Produção |

**Fluxo Atual:**
1. 🤖 **OpenAI Assistant** faz cálculo inicial baseado nas regras
2. ✅ **Lógica Interna** valida o cálculo da IA
3. ⚖️ Se houver divergência → Recálculo ou análise manual
4. 💰 **Economia:** ~60% de redução no consumo de tokens

---

## 📊 Etapa 6: Lançamento Final

```mermaid
graph TB
    A[💰 Pontuação Calculada] --> B[📝 Status do Cupom]
    B --> C{Status}
    C -->|✅ Aprovado| D[✔️ APROVADO]
    C -->|⏳ Pendente| E[⏳ PENDENTE DE ANÁLISE]
    C -->|❌ Rejeitado| F[❌ REJEITADO]
    
    D --> G[📤 Envio para Sistema<br/>de Gerenciamento de Sorteio]
    E --> G
    F --> G
    
    G --> H[🎯 Empresa Gestora do Sorteio]
    H --> I[📊 Contabilização Final]
    
    style D fill:#d4edda,stroke:#155724,stroke-width:2px
    style E fill:#fff3cd,stroke:#856404,stroke-width:2px
    style F fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style I fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
```

**Possíveis Status:**
- ✅ **APROVADO:** Cupom validado, produtos elegíveis, pontos calculados
- ⏳ **PENDENTE:** Aguardando análise manual (flag `Duvida = true`)
- ❌ **REJEITADO:** CNPJ inválido, chave inválida, sem produtos elegíveis, fora do período

---

## 🔁 Fluxo Completo Integrado com Sistema de Retry

```mermaid
graph TB
    subgraph "📱 ENTRADA"
        A[Usuário envia foto]
    end
    
    subgraph "🎯 ORQUESTRAÇÃO"
        B[Hangfire - Orquestrador]
        C[10 Agentes Paralelos]
    end
    
    subgraph "🖼️ PROCESSAMENTO"
        D[Compressão OpenCvSharp]
        E[Inversão de Cores]
    end
    
    subgraph "📝 OCR - TENTATIVAS SEQUENCIAIS"
        F[1ª Google Cloud Vision<br/>💰 Baixo custo]
        G[2ª AWS OCR<br/>💰💰 Médio custo]
        H[3ª Document AI<br/>💰💰💰 Alto custo]
    end
    
    subgraph "🤖 INTELIGÊNCIA ARTIFICIAL"
        I[OpenAI Assistant<br/>Conversão JSON]
        J[OpenAI Assistant<br/>Cálculo Pontos]
    end
    
    subgraph "✅ VALIDAÇÃO COM RETRY"
        K[CNPJ + Chave]
        L[Produtos Elegíveis]
        M[Pontuação Final]
    end
    
    subgraph "📊 SAÍDA"
        N[Lançamento]
        O[Sistema de Sorteio]
    end
    
    A --> B --> C --> D --> E --> F
    
    F -->|✅ OK| I
    F -->|❌ Erro em<br/>QUALQUER etapa| G
    
    G -->|✅ OK| I
    G -->|❌ Erro em<br/>QUALQUER etapa| H
    
    H -->|✅ OK| I
    H -->|❌ Falha Total| Z[⚠️ Não Processado]
    
    I -->|✅ JSON OK| K
    I -->|❌ Falha| G
    
    K -->|✅ OK| L
    K -->|❌ Inválido| G
    
    L -->|✅ Produtos OK| J
    L -->|❌ Nenhum válido| G
    
    J --> M --> N --> O
    
    style A fill:#d1ecf1,stroke:#0c5460,stroke-width:2px
    style B fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style F fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
    style G fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style H fill:#dc3545,stroke:#721c24,stroke-width:3px,color:#fff
    style I fill:#d4edda,stroke:#155724,stroke-width:2px
    style O fill:#e2d9f3,stroke:#6f42c1,stroke-width:2px
    style Z fill:#f8d7da,stroke:#721c24,stroke-width:2px
```

### 🔄 Pontos de Retry no Sistema

O sistema tenta os **3 OCRs em sequência** quando detecta erro em:

1. **Leitura do OCR**
   - Texto ilegível ou incompleto
   - Imagem de baixa qualidade
   - Caracteres não reconhecidos

2. **Conversão JSON (OpenAI)**
   - Falha ao estruturar dados
   - JSON malformado
   - Campos essenciais ausentes

3. **Validação de CNPJ/Chave**
   - CNPJ com dígitos incorretos
   - Chave NF-e inválida
   - Formato incorreto

4. **Validação de Produtos**
   - Nenhum produto SpecialDog encontrado
   - Produtos não reconhecidos
   - Descrições muito curtas/truncadas

5. **Cálculo de Pontos**
   - Valores inconsistentes
   - Multiplicadores incorretos
   - Quantidades inválidas

### 📊 Taxa de Sucesso por Tentativa

```mermaid
graph LR
    A[100% Cupons] -->|85%| B[✅ Sucesso na 1ª]
    A -->|15%| C[Tentam 2ª]
    C -->|80%| D[✅ Sucesso na 2ª]
    C -->|20%| E[Tentam 3ª]
    E -->|90%| F[✅ Sucesso na 3ª]
    E -->|10%| G[❌ Falha Total]
    
    style B fill:#d4edda,stroke:#155724,stroke-width:2px
    style D fill:#d4edda,stroke:#155724,stroke-width:2px
    style F fill:#d4edda,stroke:#155724,stroke-width:2px
    style G fill:#f8d7da,stroke:#721c24,stroke-width:2px
```

**Resultado Final:** ~98% de taxa de sucesso após as 3 tentativas

---

## 🔄 Sistema de Retry Inteligente

### Como Funciona o Retry

```mermaid
graph TB
    A[Cupom Recebido] --> B[Processar com OCR 1]
    B --> C{Sucesso em<br/>TODAS etapas?}
    C -->|✅ Sim| D[✅ Cupom Aprovado]
    C -->|❌ Não| E{Já tentou<br/>OCR 2?}
    
    E -->|Não| F[Processar com OCR 2<br/>DO ZERO]
    E -->|Sim| G{Já tentou<br/>OCR 3?}
    
    F --> H{Sucesso em<br/>TODAS etapas?}
    H -->|✅ Sim| D
    H -->|❌ Não| G
    
    G -->|Não| I[Processar com OCR 3<br/>DO ZERO]
    G -->|Sim| J[❌ Cupom Não Processado]
    
    I --> K{Sucesso em<br/>TODAS etapas?}
    K -->|✅ Sim| D
    K -->|❌ Não| J
    
    style D fill:#d4edda,stroke:#155724,stroke-width:2px
    style J fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style F fill:#ffc107,stroke:#856404,stroke-width:3px,color:#000
    style I fill:#dc3545,stroke:#721c24,stroke-width:3px,color:#fff
```

### 📋 Checklist de Validação (Cada Tentativa)

Para um cupom ser considerado **SUCESSO**, ele precisa passar por TODAS as etapas:

| # | Etapa | Validação | Se Falhar |
|---|-------|-----------|-----------|
| 1 | OCR | Texto legível extraído | ❌ Tenta próximo OCR |
| 2 | OpenAI | JSON estruturado criado | ❌ Tenta próximo OCR |
| 3 | CNPJ | Formato válido e dígitos OK | ❌ Tenta próximo OCR |
| 4 | Chave NF-e | 44 dígitos válidos | ❌ Tenta próximo OCR |
| 5 | Data | Dentro do período da campanha | ❌ Tenta próximo OCR |
| 6 | Produtos | Pelo menos 1 produto elegível | ❌ Tenta próximo OCR |
| 7 | Pontos | Cálculo validado | ❌ Tenta próximo OCR |

**Se QUALQUER uma falhar** → Sistema tenta o próximo OCR

### 💡 Exemplo Real de Retry

```
┌──────────────────────────────────────────────────────────────┐
│ CUPOM #45678 - Foto com baixa qualidade                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ 🔵 TENTATIVA 1: Google Cloud Vision                         │
│    ├─ OCR: ✅ Extraiu texto                                 │
│    ├─ OpenAI: ✅ JSON criado                                │
│    ├─ CNPJ: ✅ 12345678000190                               │
│    ├─ Chave: ❌ Apenas 40 dígitos (faltam 4)               │
│    └─ 🔄 RETRYING...                                        │
│                                                              │
│ 🟡 TENTATIVA 2: AWS OCR                                     │
│    ├─ OCR: ✅ Extraiu texto (melhor qualidade)             │
│    ├─ OpenAI: ✅ JSON criado                                │
│    ├─ CNPJ: ✅ 12345678000190                               │
│    ├─ Chave: ✅ 12345678901234567890123456789012345678901234│
│    ├─ Produtos: ❌ Nenhum produto elegível encontrado      │
│    └─ 🔄 RETRYING...                                        │
│                                                              │
│ 🔴 TENTATIVA 3: Google Document AI                          │
│    ├─ OCR: ✅ Extraiu texto (máxima qualidade)             │
│    ├─ OpenAI: ✅ JSON criado                                │
│    ├─ CNPJ: ✅ 12345678000190                               │
│    ├─ Chave: ✅ 12345678901234567890123456789012345678901234│
│    ├─ Produtos: ✅ 3 produtos SpecialDog encontrados       │
│    ├─ Pontos: ✅ 42 pontos calculados                       │
│    └─ ✅ CUPOM APROVADO!                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 🎯 Por Que Cada OCR Pode Ter Resultado Diferente?

```mermaid
graph TB
    A[Mesma Imagem] --> B[OCR 1: Google Cloud Vision]
    A --> C[OCR 2: AWS OCR]
    A --> D[OCR 3: Document AI]
    
    B --> E[Texto: SP3C1AL D0G<br/>❌ Confundiu caracteres]
    C --> F[Texto: SPECIAL DG<br/>❌ Faltam letras]
    D --> G[Texto: SPECIAL DOG<br/>✅ Perfeito!]
    
    style E fill:#f8d7da,stroke:#721c24,stroke-width:2px
    style F fill:#fff3cd,stroke:#856404,stroke-width:2px
    style G fill:#d4edda,stroke:#155724,stroke-width:2px
```

**Motivos das diferenças:**
- 🔍 **Algoritmos diferentes** de reconhecimento
- 📊 **Modelos de IA** treinados com datasets distintos
- 🎯 **Especialização:** cada OCR é melhor em cenários específicos
- 💰 **Custo vs Qualidade:** os mais caros têm melhor precisão

### 📊 Estatísticas do Sistema de Retry

| Métrica | Valor |
|---------|-------|
| Cupons que passam na 1ª tentativa | 85% |
| Cupons que passam na 2ª tentativa | 12% |
| Cupons que passam na 3ª tentativa | 3% |
| **Taxa de sucesso total** | **~98%** |
| Taxa de falha total | ~2% |

### ⚙️ Configurações do Sistema

```json
{
  "retry_config": {
    "max_tentativas": 3,
    "timeout_por_ocr": "30s",
    "backoff_strategy": "nenhum (sequencial)",
    "ocr_sequence": [
      "GoogleCloudVision",
      "AWS_Textract", 
      "GoogleDocumentAI"
    ],
    "validacoes_obrigatorias": [
      "cnpj_valido",
      "chave_nfe_valida",
      "data_no_periodo",
      "pelo_menos_um_produto_elegivel"
    ]
  }
}
```

---

## 🎯 Diferenciais do Sistema

### ⚡ Performance
- **10 agentes paralelos** processando simultaneamente
- **Orquestração com Hangfire** para distribuição eficiente
- Processamento médio de milhares de cupons por dia
- **Tempo médio:** < 30 segundos por cupom

### 🛡️ Confiabilidade
- **3 camadas de OCR** com fallback automático
- **Validação dupla** de pontuação (IA + Lógica)
- Sistema de retry automático em caso de falhas
- **Taxa de sucesso:** ~98% com 3 camadas

### 💰 Otimização de Custos
- Priorização de OCR mais econômico (Google Cloud Vision)
- Redução de uso de tokens OpenAI (cálculo híbrido)
- Compressão inteligente de imagens
- **Economia:** 60% em tokens vs versão 1.0

### 🎯 Precisão
- **Base de descritivos cadastrada manualmente**
- Lógica de similaridade para variações de nome (>80%)
- Validação CNPJ e chave de nota fiscal
- Double-check em pontuações

---

## 📈 Métricas de Sucesso

| Métrica | Valor |
|---------|-------|
| **Cupons Processados** | 132.354 |
| **Valor Total em Produtos** | R$ 14.936.852,30 |
| **Média por Cupom** | R$ 112,86 |
| **Taxa de Sucesso OCR** | ~98% (com 3 camadas) |
| **Tempo Médio** | < 30 segundos |
| **Economia de Tokens** | 60% vs v1.0 |

### 📊 Distribuição de Uso de OCR

```mermaid
pie title "Uso de OCR por Camada"
    "Google Cloud Vision (85%)" : 85
    "AWS OCR (12%)" : 12
    "Google Document AI (3%)" : 3
```

---

## 🔮 Evolução do Sistema

```mermaid
timeline
    title Evolução do Sistema de Análise de Cupons
    
    section Versão 1.0
        Sistema Inicial : OCR único
                       : OpenAI calcula tudo
                       : Alto custo
                       
    section Versão 2.0
        OCR em Cascata : 3 camadas OCR
                      : Fallback automático
                      : Melhoria confiabilidade
                      
    section Versão 2.5
        Otimização IA : Cálculo híbrido
                     : Validação dupla
                     : 60% economia tokens
                     
    section Versão 3.0
        Escalabilidade : 10 agentes paralelos
                      : Hangfire orquestrador
                      : Alto throughput
```

### Versão 1.0 (Inicial)
- ❌ OpenAI calculava toda pontuação
- ❌ Alto custo com tokens
- ❌ OCR único
- ❌ Sem processamento paralelo

### Versão 2.0 (Atual)
- ✅ Cálculo híbrido (IA + Lógica)
- ✅ Economia de tokens significativa
- ✅ 3 camadas de OCR
- ✅ 10 agentes paralelos
- ✅ Validação dupla de pontos
- ✅ Base de descritivos expandida

---

## 🛠️ Manutenção e Monitoramento

```mermaid
graph LR
    A[📊 Dashboard Hangfire] --> B[Métricas em Tempo Real]
    
    B --> C[Taxa de Sucesso OCR]
    B --> D[Tempo de Processamento]
    B --> E[Consumo de APIs]
    B --> F[Filas de Processamento]
    
    C --> G[Alertas Automáticos]
    D --> G
    E --> G
    F --> G
    
    G --> H[Equipe de Operações]
    
    style A fill:#e2d9f3,stroke:#6f42c1,stroke-width:2px
    style G fill:#dc3545,stroke:#721c24,stroke-width:3px,color:#fff
    style H fill:#28a745,stroke:#155724,stroke-width:3px,color:#fff
```

**Pontos Monitorados:**
- 📊 Taxa de sucesso por camada de OCR
- ⏱️ Tempo de processamento por agente
- 💰 Consumo de tokens OpenAI
- ⚠️ Erros e exceções
- 📈 Filas de processamento
- 🔍 Cupons em análise manual

**Alertas Configurados:**
- 🚨 Taxa de erro > 5%
- 🚨 Tempo de processamento > 60s
- 🚨 Fila com mais de 100 cupons
- 🚨 Consumo de API acima do esperado

---

## 🎓 Conclusão

O sistema de análise de cupom fiscal da campanha **Sorte de Quem Indica** representa uma solução robusta e escalável que combina:

✅ **Múltiplas tecnologias de OCR** para máxima confiabilidade (98% taxa de sucesso)  
✅ **Inteligência Artificial** para extração e validação de dados  
✅ **Processamento paralelo** com 10 agentes para alta performance  
✅ **Otimização de custos** através de estratégias inteligentes (60% economia)  
✅ **Validação rigorosa** para garantir precisão nos resultados  
✅ **Regras de pontuação flexíveis** (1x, 2x, 3x, 4x) por linha de produto

### 🎯 Impacto da Campanha

**Resultado:** Mais de **132 mil cupons** processados com sucesso, representando quase **R$ 15 milhões** em produtos SpecialDog, com um sistema eficiente, confiável e escalável!



---

*Documento gerado em: 18/11/2025*  
*Sistema: Campanha Sorte de Quem Indica*  
*Versão: 3.0*
