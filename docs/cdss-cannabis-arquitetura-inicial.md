# Sistema de Suporte à Decisão Clínica para Cannabis Medicinal (CDSS)

## 1) Posicionamento do produto (clínico, legal e regulatório)

Este produto deve ser definido e comunicado como **Clinical Decision Support System (CDSS)**.

- O sistema **não prescreve**.
- O sistema **não substitui decisão médica**.
- O sistema **sugere protocolos, evidencia riscos e organiza justificativas clínicas**.
- A responsabilidade final permanece com o médico prescritor (com assinatura e trilha de auditoria).

Essa separação é essencial para reduzir risco regulatório e alinhar o produto às práticas de software médico de apoio à decisão.

---

## 2) Requisitos funcionais por camada

## Camada 1 — Coleta de input clínico estruturado

Campos mínimos obrigatórios:

1. Diagnóstico principal (CID-10)
2. Comorbidades (CID-10 secundários)
3. Medicamentos em uso (nome genérico, dose, frequência)
4. Perfil clínico do paciente:
   - idade
   - sexo
   - peso
   - função hepática/renal (quando relevante)
5. Histórico de exposição à cannabis:
   - naïve
   - tolerante
   - uso recreativo prévio
6. Objetivos terapêuticos:
   - analgesia
   - anticonvulsivante
   - ansiolítico
   - antiemético
   - outros
7. Via de administração preferencial:
   - oral
   - sublingual
   - inalatória

## Camada 2 — Base de conhecimento estruturada

### 2A) Protocolos por patologia

Cada protocolo deve conter:

- indicação clínica e nível de evidência (A/B/C/D)
- quimiótipo preferencial (CBD dominante, THC dominante, proporção THC:CBD)
- dose inicial
- esquema de titulação
- dose alvo
- dose máxima
- janela terapêutica esperada
- tempo sugerido para reavaliação
- referências bibliográficas vinculadas

Indicações iniciais sugeridas para MVP:

- epilepsia refratária (Dravet, LGS)
- dor neuropática crônica
- espasticidade em esclerose múltipla
- náusea associada à quimioterapia
- ansiedade generalizada
- insônia

### 2B) Motor de interações medicamentosas

Enfoque inicial em interações ligadas aos eixos CYP2C9, CYP3A4 e CYP2C19.

Classes críticas para monitoramento no MVP:

- anticoagulantes (ex.: varfarina)
- antiepilépticos (ex.: clobazam)
- imunossupressores (ex.: tacrolimus, ciclosporina)
- antidepressivos (ISRS/IRSN)
- antirretrovirais
- benzodiazepínicos
- estatinas

Classificação de alertas:

- **ALERTA CRÍTICO**: contraindicação, risco grave ou necessidade de bloqueio do protocolo automático
- **ALERTA MODERADO**: uso possível com monitoramento e ajuste
- **INFORMATIVO**: evidência limitada ou baixo impacto esperado

### 2C) Contraindicações absolutas e relativas

Regras clínicas mínimas:

- histórico de psicose/esquizofrenia: restringir exposição a THC
- gestação e amamentação: contraindicação geral (salvo justificativa excepcional formal)
- menor de 18 anos: restringir fora de protocolos pediátricos específicos
- cardiopatia instável: exigir alerta clínico elevado
- dependência ativa de substâncias: gatilho de cautela/contraindicação relativa

## Camada 3 — Output clínico para revisão médica

Saída estruturada por consulta:

1. Protocolo sugerido (quimiótipo, dose inicial, titulação, alvo, via)
2. Score de adequação ao perfil do paciente:
   - adequado
   - adequado com ressalvas
   - não recomendado
3. Alertas de interação por severidade
4. Contraindicações detectadas e justificativa
5. Sugestões de opções disponíveis no Brasil (conforme marcos vigentes)
6. Texto de orientação ao paciente para revisão/assinatura médica
7. Referências bibliográficas usadas na recomendação

