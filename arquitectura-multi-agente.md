# Arquitectura Multi-Agente IA - Recomendador de Trayectorias

## Visión General

Arquitectura basada en **agentes IA especializados** que colaboran de forma autónoma para ofrecer recomendaciones de trayectorias académico-profesionales personalizadas. Cada agente tiene un dominio de conocimiento específico y puede usar herramientas, acceder a memoria y coordinar con otros agentes.

## Filosofía de Diseño

### Principios Fundamentales

1. **Especialización por Dominio**: Cada agente es experto en un área específica (orientación profesional, análisis de mercado, catálogo de cursos)
2. **Autonomía**: Los agentes pueden tomar decisiones y ejecutar acciones de forma independiente
3. **Colaboración**: Los agentes se comunican entre sí para resolver problemas complejos
4. **Observabilidad**: Todas las acciones de agentes son trazables y monitorizables
5. **Memoria Compartida**: Los agentes comparten conocimiento relevante para mejorar recomendaciones

### Arquitectura Multi-Layer

```
┌─────────────────────────────────────────────────────────┐
│           Presentation Layer (Web/Mobile/Chat)          │
└─────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│     AI Orchestration Layer (Agent Coordinator)          │
│  - Agent Orchestrator                                   │
│  - Task Planner                                         │
│  - Conversation Manager                                 │
└─────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│         Specialized AI Agents Layer (7 Agents)          │
│  Career Advisor | Market Intelligence | Course Expert   │
│  Student Coach | Skills Assessment | Enrollment         │
│  Research Agent                                         │
└─────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│      Agent Tools & Capabilities Layer                   │
│  RAG Engine | Function Registry | Web Search            │
│  Calculator & Analytics Tools                           │
└─────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│      Memory & Knowledge Layer                           │
│  Agent Memory | Vector DB | Knowledge Graph             │
│  Shared Memory                                          │
└─────────────────────────────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────┐
│      Domain Services & Data Layer                       │
│  Profile Service | Course Catalog | Job Market Analyzer │
│  Student DB | Courses DB | Job Market DB                │
└─────────────────────────────────────────────────────────┘
```

## Capa 1: AI Orchestration Layer

### Agent Orchestrator (Orquestador Central)

**Responsabilidad**: Coordinador maestro que gestiona el ciclo de vida de las consultas de usuarios.

**Framework**: LangGraph + CrewAI

**Funcionalidades**:
- Recibe intención del usuario desde el API Gateway
- Analiza el objetivo y contexto de la consulta
- Selecciona qué agentes deben participar y en qué orden
- Coordina comunicación entre agentes (secuencial o paralela)
- Agrega resultados de múltiples agentes
- Gestiona excepciones y reintentos
- Proporciona respuesta final estructurada al usuario

**Patrones de Orquestación**:

1. **Sequential Chain**: Agentes ejecutan en secuencia
   ```
   User Query → Task Planner → Skills Assessment Agent
   → Career Advisor Agent → Course Expert Agent → Response
   ```

2. **Parallel Execution**: Agentes ejecutan en paralelo
   ```
   User Query → Task Planner → [Market Intelligence Agent
                                 Course Expert Agent
                                 Research Agent] → Aggregation → Response
   ```

3. **Hierarchical**: Agente principal delega a sub-agentes
   ```
   User Query → Career Advisor Agent → [Skills Assessment Agent
                                         Course Expert Agent] → Career Advisor → Response
   ```

**Ejemplo de código (LangGraph)**:
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

class AgentState(TypedDict):
    user_query: str
    user_profile: dict
    task_plan: list
    agent_results: Annotated[list, operator.add]
    final_recommendation: dict

def plan_tasks(state: AgentState):
    # Task Planner analiza query y crea plan
    plan = task_planner.create_plan(state["user_query"])
    return {"task_plan": plan}

def execute_skills_assessment(state: AgentState):
    # Skills Assessment Agent evalúa competencias
    result = skills_assessment_agent.run(state["user_profile"])
    return {"agent_results": [result]}

def execute_career_advisor(state: AgentState):
    # Career Advisor Agent genera recomendaciones
    result = career_advisor_agent.run(
        state["user_profile"],
        state["agent_results"]
    )
    return {"final_recommendation": result}

# Definir workflow
workflow = StateGraph(AgentState)
workflow.add_node("plan", plan_tasks)
workflow.add_node("assess_skills", execute_skills_assessment)
workflow.add_node("recommend", execute_career_advisor)

workflow.set_entry_point("plan")
workflow.add_edge("plan", "assess_skills")
workflow.add_edge("assess_skills", "recommend")
workflow.add_edge("recommend", END)

app = workflow.compile()
```

### Task Planner (Planificador de Tareas)

**Responsabilidad**: Planifica la secuencia óptima de agentes para resolver la consulta del usuario.

**Funcionalidades**:
- Analiza intención del usuario usando LLM
- Identifica agentes necesarios para cumplir objetivo
- Determina dependencias entre tareas
- Genera plan de ejecución (secuencial, paralelo, condicional)
- Estima complejidad y tiempo de ejecución

**Algoritmo**: ReAct (Reasoning + Acting) o Plan-and-Execute

**Ejemplo de Plan**:
```json
{
  "user_intent": "recomendar_trayectoria_data_science",
  "complexity": "medium",
  "execution_plan": [
    {
      "step": 1,
      "agent": "skills_assessment_agent",
      "action": "evaluate_current_competencies",
      "parallel": false
    },
    {
      "step": 2,
      "agents": ["market_intelligence_agent", "course_expert_agent"],
      "action": "gather_market_and_course_data",
      "parallel": true
    },
    {
      "step": 3,
      "agent": "career_advisor_agent",
      "action": "generate_recommendations",
      "dependencies": [1, 2]
    }
  ]
}
```

### Conversation Manager (Gestor de Conversación)

**Responsabilidad**: Mantiene contexto conversacional multi-turno y estado de sesión.

**Funcionalidades**:
- Almacena historial de conversación
- Gestiona contexto de sesión (usuario, preferencias, estado actual)
- Resuelve correferencias ("ese máster", "la segunda opción")
- Detecta cambios de tema
- Gestiona interrupciones y clarificaciones

**Stack**: LangChain + Redis (para sesiones)

## Capa 2: Specialized AI Agents Layer

### 1. Career Advisor Agent (Agente Asesor de Carrera)

**Rol**: Asesor profesional experto en orientación vocacional y planificación de trayectorias.

**Personalidad**: Empático, estratégico, orientado a resultados.

**Capacidades**:
- Analiza perfil del estudiante (background, intereses, objetivos)
- Identifica brechas entre situación actual y objetivos profesionales
- Genera trayectorias multi-nivel (Grado → Máster → Certificación)
- Evalúa alineamiento con mercado laboral
- Proporciona explicaciones detalladas de cada recomendación
- Considera restricciones (tiempo, presupuesto, modalidad)

**Herramientas disponibles**:
- `get_student_profile()`: Obtiene perfil completo del estudiante
- `search_career_paths()`: Busca trayectorias similares exitosas
- `calculate_roi()`: Calcula ROI estimado de una trayectoria
- `get_job_outlook()`: Obtiene perspectivas laborales de un campo

**Memoria**:
- Casos de éxito previos (trayectorias que funcionaron bien)
- Patrones de decisión por perfil de estudiante
- Feedback de estudiantes sobre recomendaciones

**Prompt System**:
```python
CAREER_ADVISOR_SYSTEM_PROMPT = """
Eres un asesor de carrera experto con 15 años de experiencia en orientación vocacional
y diseño de trayectorias académicas. Tu objetivo es ayudar a estudiantes a diseñar
trayectorias formativas que maximicen su empleabilidad y alineación con sus objetivos
profesionales.

## Tu Proceso:
1. Analiza el perfil del estudiante (competencias, intereses, restricciones)
2. Evalúa brechas respecto a objetivos profesionales
3. Busca en tu memoria casos similares exitosos
4. Consulta tendencias del mercado laboral actual
5. Diseña 2-3 trayectorias alternativas (conservadora, equilibrada, ambiciosa)
6. Explica pros/contras y ROI de cada trayectoria
7. Recomienda la más adecuada según perfil y contexto

## Herramientas disponibles:
{tools}

## Formato de Respuesta:
Proporciona recomendaciones estructuradas, con explicaciones claras del razonamiento
y datos concretos (empleabilidad, salario esperado, tiempo estimado).
"""
```

**Ejemplo de interacción**:
```python
# Usuario: "Quiero ser Data Scientist pero solo tengo un grado en Economía"

