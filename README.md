## 2. README — Versão 2.0 (Arquitetura Full-Stack Desacoplada: FastAPI + Next.js)

Ideal para o repositório atual modernizado (ou para um monorepo contendo as pastas `backend_ggtab` e `frontend_ggtab`).

```markdown
# 🇧🇷 Plataforma Enterprise de Fiscalização e BI — GGTAB / ANVISA (v2.0 Full-Stack)

![Next.js](https://img.shields.io/badge/Next.js_16-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama_AI-Llama_3.2-000000?style=for-the-badge)

Evolução arquitetural do sistema de fiscalização sanitária da **GGTAB/ANVISA**. A versão 2.0 substitui o protótipo monolítico por uma **arquitetura desacoplada (Client-Server)** de alta performance, combinando um Back-end assíncrono em **Python (FastAPI)** com um Front-end moderno e fluido em **Next.js (React + Tailwind CSS)**, projetado para operação ágil em dispositivos móveis (PWA Ready) e análise executiva de dados (**BI Integrado**).

---

## 🏗️ Arquitetura do Sistema

O projeto é dividido em dois ecossistemas independentes que se comunicam via API REST JSON:

```text
├── 📁 backend_ggtab/          # Cérebro (API REST em Python + FastAPI na porta 8000)
│   └── main.py                # Rotas de BI, CRUD Supabase, Motor Jurídico Ollama, OCR, PDF e Excel
│
└── 📁 frontend_ggtab/         # Corpo (SPA/PWA em Next.js + TypeScript + Tailwind na porta 3000)

1. 📈 Business Intelligence Nativo ("BI fora do BI")
Painel analítico interativo processado em tempo real no Front-end (useMemo), dispensando ferramentas externas:

Slicers Globais: Filtragem dinâmica por Operação específica ou por Modalidade de Infração.

Gráfico de Rosca (SVG Vetorial): Proporção exata de apreensões por natureza da infração.

Rankings Horizontais Dinâmicos:

Top Produtos e Marcas Apreendidas (em volume de unidades).

Incidência por Empresa / Fabricante / CNPJ (separando fabricantes regulares autuados por propaganda de produtos de origem clandestina/DEF).

Mapeamento por Subclasse de Produto (Vapes/Pods, Cigarros, Narguilé, Charutos, Palheiros) e Estabelecimentos Críticos.

2. 📱 Experiência de Campo Mobile-First (Zero Reload)
Navegação instantânea entre abas sem recarregamento de página.

Integração com Câmera Nativa (capture="environment"): Aciona diretamente a câmera traseira de smartphones Android e iOS pelo navegador para registro de evidências.

Robô OCR via API (/api/ocr-cnpj): Envia a foto do rótulo/lote em Base64 para o FastAPI, processa via pytesseract, cruza os dígitos com a base oficial da ANVISA e preenche automaticamente o CNPJ e a marca sugerida.

3. 📝 Central Dinâmica de Histórico, Edição e Exportação
Edição inline em tempo real de qualquer operação (alteração de quantidades, nomes comerciais, CNPJs e status de infração).

Adição retroativa de novos estabelecimentos ou novos produtos diretamente em operações já encerradas, com ressincronização no Supabase.

Streaming de Documentos Oficiais:

POST /api/exportar-pdf: Gera o Auto de Inspeção e Apreensão em PDF com enquadramento normativo automático e fotos embutidas.

POST /api/exportar-excel: Gera a Planilha .xlsx Multi-Abas, isolando cada estabelecimento auditado em uma aba própria.

4. 🤖 Motor Jurídico Híbrido (/api/chat)
Combina RAG determinístico (varredura de CNPJs e portfólios de empresas na base da ANVISA + Supabase) com geração de pareceres via Ollama (Llama 3.2), entregando um Dossiê Corporativo estruturado junto à fundamentação legal (RDC 855/2024, RDC 840/2023, RDC 896/2024 e Lei 6.437/77).

## 🧠 Arquitetura da IA Local: Motor Jurídico Híbrido (Ollama + RAG Determinístico)

ollama pull llama3.2
ollama serve

Diferente de soluções convencionais que dependem de APIs externas pagas (como OpenAI/Claude) e enviam dados sensíveis de fiscalização para servidores de terceiros, esta plataforma implementa um **Motor Jurídico Híbrido 100% Local (On-Premise)** utilizando **Ollama (`llama3.2`)**.