---

## 3) Arquitetura técnica inicial (MVP)

### Frontend

- Next.js (React)
- Formulário clínico estruturado por etapas (wizard)
- UX com validação de campos críticos e bloqueios por risco

### Backend

- API em FastAPI (Python) ou Node.js (TypeScript)
- PostgreSQL como fonte primária transacional
- Estrutura híbrida:
  - tabelas relacionais para pacientes (anonimizados), prescrições, interações, auditoria
  - JSON Schema versionado para protocolos por patologia

### IA / LLM

- LLM para sumarização clínica e geração de texto de apoio
- RAG sobre base curada de protocolos e evidências
- embeddings para busca semântica por CID-10 + objetivo terapêutico
- output sempre em JSON estruturado + justificativa legível

### Compliance, segurança e auditoria

- trilha de auditoria imutável por consulta (input, output, versão de protocolo, timestamp)
- assinatura digital do médico no fechamento da decisão
- segregação de dados identificáveis vs. dados clínicos
- criptografia em trânsito e em repouso
- controle de acesso por perfil (RBAC)

---

## 4) Fluxo operacional de ponta a ponta

1. Login médico com validação de credenciais profissionais.
2. Registro do caso clínico em formulário estruturado.
3. Orquestrador executa:
   - busca de protocolos (CID + objetivos)
   - checagem de interações medicamentosas
   - checagem de contraindicações
4. Motor de decisão produz recomendação estruturada.
5. LLM converte em resumo clínico explicável para revisão humana.
6. Médico revisa, ajusta e confirma conduta.
7. Sistema registra assinatura e auditoria final.
8. Exportação para prontuário e documento de orientação ao paciente.

---

## 5) Modelo de dados (mínimo viável)

Tabelas iniciais:

- `clinician`
- `patient_case` (com pseudonimização)
- `case_medication`
- `protocol`
- `protocol_version`
- `drug_interaction`
- `contraindication_rule`
- `recommendation_run`
- `audit_log`
- `clinical_reference`

Campos críticos em `recommendation_run`:

- versão do protocolo
- versão das regras de interação
- versão das regras de contraindicação
- hash do prompt/contexto enviado ao LLM
- resposta estruturada (JSON)
- justificativa textual
- decisão final do médico

---

## 6) Roadmap do MVP (8 a 12 semanas)

### Fase 0 — Definição clínica e regulatória

- delimitação do escopo clínico por especialidade
- validação de protocolos com consultor médico especialista
- definição da matriz de risco clínico-regulatório

### Fase 1 — Núcleo técnico

- formulário estruturado
- banco de protocolos versionado
- motor de interação medicamentosa v1
- motor de contraindicações v1

### Fase 2 — Geração assistida e auditoria

- integração LLM + RAG
- explicabilidade da recomendação
- trilha de auditoria e assinatura digital

### Fase 3 — Piloto controlado

- rollout com pequena coorte de médicos
- coleta de métricas de segurança e utilidade
- revisão de protocolos e regras antes de escalar

---

## 7) Métricas de qualidade e segurança

KPIs sugeridos:

- taxa de recomendações aceitas com ajuste médico
- taxa de alertas críticos por consulta
- taxa de discordância clínica (médico vs. motor)
- tempo médio para gerar recomendação
- completude de documentação para prontuário
- incidentes de segurança e privacidade

---

## 8) Próximos passos práticos

1. Fechar escopo clínico do MVP (5–8 indicações de maior prevalência).
2. Nomear consultor médico responsável pela curadoria inicial.
3. Definir fonte primária de interações (curada/licenciada).
4. Especificar contrato de dados e formato de output JSON.
5. Prototipar interface para 2 fluxos reais (dor crônica e epilepsia refratária).
6. Planejar piloto com governança clínica e jurídica documentada.

---

## Observação final

Este documento é um rascunho técnico inicial de produto. Não constitui diretriz médica, prescrição ou parecer jurídico.