result = career_advisor_agent.run(
    user_profile={
        "current_degree": "Grado en Economía",
        "target_role": "Data Scientist",
        "current_skills": ["Excel", "Estadística básica"],
        "constraints": {"budget": "medium", "time": "2 años"}
    }
)

# Agente planifica:
# 1. Llamar a skills_assessment_agent para evaluar gap
# 2. Consultar market_intelligence_agent sobre demanda de Data Scientists
# 3. Buscar en course_catalog cursos de "Data Science" compatibles con background en Economía
# 4. Generar 3 trayectorias alternativas
# 5. Calcular ROI y probabilidad de éxito de cada una
# 6. Recomendar la óptima
```

### 2. Market Intelligence Agent (Agente de Inteligencia de Mercado)

**Rol**: Analista de mercado laboral especializado en tendencias de empleo y competencias.

**Personalidad**: Analítico, basado en datos, prospectivo.

**Capacidades**:
- Monitoriza tendencias del mercado laboral en tiempo real
- Identifica competencias emergentes y en declive
- Analiza demanda por rol, sector y ubicación
- Detecta sectores en crecimiento
- Proporciona insights salariales
- Predice futuras necesidades del mercado

**Herramientas disponibles**:
- `analyze_job_trends()`: Analiza tendencias de ofertas de empleo
- `extract_top_skills()`: Extrae competencias más demandadas
- `get_salary_insights()`: Obtiene rangos salariales por rol
- `detect_emerging_roles()`: Identifica roles profesionales emergentes
- `compare_markets()`: Compara mercados laborales (ciudades, países)

**Fuentes de datos**:
- Job Market DB (MongoDB con ofertas scraped)
- LinkedIn API (ofertas en tiempo real)
- InfoJobs API (mercado español)
- Web Search Tool (artículos sobre tendencias)

**Análisis que realiza**:

1. **Trend Analysis**: Evolución temporal de demanda
   ```python
   # Ejemplo: Demanda de "Python" en ofertas de Data Science
   {
     "skill": "Python",
     "role": "Data Scientist",
     "trend": "increasing",
     "growth_rate": "+25% YoY",
     "current_demand": "8,500 ofertas/mes",
     "regions": ["Madrid", "Barcelona", "Valencia"]
   }
   ```

2. **Skills Gap Analysis**: Competencias más demandadas vs. oferta de graduados
   ```python
   {
     "role": "Cloud Architect",
     "high_demand_skills": ["Kubernetes", "Terraform", "AWS"],
     "supply_gap": "alto",  # Pocas personas con estas skills
     "opportunity_score": 9.2  # Alta oportunidad
   }
   ```

3. **Salary Benchmarking**: Rangos salariales por competencias
   ```python
   {
     "role": "Data Engineer",
     "base_skills_salary": "35K-45K EUR",
     "with_spark": "+8K EUR",
     "with_cloud": "+12K EUR",
     "with_ml": "+10K EUR"
   }
   ```

### 3. Course Expert Agent (Agente Experto en Cursos)

**Rol**: Experto en el catálogo formativo de la Universidad Europea.

**Personalidad**: Detallista, educador, servicial.

**Capacidades**:
- Conocimiento profundo de todos los cursos, másteres y certificaciones
- Responde dudas sobre contenidos, modalidades, requisitos
- Sugiere formaciones complementarias
- Identifica prerequisitos y dependencias
- Explica competencias desarrolladas por cada formación

**Herramientas disponibles**:
- `search_courses()`: Búsqueda semántica en catálogo
- `get_course_details()`: Detalles completos de un curso
- `find_prerequisites()`: Identifica requisitos previos
- `suggest_complementary()`: Sugiere cursos complementarios
- `check_competency_coverage()`: Verifica si un curso cubre competencias específicas

**RAG (Retrieval Augmented Generation)**:
- Vector DB con embeddings de todos los cursos
- Documentación oficial de programas (syllabi, guías docentes)
- FAQs sobre formaciones

**Ejemplo de consulta**:
```python
# Usuario: "¿Qué máster me recomiendas para especializarme en Machine Learning?"

course_expert_agent.run(
    query="Máster en Machine Learning",
    context={
        "current_degree": "Grado en Ingeniería Informática",
        "target_skills": ["Deep Learning", "NLP", "Computer Vision"]
    }
)