### 🔒 Por que IA Local na Fiscalização Sanitária?
1. **Soberania e Sigilo de Dados:** Informações sobre operações em andamento, estabelecimentos autuados e cruzamento de CNPJs permanecem estritamente na máquina/servidor da **GGTAB/ANVISA**, garantindo conformidade total com a LGPD e o sigilo de processos administrativos.
2. **Custo Zero de Tokenização:** Consultas ilimitadas pelos fiscais sem custos de API por requisição.
3. **Zero Alucinação Cadastral (RAG Determinístico):** Modelos de linguagem puros costumam "alucinar" números de CNPJ ou status de registro. Para eliminar esse risco, criamos um pipeline em **duas camadas**:

### ⚙️ Fluxo de Execução do Pipeline Híbrido (`/api/chat`)

```text
[Pergunta do Fiscal] 
       │
       ▼
┌─────────────────────────────────────────────────────────────────┐
│ 1. CAMADA DETERMINÍSTICA (Python + Regex + Base ANVISA/Supabase)│
│ • Extrai possíveis CNPJs (com ou sem pontuação) e marcas.       │
│ • Varre o banco nativo da ANVISA + banco dinâmico do Supabase.  │
│ • Monta o "Dossiê Corporativo" exato (Fabricante, Portfólio e   │
│   Status) ou dispara o "Alerta Vermelho" de produto clandestino.│
└─────────────────────────────────────────────────────────────────┘
       │ (Injeta o Contexto Auditado no System Prompt)
       ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. CAMADA GENERATIVA LOCAL (Ollama — Llama 3.2, Temp = 0.1)     │
│ • Recebe o dossiê verificado + Arcabouço Legal da GGTAB:        │
│   - RDC 855/2024 (Proibição de DEF / Vapes / Pods)              │
│   - RDC 840/2023 e Lei 9.294/96 (Propaganda Irregular)          │
│   - RDC 896/2024 (Registro Sanitário Obrigatório)               │
│   - Lei 6.437/1977 (Infrações Sanitárias Federais)              │
│ • Redige o parecer técnico-jurídico orientando a ação em campo. │
└─────────────────────────────────────────────────────────────────┘
       │
       ▼
[Interface Next.js: Renderiza Card de Dossiê Oficial + Parecer Jurídico]

🚀 Como Executar o Ambiente CompletoPasso 1: Iniciar o Back-end (FastAPI)Entre no diretório do servidor Python:Bashcd backend_ggtab
Instale as dependências:Bashpip install fastapi uvicorn pydantic supabase ollama pandas openpyxl fpdf2 pillow pytesseract
Configure suas credenciais SUPABASE_URL e SUPABASE_KEY no arquivo main.py.Inicie o servidor Uvicorn na porta 8000:Bashpython -m uvicorn main:app --reload


📌 A documentação interativa da API (Swagger UI) estará disponível em: http://127.0.0.1:8000/docsPasso 2: Iniciar o Front-end (Next.js)Em um novo terminal, acesse a pasta do Front-end:Bashcd frontend_ggtab
Instale os pacotes Node.js (caso ainda não tenha instalado):Bashnpm install
npm install lucide-react
Inicie o servidor de desenvolvimento na porta 3000:Bashnpm run dev
Acesse no navegador: http://localhost:3000


🔌 Principais Endpoints da API (FastAPI)
Método:   Rota:   Descrição:
GET/api/dashboardRetorna os KPIs globais consolidados do Supabase.
GET/api/produtosLista todos os produtos (Base nativa ANVISA + Nuvem Customizada).
POST/api/produtosCadastra uma nova marca comercial diretamente no Supabase.
GET/api/operacoesRecupera o histórico estruturado de operações, lojas e itens.
POST/api/salvar-operacaoCria ou atualiza (sincroniza edições) de uma operação completa na nuvem.
DELETE/api/operacoes/{id}Remove permanentemente uma operação do banco de dados.
POST/api/ocr-cnpjRecebe imagem em Base64, extrai texto via Tesseract OCR e identifica CNPJs.
POST/api/exportar-pdfGera e transmite o Auto de Inspeção em PDF com fotos e base legal.
POST/api/exportar-excelGera e transmite planilha Excel separando cada loja em uma aba.
POST/api/chatProcessa consultas no banco de marcas e gera parecer jurídico via Ollama.
GET/POST/api/logs e /api/logConsulta e registra eventos na trilha de auditoria do sistema.