# Agente:
# 1. Busca en Vector DB cursos con embeddings similares a "Machine Learning"
# 2. Filtra por nivel "Máster"
# 3. Verifica cobertura de competencias requeridas
# 4. Consulta requisitos de admisión
# 5. Retorna 2-3 opciones con pros/contras
```

### 4. Student Coach Agent (Agente Coach del Estudiante)

**Rol**: Guía personal que acompaña al estudiante en su proceso de decisión.

**Personalidad**: Empático, paciente, motivador.

**Capacidades**:
- Guía conversacional personalizada
- Hace preguntas para entender mejor objetivos y restricciones
- Ayuda a clarificar dudas y miedos
- Proporciona motivación y perspectiva
- Recuerda interacciones previas y contexto personal
- Adapta comunicación al perfil del estudiante (edad, background)

**Memoria episódica**:
- Historial completo de interacciones con el estudiante
- Preferencias expresadas
- Dudas recurrentes
- Progreso en el proceso de decisión

**Prompt System**:
```python
STUDENT_COACH_SYSTEM_PROMPT = """
Eres un coach educativo empático y experimentado. Tu rol es acompañar al estudiante
en su proceso de decisión sobre su trayectoria académica y profesional.

## Tu Estilo:
- Escucha activa: haces preguntas abiertas para entender mejor
- Empático: reconoces preocupaciones y miedos
- No juzgas: respetas todas las situaciones y objetivos
- Motivador: ayudas a ver posibilidades y fortalezas
- Claro: explicas conceptos complejos de forma simple

## Recuerdas:
- Todas las conversaciones previas con este estudiante
- Sus objetivos, preocupaciones y restricciones
- Su fase actual en el proceso de decisión

## Evitas:
- Presionar o forzar decisiones
- Dar respuestas técnicas (delegas a agentes especializados)
- Hacer promesas que no puedes cumplir
"""
```

**Ejemplo de interacción**:
```
Usuario: "No estoy seguro si puedo permitirme un máster ahora"

Student Coach Agent:
"Entiendo tu preocupación sobre el aspecto económico. Es completamente normal
planteárselo. ¿Me puedes contar un poco más sobre tu situación? ¿Has explorado
opciones como becas, financiación o programas part-time que te permitan trabajar
mientras estudias?"

[Almacena en memoria: usuario tiene restricción presupuestaria]
[Notifica a Career Advisor Agent: filtrar recomendaciones por opciones de financiación]
```

### 5. Skills Assessment Agent (Agente de Evaluación de Competencias)

**Rol**: Evaluador técnico de competencias y habilidades.

**Personalidad**: Objetivo, preciso, constructivo.

**Capacidades**:
- Evalúa competencias actuales del estudiante
- Identifica gaps respecto a roles profesionales objetivo
- Sugiere áreas de mejora prioritarias
- Verifica competencias mediante preguntas o tests
- Mapea experiencia laboral/académica a competencias

**Herramientas disponibles**:
- `assess_competency_level()`: Evalúa nivel de dominio de una competencia
- `identify_skill_gaps()`: Compara perfil actual vs. perfil objetivo
- `map_experience_to_skills()`: Extrae competencias de experiencia descrita
- `suggest_learning_path()`: Propone secuencia de aprendizaje

**Modelos ML utilizados**:
- Modelo de NLP para extraer competencias de CV/LinkedIn
- Modelo de clasificación de nivel de competencia (básico, intermedio, avanzado)
- Modelo de similitud semántica entre competencias

**Ejemplo**:
```python
skills_assessment_agent.run(
    student_profile={
        "experience": "2 años como analista de datos en sector financiero",
        "skills_claimed": ["Python", "SQL", "Excel", "Tableau"],
        "target_role": "Data Scientist"
    }
)

# Agente:
# 1. Extrae competencias de experiencia usando NLP
# 2. Consulta en Knowledge Graph requisitos de "Data Scientist"
# 3. Identifica gaps: ["Machine Learning", "Deep Learning", "MLOps"]
# 4. Evalúa nivel actual: Python (intermedio), SQL (avanzado)
# 5. Retorna assessment detallado con áreas de mejora
```

### 6. Enrollment Agent (Agente de Matriculación)

**Rol**: Asistente de gestión de matrícula y trámites administrativos.

**Personalidad**: Eficiente, claro, servicial.

**Capacidades**:
- Guía paso a paso en proceso de matrícula
- Resuelve dudas sobre documentación, plazos, pagos
- Integración directa con Sistema de Matrícula
- Gestiona excepciones y casos especiales
- Proporciona información sobre becas y financiación

**Herramientas disponibles**:
- `start_enrollment()`: Inicia proceso de matrícula
- `check_enrollment_status()`: Consulta estado del trámite
- `upload_document()`: Carga documentación requerida
- `apply_for_scholarship()`: Solicita beca
- `calculate_fees()`: Calcula coste total y opciones de pago

**Integración con sistemas**:
- Sistema de Matrícula (API REST)
- Sistema de Becas
- Sistema de Pagos
- CRM (para seguimiento de leads)

### 7. Research Agent (Agente de Investigación)

**Rol**: Investigador que busca información actualizada en tiempo real.

**Personalidad**: Curioso, riguroso, verificador de fuentes.

**Capacidades**:
- Búsquedas web de información actualizada
- Investigación sobre universidades, programas, becas
- Fact-checking de información proporcionada
- Recopilación de opiniones y reviews de programas
- Análisis de noticias sobre tendencias educativas

**Herramientas disponibles**:
- Web Search Tool (Tavily API / SerpAPI)
- Web Scraping (Beautiful Soup para páginas específicas)
- Document Reader (para PDFs de programas académicos)

**Casos de uso**:
- "¿Qué opinan los estudiantes del Máster en IA de la Universidad Europea?"
- "¿Hay becas disponibles para estudiantes internacionales?"
- "¿Cuáles son las últimas tendencias en educación sobre ciberseguridad?"

## Capa 3: Agent Tools & Capabilities Layer

### RAG Engine (Motor de Generación Aumentada por Recuperación)

**Responsabilidad**: Proporcionar contexto relevante a agentes desde base de conocimiento.

**Stack**: LangChain + LlamaIndex

**Componentes**:

1. **Document Loaders**: Carga documentación desde múltiples fuentes
   - PDFs de programas académicos
   - Páginas web de la universidad
   - Syllabi y guías docentes
   - FAQs

2. **Text Splitters**: Divide documentos en chunks semánticamente coherentes
   ```python
   from langchain.text_splitter import RecursiveCharacterTextSplitter

   splitter = RecursiveCharacterTextSplitter(
       chunk_size=1000,
       chunk_overlap=200,
       separators=["\n\n", "\n", ". ", " "]
   )
   ```

3. **Embeddings**: Genera vectores semánticos
   ```python
   from langchain.embeddings import HuggingFaceEmbeddings

   embeddings = HuggingFaceEmbeddings(
       model_name="sentence-transformers/paraphrase-multilingual-mpnet-base-v2"
   )
   ```

4. **Vector Store**: Almacena y recupera chunks por similitud
   ```python
   from langchain.vectorstores import Qdrant

   vectorstore = Qdrant.from_documents(
       documents,
       embeddings,
       url="http://qdrant:6333",
       collection_name="university_knowledge"
   )
   ```

5. **Retriever**: Recupera top-k documentos relevantes
   ```python
   retriever = vectorstore.as_retriever(
       search_type="mmr",  # Maximal Marginal Relevance
       search_kwargs={"k": 5, "fetch_k": 20}
   )
   ```

**Ejemplo de uso por agente**:
```python
# Course Expert Agent necesita información sobre un máster
query = "Contenidos del Máster en Inteligencia Artificial"
relevant_docs = rag_engine.retrieve(query)

# relevant_docs contiene:
# - Syllabus del máster
# - Descripción de asignaturas
# - Requisitos de admisión
# - Salidas profesionales

# Agente usa estos docs como contexto para responder
```

### Function Registry (Registro de Funciones/Herramientas)

**Responsabilidad**: Catálogo de herramientas disponibles para agentes (function calling).

**Stack**: LangChain Tools

**Funciones disponibles**:

```python
from langchain.tools import Tool
from typing import List

tools: List[Tool] = [
    Tool(
        name="get_student_profile",
        description="Obtiene el perfil completo de un estudiante dado su ID",
        func=profile_service.get_profile
    ),
    Tool(
        name="search_courses",
        description="Busca cursos en el catálogo dado keywords y filtros",
        func=course_catalog_service.search
    ),
    Tool(
        name="analyze_job_market",
        description="Analiza tendencias del mercado laboral para un rol específico",
        func=job_market_analyzer.analyze_role
    ),
    Tool(
        name="calculate_roi",
        description="Calcula el ROI estimado de una inversión formativa",
        func=calculator_tool.calculate_roi
    ),
    Tool(
        name="start_enrollment",
        description="Inicia el proceso de matrícula para un estudiante",
        func=matricula_system.start_enrollment
    ),
    Tool(
        name="web_search",
        description="Realiza una búsqueda web para obtener información actualizada",
        func=web_search_tool.search
    )
]
```

**Ejemplo de uso (ReAct pattern)**:
```
Usuario: "¿Cuánto gana un Data Scientist en Madrid?"

Agente (Market Intelligence):
Thought: Necesito información actualizada sobre salarios de Data Scientist en Madrid
Action: web_search
Action Input: "Data Scientist salary Madrid 2026"
Observation: Los Data Scientists en Madrid ganan entre 45K-75K EUR según experiencia...

Thought: Ahora tengo información actualizada, puedo responder
Action: Responder al usuario
```

### Web Search Tool (Herramienta de Búsqueda Web)

**Responsabilidad**: Búsquedas web para información en tiempo real.

**Proveedores**:
- **Tavily API** (recomendado): Diseñado para LLMs, retorna resultados limpios
- **SerpAPI**: Acceso a Google Search con parsing automático
- **Bing Search API**: Alternativa de Microsoft

**Ejemplo**:
```python
from tavily import TavilyClient

tavily = TavilyClient(api_key="...")

def web_search(query: str, max_results: int = 5) -> List[dict]:
    """Busca en la web y retorna resultados relevantes"""
    results = tavily.search(
        query=query,
        search_depth="advanced",
        max_results=max_results
    )
    return results["results"]
```

### Calculator & Analytics Tool (Herramientas de Cálculo)

**Responsabilidad**: Cálculos numéricos, análisis estadístico, visualizaciones.

**Capacidades**:
- Cálculo de ROI de inversiones formativas
- Análisis de tendencias temporales
- Agregaciones estadísticas
- Comparaciones de métricas

**Stack**: NumPy, Pandas, Matplotlib

**Ejemplo - Cálculo de ROI**:
```python
def calculate_education_roi(
    investment: float,  # Coste del máster
    salary_increase: float,  # Incremento salarial esperado
    years: int = 5  # Horizonte temporal
) -> dict:
    """Calcula ROI de una inversión formativa"""

    total_benefit = salary_increase * years
    roi_percentage = ((total_benefit - investment) / investment) * 100
    breakeven_years = investment / salary_increase

    return {
        "roi_percentage": roi_percentage,
        "breakeven_years": breakeven_years,
        "total_benefit": total_benefit,
        "net_benefit": total_benefit - investment
    }

# Ejemplo:
# Máster cuesta 15K EUR
# Incremento salarial esperado: 10K EUR/año
roi = calculate_education_roi(15000, 10000, 5)
# roi = {
#   "roi_percentage": 233%,
#   "breakeven_years": 1.5,
#   "total_benefit": 50K EUR,
#   "net_benefit": 35K EUR
# }
```

## Capa 4: Memory & Knowledge Layer

### Agent Memory Store (Memoria de Agentes)

**Base de datos**: PostgreSQL + pgvector (para embeddings)

**Tipos de memoria**:

1. **Memoria Episódica** (conversaciones pasadas):
   ```sql
   CREATE TABLE agent_memory_episodes (
       id SERIAL PRIMARY KEY,
       agent_id VARCHAR(100),
       user_id VARCHAR(100),
       session_id VARCHAR(100),
       interaction_type VARCHAR(50),
       user_message TEXT,
       agent_response TEXT,
       metadata JSONB,
       embedding VECTOR(768),
       created_at TIMESTAMP
   );
   ```

2. **Memoria Semántica** (conocimiento aprendido):
   ```sql
   CREATE TABLE agent_knowledge (
       id SERIAL PRIMARY KEY,
       agent_id VARCHAR(100),
       knowledge_type VARCHAR(50), -- 'insight', 'pattern', 'rule'
       content TEXT,
       confidence FLOAT,
       source VARCHAR(100),
       embedding VECTOR(768),
       created_at TIMESTAMP
   );
   ```

3. **Memoria de Trabajo** (contexto temporal):
   - Almacenada en Redis con TTL
   - Contexto de sesión actual
   - Variables intermedias

**Ejemplo de uso**:
```python
# Career Advisor Agent recuerda caso similar exitoso
similar_cases = agent_memory.search_similar_episodes(
    query_embedding=current_student_embedding,
    agent_id="career_advisor",
    interaction_type="successful_recommendation",
    top_k=3
)

# Usa estos casos como ejemplos para mejorar recomendación actual
```

### Vector Knowledge Base (Base de Conocimiento Vectorial)

**Tecnología**: Qdrant (self-hosted) o Pinecone (managed)

**Colecciones**:

1. **Courses Collection**: Embeddings de cursos
   ```python
   {
       "id": "master-ia-001",
       "vector": [0.234, -0.123, ...],  # 768 dims
       "payload": {
           "name": "Máster en Inteligencia Artificial",
           "type": "master",
           "duration": "12 meses",
           "competencies": ["ML", "Deep Learning", "NLP"],
           "price": 15000
       }
   }
   ```

2. **Student Profiles Collection**: Embeddings de perfiles
3. **Job Offers Collection**: Embeddings de ofertas de empleo
4. **FAQs Collection**: Embeddings de preguntas frecuentes

**Búsqueda híbrida** (vector + filtros):
```python
search_result = qdrant_client.search(
    collection_name="courses",
    query_vector=user_query_embedding,
    query_filter={
        "must": [
            {"key": "type", "match": {"value": "master"}},
            {"key": "price", "range": {"lte": 20000}}
        ]
    },
    limit=10
)
```

### Shared Memory Service (Memoria Compartida)

**Tecnología**: Redis + Semantic Kernel Memory

**Propósito**: Coordinación entre agentes mediante memoria compartida.

**Casos de uso**:

1. **Agent Handoffs**: Pasar contexto entre agentes
   ```python
   # Student Coach Agent pasa contexto a Career Advisor Agent
   shared_memory.set(
       key=f"session:{session_id}:context",
       value={
           "user_concerns": ["budget constraints", "time commitment"],
           "preferences": {"modality": "online", "language": "Spanish"},
           "current_phase": "exploring_options"
       },
       ttl=3600  # 1 hora
   )
   ```

2. **Insights Sharing**: Agentes comparten descubrimientos
   ```python
   # Market Intelligence Agent comparte insight
   shared_memory.publish(
       channel="market_insights",
       message={
           "type": "emerging_skill",
           "skill": "Prompt Engineering",
           "trend": "growing_fast",
           "demand_increase": "+300% YoY"
       }
   )

   # Career Advisor Agent está suscrito y ajusta recomendaciones
   ```

### Knowledge Graph (Grafo de Conocimiento)

**Tecnología**: Neo4j

**Estructura**:

```cypher
// Nodos
(Student)-[:HAS_SKILL]->(Skill)
(Student)-[:INTERESTED_IN]->(Role)
(Course)-[:TEACHES]->(Skill)
(Course)-[:LEADS_TO]->(Role)
(Role)-[:REQUIRES]->(Skill)
(Course)-[:PREREQUISITE]->(Course)
(Trajectory)-[:INCLUDES]->(Course)
```

**Queries útiles**:

1. **Encontrar trayectorias de estudiante a rol objetivo**:
   ```cypher
   MATCH path = (s:Student {id: $student_id})-[:HAS_SKILL*]->(sk:Skill)
               <-[:TEACHES]-(c:Course)-[:LEADS_TO]->(r:Role {name: $target_role})
   WHERE NOT (s)-[:COMPLETED]->(c)
   RETURN path
   ORDER BY length(path) ASC
   LIMIT 5
   ```

2. **Identificar competencias faltantes**:
   ```cypher
   MATCH (s:Student {id: $student_id}),
         (r:Role {name: $target_role})-[:REQUIRES]->(required_skill:Skill)
   WHERE NOT (s)-[:HAS_SKILL]->(required_skill)
   RETURN required_skill.name, required_skill.importance
   ORDER BY required_skill.importance DESC
   ```

3. **Recomendar siguiente curso en trayectoria**:
   ```cypher
   MATCH (s:Student {id: $student_id})-[:HAS_SKILL]->(sk:Skill),
         (next_course:Course)-[:TEACHES]->(new_skill:Skill),
         (next_course)-[:LEADS_TO]->(s.target_role)
   WHERE NOT (s)-[:COMPLETED]->(next_course)
     AND (next_course)-[:PREREQUISITE*0..1]->(:Course)<-[:COMPLETED]-(s)
   RETURN next_course, count(sk) as skills_overlap
   ORDER BY skills_overlap DESC
   LIMIT 3
   ```

## Flujos de Interacción Multi-Agente

### Flujo 1: Recomendación de Trayectoria Completa (Multi-Agent Collaboration)

```
1. Usuario: "Quiero ser Cloud Architect pero vengo de un Grado en Física"

2. API Gateway → Agent Orchestrator

3. Agent Orchestrator → Task Planner
   - Analiza intención: "recomendar_trayectoria_profesional"
   - Complejidad: HIGH (requiere assessment, análisis de mercado, diseño de trayectoria)
   - Plan: Sequential + Parallel execution

4. Task Planner retorna plan:
   Step 1: Skills Assessment Agent (evaluar competencias actuales)
   Step 2 (paralelo): Market Intelligence Agent + Course Expert Agent
   Step 3: Career Advisor Agent (sintetizar y recomendar)
   Step 4: Student Coach Agent (presentar y acompañar)

5. Agent Orchestrator ejecuta Step 1:
   → Skills Assessment Agent
     - Obtiene perfil: get_student_profile(user_id)
     - Analiza background: "Grado en Física" → extrae skills ["Matemáticas avanzadas", "Programación básica (Python)"]
     - Consulta Knowledge Graph: requisitos de "Cloud Architect"
     - Identifica gaps: ["Cloud Platforms (AWS/Azure)", "Containers", "IaC", "DevOps", "Networking"]
     - Retorna: {"current_skills": [...], "missing_skills": [...], "transferable_skills": [...]}

6. Agent Orchestrator ejecuta Step 2 (paralelo):

   → Market Intelligence Agent:
     - Consulta Job Market DB: ofertas de "Cloud Architect"
     - Llama analyze_job_trends("Cloud Architect")
     - Calcula demanda: "Alta (+40% ofertas YoY)"
     - Identifica top skills: ["Kubernetes", "Terraform", "AWS", "CI/CD"]
     - Obtiene salary insights: "55K-85K EUR en España"
     - Retorna: market_analysis

   → Course Expert Agent:
     - Busca en Vector DB: cursos relacionados con "Cloud Architecture"
     - Filtra por compatible con background "Física" (requiere base técnica)
     - Encuentra: "Máster en Cloud Computing", "Certificación AWS Solutions Architect", "Bootcamp DevOps"
     - Retorna: relevant_courses

7. Agent Orchestrator agrega resultados y ejecuta Step 3:
   → Career Advisor Agent
     - Recibe: skills_assessment + market_analysis + relevant_courses
     - Consulta shared_memory para ver si hay constraints del usuario
     - Busca en agent_memory casos similares: "Físicos → Cloud Architects"
     - Diseña 3 trayectorias alternativas:

       Trayectoria 1 (Conservadora - 2 años):
         - Bootcamp DevOps (6 meses) → Certificación AWS (3 meses) → Prácticas → Empleo
         - Coste: 8K EUR | ROI: 250% en 3 años | Riesgo: Bajo

       Trayectoria 2 (Equilibrada - 2.5 años):
         - Máster Cloud Computing (12 meses) → Certificaciones AWS + Azure → Empleo
         - Coste: 18K EUR | ROI: 300% en 4 años | Riesgo: Medio

       Trayectoria 3 (Ambiciosa - 3 años):
         - Máster en Ingeniería de Software (18 meses) → Certificaciones → Specialization Cloud Security → Empleo senior
         - Coste: 25K EUR | ROI: 400% en 5 años | Riesgo: Alto

     - Recomienda: Trayectoria 2 (mejor balance para perfil)
     - Retorna: recommendations

8. Agent Orchestrator ejecuta Step 4:
   → Student Coach Agent
     - Recibe recommendations
     - Consulta agent_memory: historial previo con este usuario
     - Adapta comunicación: tono motivador, destaca ventajas del background en Física
     - Genera respuesta empática y estructurada
     - Almacena interacción en agent_memory
     - Retorna: final_response

9. Agent Orchestrator → API Gateway → Usuario

10. Usuario recibe:
    "Basándome en tu perfil, he diseñado 3 trayectorias posibles para convertirte en Cloud Architect.

    Tu background en Física es una gran ventaja - tienes pensamiento analítico sólido y ya conoces Python.

    [Trayectoria 1: Conservadora]
    ...

    [Trayectoria 2: Equilibrada - RECOMENDADA]
    ✅ Máster en Cloud Computing (12 meses)
       - Aprenderás AWS, Azure, Kubernetes, Terraform
       - Incluye proyecto real de arquitectura cloud
       - Coste: 15.000 EUR (becas disponibles)

    ✅ Certificaciones AWS Solutions Architect + Azure Administrator (6 meses)
       - Validación oficial de competencias
       - Alta demanda en el mercado
       - Coste: 500 EUR por certificación

    ✅ Perspectivas laborales:
       - Demanda: MUY ALTA (+40% ofertas este año)
       - Salario inicial: 50-60K EUR
       - Salario con experiencia: 70-90K EUR
       - ROI estimado: 300% en 4 años

    [Trayectoria 3: Ambiciosa]
    ...

    ¿Te gustaría que profundice en alguna de estas opciones?"
```

### Flujo 2: Conversación Multi-Turno con Handoffs entre Agentes

```
Turno 1:
Usuario: "Hola, estoy pensando en estudiar algo relacionado con datos"

Agent Orchestrator → Student Coach Agent
  - Detecta fase: "exploración inicial"
  - Responde con preguntas para clarificar:
    "¡Hola! Me alegra que estés interesado en el mundo de datos. Es un campo con muchas
    oportunidades. Para poder ayudarte mejor, ¿me puedes contar un poco más?
    - ¿Qué te atrae específicamente del área de datos?
    - ¿Tienes alguna formación o experiencia previa?
    - ¿Qué tipo de trabajo te gustaría hacer? (análisis, ingeniería, ciencia de datos...)"

  - Almacena en shared_memory: {phase: "exploration", topic: "data"}

Turno 2:
Usuario: "Me gusta mucho el tema de análisis y sacar insights. Tengo un grado en Economía
y he trabajado 1 año como analista junior"

Student Coach Agent:
  - Extrae información clave:
    * Background: Economía
    * Experiencia: 1 año analista
    * Interés: análisis, insights
  - Actualiza shared_memory
  - Decide: necesita evaluación técnica
  - Handoff → Skills Assessment Agent

Skills Assessment Agent:
  - Lee shared_memory para contexto
  - Pregunta técnicas:
    "Genial, tu experiencia en análisis es muy valiosa. Para entender mejor tu nivel técnico:
    - ¿Qué herramientas usas actualmente? (Excel, SQL, Python, R, Tableau...)
    - ¿Has trabajado con bases de datos?
    - ¿Conoces conceptos de Machine Learning?"

  - Almacena en shared_memory: {assessment_in_progress: true}

Turno 3:
Usuario: "Uso mucho Excel y SQL. Algo de Python pero básico. Machine Learning no he tocado"

Skills Assessment Agent:
  - Evalúa nivel:
    * Excel: Avanzado
    * SQL: Intermedio-Avanzado
    * Python: Básico
    * ML: Ninguno
  - Identifica buen punto de partida para Data Science
  - Almacena assessment en agent_memory
  - Actualiza shared_memory: {current_skills: [...], target_domain: "data_science"}
  - Handoff → Career Advisor Agent

Career Advisor Agent:
  - Lee shared_memory: contexto completo
  - Lee agent_memory: assessment de skills
  - Consulta Market Intelligence Agent (llamada interna): demanda de Data Scientists
  - Consulta Course Expert Agent: cursos de Data Science para perfil Economía + SQL
  - Genera recomendación preliminar:
    "Basándome en tu perfil (Economía + SQL + analítica), tienes una excelente base para
    Data Science. Te recomendaría:

    1. Máster en Data Science (enfoque business)
       - Aprovecha tu background en negocio
       - Fortalece Python y ML
       - Duración: 12 meses

    2. Bootcamp de Machine Learning
       - Más intensivo y práctico
       - Duración: 6 meses
       - Menor coste

    ¿Tienes preferencia entre formación más académica (máster) o más práctica (bootcamp)?"

  - Almacena en shared_memory: {recommendations_presented: true, awaiting_preference: true}

Turno 4:
Usuario: "Me interesa más el máster. ¿Cuánto cuesta y hay opciones online?"

Course Expert Agent:
  - Lee shared_memory: contexto
  - Consulta courses_db: "Máster en Data Science"
  - Filtra por: modalidad=online
  - Retorna detalles:
    "El Máster en Data Science tiene las siguientes opciones:

    Modalidad Online:
    - Duración: 12 meses
    - Coste: 15.000 EUR
    - Horario: Flexible (clases grabadas + sesiones en directo opcionales)
    - Becas disponibles: hasta 30% según perfil

    Modalidad Blended:
    - Duración: 12 meses
    - Coste: 17.000 EUR
    - 70% online + 30% presencial (fines de semana)

    ¿Te gustaría que te ayude con el proceso de solicitud de beca?"

  - Actualiza shared_memory: {interested_in: "master_data_science", prefers_online: true}

Turno 5:
Usuario: "Sí, me interesa la beca. ¿Cómo funciona?"

Student Coach Agent:
  - Detecta intención: quiere avanzar con matrícula
  - Handoff → Enrollment Agent

Enrollment Agent:
  - Lee shared_memory: contexto completo
  - Consulta sistema de becas (API externa)
  - Explica proceso:
    "¡Perfecto! El proceso de solicitud de beca es el siguiente:

    1. Completas formulario online (10 min)
    2. Subes documentación:
       - CV
       - Certificado del grado
       - Carta de motivación (opcional pero recomendada)
    3. Evaluación (2-3 semanas)
    4. Notificación de resultado

    Con tu perfil (grado + experiencia), tienes buenas opciones de conseguir al menos un 20%.

    ¿Quieres que inicie el proceso de solicitud ahora?"

Turno 6:
Usuario: "Sí, adelante"

Enrollment Agent:
  - Ejecuta: start_enrollment(user_id, course_id="master_data_science")
  - Crea lead en CRM
  - Envía email con enlace a formulario
  - Notifica al equipo de admisiones
  - Responde:
    "¡Listo! Te he enviado un email a [email] con el enlace al formulario de solicitud.

    Próximos pasos:
    ✅ Completa el formulario (link en email)
    ✅ Sube documentación requerida
    ✅ Recibirás respuesta en 2-3 semanas

    Si tienes dudas durante el proceso, estoy aquí para ayudarte. ¡Mucha suerte!"

  - Almacena en agent_memory: conversión exitosa
  - Publica evento en Kafka: "lead_converted"
```

### Flujo 3: Agentes en Paralelo para Análisis Comparativo

```
Usuario: "Compara Máster en Data Science vs. Máster en IA"

Agent Orchestrator → Task Planner
  - Detecta: necesita información detallada de 2 cursos + comparación
  - Plan: Parallel execution de 2 instancias de Course Expert Agent + agregación

Ejecución paralela:

→ Course Expert Agent (Instancia 1):
  Tarea: Analizar "Máster en Data Science"
  - Busca en Vector DB
  - Obtiene detalles completos
  - Extrae competencias desarrolladas
  - Retorna: data_science_details

→ Course Expert Agent (Instancia 2):
  Tarea: Analizar "Máster en IA"
  - Busca en Vector DB
  - Obtiene detalles completos
  - Extrae competencias desarrolladas
  - Retorna: ai_details

→ Market Intelligence Agent:
  Tarea: Comparar empleabilidad de ambos perfiles
  - Analiza demanda de "Data Scientist" vs. "AI Engineer"
  - Compara salarios promedio
  - Identifica tendencias de cada campo
  - Retorna: market_comparison

Agent Orchestrator agrega resultados:
  - Combina data_science_details + ai_details + market_comparison
  - Genera tabla comparativa estructurada
  - Retorna al usuario:

"Aquí tienes una comparación detallada:

| Aspecto | Máster Data Science | Máster IA |
|---------|-------------------|-----------|
| Duración | 12 meses | 12 meses |
| Coste | 15.000 EUR | 16.500 EUR |
| Enfoque | Análisis + ML + Business | Deep Learning + NLP + Visión |
| Competencias clave | Python, SQL, ML, Estadística | Python, TensorFlow, PyTorch, DL |
| Perfil ideal | Background business/análisis | Background técnico/matemáticas |
| Demanda laboral | MUY ALTA (8.000 ofertas/mes) | ALTA (3.500 ofertas/mes) |
| Salario promedio | 50-70K EUR | 55-80K EUR |
| Sectores principales | Finance, Consulting, Retail | Tech, Research, Automotive |

Recomendación:
- Si vienes de perfil business/análisis → Data Science
- Si vienes de perfil técnico y te apasiona la investigación → IA

¿Quieres que profundice en algún aspecto?"
```

## Stack Tecnológico Multi-Agente

### AI/ML Frameworks

**Orquestación de Agentes**:
- **LangGraph**: Workflow orchestration, state management
- **CrewAI**: Multi-agent collaboration framework
- **AutoGen**: Microsoft's multi-agent framework (alternativa)

**Agentes y Tools**:
- **LangChain**: Agent framework, tool calling, RAG
- **LlamaIndex**: Advanced RAG, data connectors
- **Semantic Kernel**: Microsoft's SDK for AI orchestration

**LLMs**:
- **Modelos locales** (según requisitos del PDF):
  - **Llama 3 70B** (para agentes principales como Career Advisor)
  - **Mixtral 8x7B** (buen balance performance/coste)
  - **Phi-3 Medium** (para tareas más simples)
- **Embeddings**:
  - `paraphrase-multilingual-mpnet-base-v2` (español + inglés)
  - `text-embedding-3-large` (OpenAI, si se permite)

**Compresión de modelos** (según PDF: "modelos comprimidos para mayor velocidad"):
- **Multiverse Computing**: Quantum-inspired model compression
- **GGUF/llama.cpp**: Quantización de modelos
- **Optimum**: Optimización de Transformers para inference

### Backend

**Lenguaje**: Python 3.11+

**Frameworks**:
- **FastAPI**: APIs de servicios
- **Pydantic**: Validación de datos y schemas

**Orquestación**:
- **Apache Airflow**: Pipelines de datos (scraping, ML training)
- **Celery**: Task queue para procesamiento asíncrono

### Bases de Datos y Almacenamiento

**Relacionales**:
- **PostgreSQL 16 + pgvector**: Agent memory, student/course data

**NoSQL**:
- **MongoDB 7**: Job market data (esquema flexible)

**Vector Databases**:
- **Qdrant**: Vector search (self-hosted, open source)
- **Pinecone**: Alternativa managed (mayor coste)

**Graph Database**:
- **Neo4j**: Knowledge graph de competencias, cursos, trayectorias

**Cache & Memory**:
- **Redis 7**: Shared memory, session management, cache

**Message Broker**:
- **Apache Kafka**: Event streaming entre agentes

### Observabilidad de Agentes

**Agent Tracing**:
- **LangSmith**: Tracing y debugging de LangChain agents
- **Phoenix (Arize AI)**: Observability para LLM apps
- **Weights & Biases**: Tracking de experimentos

**Métricas y Logs**:
- **Prometheus + Grafana**: Métricas de sistema y agentes
- **ELK Stack**: Logs centralizados
- **OpenTelemetry**: Distributed tracing

**Dashboards específicos para monitorizar**:
- Latencia por agente
- Tasa de éxito de recomendaciones
- Costes de inferencia por agente
- Calidad de respuestas (evaluaciones)
- Handoffs entre agentes (flujo)

### Frontend

**Web**: React 18 + TypeScript + Vite
**Mobile**: React Native + Expo
**UI Components**: Shadcn/ui, Tailwind CSS

### Infraestructura

**Cloud**: AWS / GCP / Azure
**Containers**: Docker + Kubernetes
**CI/CD**: GitHub Actions
**IaC**: Terraform

## Consideraciones de Implementación

### Costes de Inferencia

**Modelos locales** (hosted on-premise o cloud VMs):

- **GPU Requirements**:
  - Llama 3 70B: 2x A100 80GB (inferencia) → ~$6/hora AWS
  - Mixtral 8x7B: 1x A100 40GB → ~$3/hora AWS
  - Alternativa: vLLM para maximizar throughput

- **Estimación mensual**:
  - 10,000 conversaciones/mes
  - Promedio 10 interacciones por conversación
  - 100,000 llamadas a LLM/mes
  - Con modelos locales optimizados: **~$2,000-3,000/mes**

**Modelos comerciales** (si se permiten):
- GPT-4 Turbo: $0.01/1K tokens (input), $0.03/1K tokens (output)
- Claude 3.5 Sonnet: Similar pricing
- Para 100K llamadas: **~$8,000-12,000/mes**

**Conclusión**: Modelos locales ahorran 70-75% en costes de inferencia a escala.

### Latencia Multi-Agente

**Latencia por agente** (local inference):
- Simple agent (1 LLM call): 200-500ms
- Complex agent (RAG + 2-3 LLM calls): 1-2s
- Multi-agent sequential (3 agents): 3-6s
- Multi-agent parallel (3 agents): 1.5-2.5s

**Optimizaciones**:
- Paralelización máxima donde sea posible
- Caché agresivo de respuestas similares
- Streaming de respuestas (mostrar pensamiento en tiempo real)
- Modelos comprimidos para agentes secundarios

### Evaluación de Calidad de Agentes

**Métricas por agente**:

1. **Career Advisor Agent**:
   - % recomendaciones que resultaron en matrícula
   - Satisfacción del usuario (survey post-recomendación)
   - Alineamiento con mercado laboral (validación retrospectiva)

2. **Market Intelligence Agent**:
   - Precisión de predicciones (comparar con datos reales 6 meses después)
   - Frescura de datos (% datos <1 mes)
   - Cobertura de fuentes

3. **Student Coach Agent**:
   - % conversaciones que avanzan en el funnel
   - Sentiment analysis de respuestas del usuario
   - Tiempo hasta conversión

4. **Course Expert Agent**:
   - Relevancia de cursos sugeridos (evaluación humana)
   - % consultas respondidas sin escalación

**Framework de evaluación**:
```python
from langsmith import Client
from langsmith.evaluation import evaluate

client = Client()

def evaluate_career_advisor_recommendation(inputs, outputs, reference):
    """Evalúa calidad de recomendación de Career Advisor"""

    # Criterios:
    # 1. Alineamiento con perfil del estudiante
    # 2. Viabilidad de la trayectoria
    # 3. Calidad de la explicación
    # 4. Consideración de restricciones

    scores = {
        "alignment": evaluate_alignment(inputs["profile"], outputs["recommendation"]),
        "feasibility": evaluate_feasibility(outputs["recommendation"]),
        "explanation_quality": evaluate_explanation(outputs["explanation"]),
        "considers_constraints": check_constraints(inputs["constraints"], outputs)
    }

    return {
        "score": sum(scores.values()) / len(scores),
        "details": scores
    }

# Ejecutar evaluación periódica
results = evaluate(
    dataset_name="career_advisor_test_cases",
    evaluator=evaluate_career_advisor_recommendation
)
```

## Fases de Implementación (Multi-Agente)

### Fase 1: MVP Multi-Agente (4-5 meses)

**Mes 1-2: Infraestructura Base**
- Setup de LangGraph + LangChain
- Deployment de modelos locales (Mixtral 8x7B)
- PostgreSQL + pgvector
- Vector DB (Qdrant)
- Redis para shared memory
- LangSmith para observabilidad

**Mes 2-3: Agentes Core (3 agentes)**
- **Student Coach Agent**: Conversación básica, handoffs
- **Course Expert Agent**: RAG sobre catálogo de cursos
- **Career Advisor Agent**: Recomendaciones simples

**Mes 3-4: Orquestación**
- Agent Orchestrator con LangGraph
- Task Planner básico (reglas heurísticas)
- Conversation Manager
- Flujos secuenciales

**Mes 4-5: Integración y Testing**
- Web application básica
- API Gateway
- Integration testing de flujos multi-agente
- Evaluación de calidad

**Entregables Fase 1**:
- 3 agentes operativos
- Recomendaciones básicas de trayectorias
- Conversación multi-turno
- Observabilidad básica

### Fase 2: Agentes Avanzados + Memoria (3 meses)

**Mes 6-7: Agentes Especializados**
- **Market Intelligence Agent**: Integración Job Market DB
- **Skills Assessment Agent**: Evaluación de competencias
- **Research Agent**: Web search integration

**Mes 7-8: Sistema de Memoria**
- Agent Memory Store (episodic + semantic)
- Knowledge Graph (Neo4j)
- Shared memory optimization
- Memoria a largo plazo

**Mes 8: Orquestación Avanzada**
- Ejecución paralela de agentes
- Task Planner inteligente (LLM-based)
- Multi-agent collaboration patterns

### Fase 3: Optimización y Escala (2-3 meses)

**Mes 9: Performance**
- Compresión de modelos (Multiverse Computing)
- Caché inteligente
- Streaming de respuestas
- Load testing

**Mes 10: Enrollment + Integraciones**
- **Enrollment Agent**
- Integración con Sistema de Matrícula
- Integración con CRM
- Mobile app

**Mes 11: Analytics y Mejora Continua**
- Dashboards de métricas de agentes
- A/B testing framework
- Feedback loop para reentrenamiento
- Evaluación automática de calidad

**Tiempo total**: 11-12 meses

## Estimación de Recursos (Multi-Agente)

### Infraestructura Cloud (AWS)

**Compute (Agentes + LLMs)**:
- 2x g5.4xlarge (Mixtral 8x7B inference): ~$2,500/mes
- 1x g5.2xlarge (Embeddings): ~$800/mes
- 6x t3.large (Services): ~$500/mes

**Databases**:
- RDS PostgreSQL (db.r6g.xlarge): ~$400/mes
- DocumentDB (db.r6g.large): ~$300/mes
- ElastiCache Redis (cache.r6g.large): ~$200/mes
- Neo4j (self-hosted en EC2 m5.xlarge): ~$150/mes

**Vector DB**:
- Qdrant (self-hosted en c6g.2xlarge): ~$250/mes

**Message Broker**:
- MSK (Kafka managed): ~$300/mes

**Storage**:
- S3: ~$100/mes
- EBS: ~$200/mes

**Observabilidad**:
- CloudWatch: ~$150/mes
- LangSmith: ~$200/mes (plan team)

**Total estimado**: **$5,500 - $6,500/mes**

(30-40% más que arquitectura sin agentes, pero con mayor autonomía y calidad)

### Equipo de Desarrollo

**Fase 1 (MVP - 5 meses)**:
- 1 Tech Lead (AI/ML)
- 2 Backend Engineers (Python, LangChain)
- 1 ML Engineer (LLMs, RAG)
- 1 Frontend Engineer (React)
- 1 DevOps Engineer

**Fase 2-3 (7 meses)**:
- +1 AI Engineer (Agent optimization)
- +1 Data Engineer (pipelines, Knowledge Graph)

## Ventajas de Arquitectura Multi-Agente vs. Tradicional

| Aspecto | Arquitectura Tradicional | Arquitectura Multi-Agente |
|---------|------------------------|--------------------------|
| **Mantenibilidad** | Monolítico, difícil de extender | Modular, fácil agregar/mejorar agentes |
| **Especialización** | Servicio genérico | Agentes expertos por dominio |
| **Escalabilidad** | Escalar todo el servicio | Escalar solo agentes bajo demanda |
| **Observabilidad** | Caja negra | Trazabilidad completa de razonamiento |
| **Calidad** | Depende de un solo modelo | Mejora continua por agente |
| **Flexibilidad** | Flujo fijo | Flujos dinámicos adaptables |
| **Costo inicial** | Menor | Mayor (más complejo) |
| **Costo a escala** | Crece linealmente | Optimizable por agente |
| **Time to market** | Más rápido (3-4 meses) | Más lento (4-5 meses MVP) |
| **Autonomía** | Requiere intervención para nuevos casos | Agentes se adaptan automáticamente |

## Próximos Pasos

1. **Validación de arquitectura** con stakeholders técnicos y de negocio
2. **PoC de agente individual** (Career Advisor Agent con LangChain)
3. **Benchmark de modelos locales** (Llama 3 vs. Mixtral vs. Phi-3)
4. **Diseño de prompts de sistema** por agente
5. **Definición de métricas de éxito** por agente
6. **Setup de LangSmith** para observabilidad temprana
7. **Prototipo de Knowledge Graph** (Neo4j con dataset sintético)
