# Propuesta de Arquitectura Multi-Agente IA
## Recomendador de Trayectorias - Universidad Europea

**Consultoría**: Inorganic
**Cliente**: Universidad Europea
**Fecha**: Enero 2026

---

## Resumen Ejecutivo

Propuesta de solución basada en **arquitectura multi-agente de IA** para el caso de uso "Recomendador de Trayectorias Académico-Profesionales". Esta arquitectura permite crear un sistema inteligente, modular y escalable que combine:

- Análisis del perfil del estudiante
- Inteligencia de mercado laboral en tiempo real
- Conocimiento profundo del catálogo formativo
- Acompañamiento personalizado

---

## Arquitectura Propuesta

### Visión General

El sistema se estructura en **5 capas principales**:

1. **Canales de Interacción**: Web, Mobile, WhatsApp
2. **Orquestador de Agentes**: Coordinación inteligente
3. **Agentes IA Especializados**: 5 agentes con dominios específicos
4. **Recursos y Herramientas**: Base de conocimiento y funcionalidades
5. **Fuentes de Datos**: Sistemas de la universidad y mercado laboral

---

## Capa 1: Canales de Interacción

### Descripción
Interfaces multicanal que permiten a usuarios interactuar con el sistema de recomendaciones.

### Componentes
- **Web Application**: Portal principal con dashboards interactivos
- **Mobile App**: Aplicación iOS/Android para acceso móvil
- **WhatsApp/Telegram**: Canal conversacional para consultas rápidas

### Tecnología
- React + TypeScript (Web)
- React Native (Mobile)
- API de WhatsApp Business

---

## Capa 2: Orquestador de Agentes IA

### Descripción
Cerebro del sistema que coordina agentes especializados según el contexto y objetivo del usuario.

### Responsabilidades
- Recibir y analizar consultas de usuarios
- Determinar qué agentes deben participar
- Coordinar ejecución (secuencial o paralela)
- Agregar resultados de múltiples agentes
- Retornar respuesta unificada

### Tecnología
- **LangGraph**: Orquestación de workflows de agentes
- **CrewAI**: Framework multi-agente colaborativo
- **Python**: Lenguaje base

### Ejemplo de Flujo
```
Usuario: "Quiero ser Data Scientist"
  ↓
Orquestador analiza → Selecciona agentes:
  1. Skills Assessment Agent (evaluar nivel actual)
  2. Market Intelligence Agent (analizar demanda)
  3. Career Advisor Agent (diseñar trayectoria)
  ↓
Coordina ejecución → Agrega resultados → Responde
```

---

## Capa 3: Agentes IA Especializados

### 1. Career Advisor Agent (Asesor de Carrera)

**Dominio**: Orientación profesional y diseño de trayectorias

**Capacidades**:
- Analiza perfil del estudiante (formación, experiencia, objetivos)
- Diseña trayectorias multi-nivel (Grado → Máster → Certificaciones)
- Evalúa alineamiento con mercado laboral
- Calcula ROI de inversiones formativas
- Explica razonamiento de cada recomendación

**Tecnología**: LLM + RAG (Retrieval Augmented Generation)

**Ejemplo**:
> "Basándome en tu perfil (Grado en Economía + experiencia en análisis), te recomiendo el Máster en Data Science. Esta trayectoria aprovecha tu background cuantitativo y tiene una demanda laboral muy alta (+40% ofertas YoY) con salarios de 50-70K EUR."

---

### 2. Market Intelligence Agent (Inteligencia de Mercado)

**Dominio**: Análisis del mercado laboral

**Capacidades**:
- Monitoriza ofertas de empleo en tiempo real (LinkedIn, InfoJobs)
- Identifica competencias más demandadas
- Analiza tendencias salariales por rol y ubicación
- Detecta sectores en crecimiento
- Predice necesidades futuras del mercado

**Tecnología**: LLM + Analytics + Web Scraping

**Fuentes de datos**:
- LinkedIn API: Ofertas internacionales
- InfoJobs API: Mercado español
- Web Search: Informes de tendencias

**Ejemplo de insight**:
> "Cloud Architect: Demanda +40% YoY | 3,500 ofertas/mes en España | Salario 55-85K EUR | Top skills: Kubernetes, Terraform, AWS"

---

### 3. Course Expert Agent (Experto en Catálogo)

**Dominio**: Conocimiento del catálogo formativo de la Universidad

**Capacidades**:
- Conoce todos los cursos, másteres y certificaciones
- Responde dudas sobre contenidos, modalidades, requisitos
- Sugiere formaciones complementarias
- Identifica prerequisitos y dependencias
- Explica competencias desarrolladas por cada formación

**Tecnología**: LLM + RAG sobre catálogo completo

**Base de conocimiento**:
- Syllabi de todos los programas
- Guías docentes
- Requisitos de admisión
- Modalidades y precios

**Ejemplo**:
> "El Máster en IA tiene enfoque técnico en Deep Learning y NLP. Requisitos: Grado técnico o científico. Duración: 12 meses. Modalidad: Online o Blended. Coste: 16,500 EUR (becas hasta 30%)."

---

### 4. Student Coach Agent (Coach del Estudiante)

**Dominio**: Acompañamiento personalizado

**Capacidades**:
- Guía conversacional empática y personalizada
- Hace preguntas para clarificar objetivos y restricciones
- Ayuda a gestionar dudas y miedos
- Recuerda interacciones previas y contexto personal
- Adapta comunicación al perfil del estudiante

**Tecnología**: LLM + Memoria episódica

**Memoria**:
- Historial completo de conversaciones
- Preferencias expresadas
- Fase actual en proceso de decisión

**Ejemplo**:
> "Entiendo tu preocupación sobre el aspecto económico. Muchos estudiantes están en tu situación. ¿Has explorado opciones como becas o programas part-time que te permitan trabajar mientras estudias?"

---

### 5. Skills Assessment Agent (Evaluador de Competencias)

**Dominio**: Evaluación técnica de competencias

**Capacidades**:
- Evalúa competencias actuales del estudiante
- Identifica gaps respecto a roles profesionales objetivo
- Mapea experiencia laboral/académica a competencias
- Sugiere áreas de mejora prioritarias
- Valida nivel de dominio

**Tecnología**: LLM + ML (modelos de NLP para extracción de competencias)

**Ejemplo de análisis**:
```
Perfil actual:
✓ Python: Intermedio
✓ SQL: Avanzado
✓ Excel: Avanzado

Gap para Data Scientist:
✗ Machine Learning: No tiene
✗ Deep Learning: No tiene
✗ Big Data (Spark): No tiene

Recomendación: Priorizar formación en ML/DL
```

---

## Capa 4: Recursos y Herramientas

### Knowledge Graph (Grafo de Conocimiento)

**Componente**: Vector Database (Qdrant) + Graph Database (Neo4j)

**Descripción**:
Sistema híbrido que combina búsqueda semántica vectorial con relaciones estructuradas en grafo, permitiendo tanto similitud semántica como navegación por relaciones complejas.

**Contenido del Vector DB (Qdrant)**:
- **Embeddings de cursos**: Representaciones vectoriales de programas formativos
- **Embeddings de competencias**: Skills y conocimientos vectorizados
- **Embeddings de ofertas de empleo**: Requisitos laborales vectorizados
- **Embeddings de trayectorias**: Perfiles de estudiantes exitosos
- **FAQs y documentación**: Contenido vectorizado para RAG

**Estructura del Graph DB (Neo4j)**:
```cypher
// Nodos principales
(Student:Persona)-[:HAS_SKILL]->(Skill:Competencia)
(Student)-[:INTERESTED_IN]->(Role:RolProfesional)
(Course:Curso)-[:TEACHES]->(Skill)
(Course)-[:LEADS_TO]->(Role)
(Role)-[:REQUIRES {level: "advanced"}]->(Skill)
(Course)-[:PREREQUISITE]->(Course)
(Trajectory:Trayectoria)-[:INCLUDES {order: 1}]->(Course)
(JobOffer:Oferta)-[:REQUIRES]->(Skill)
```

**Queries clave**:

1. **Encontrar camino óptimo de estudiante a rol objetivo**:
```cypher
MATCH path = shortestPath(
  (s:Student {id: $student_id})-[*]-(r:Role {name: $target_role})
)
RETURN path
```

2. **Identificar gaps de competencias**:
```cypher
MATCH (s:Student {id: $student_id}),
      (r:Role {name: $target_role})-[:REQUIRES]->(required:Skill)
WHERE NOT (s)-[:HAS_SKILL]->(required)
RETURN required.name, required.importance
ORDER BY required.importance DESC
```

3. **Recomendar siguiente curso en trayectoria**:
```cypher
MATCH (s:Student {id: $student_id}),
      (s)-[:HAS_SKILL]->(skill:Skill),
      (next:Course)-[:TEACHES]->(new_skill:Skill),
      (next)-[:LEADS_TO]->(s.target_role)
WHERE NOT (s)-[:COMPLETED]->(next)
  AND ALL(prereq IN [(next)-[:PREREQUISITE]->(p:Course) | p]
      WHERE (s)-[:COMPLETED]->(prereq))
WITH next, COUNT(DISTINCT skill) as skills_match
RETURN next
ORDER BY skills_match DESC
LIMIT 3
```

**Ventajas del enfoque híbrido**:
- ✅ **Vector DB**: Búsqueda por similitud semántica ("cursos parecidos a...")
- ✅ **Graph DB**: Navegación por relaciones ("¿qué necesito para llegar a X?")
- ✅ **Combinados**: "Cursos similares semánticamente que además cumplan prerequisitos"

**Tecnología**: Qdrant (Vector DB) + Neo4j (Knowledge Graph)

---

### Herramientas de Agentes

**Funcionalidades especializadas** que los agentes pueden usar:

1. **RAG Engine**: Consultas semánticas sobre documentación
2. **Web Search Tool**: Información actualizada en tiempo real
3. **Calculator & Analytics**: Cálculos de ROI, tendencias, estadísticas
4. **Function Registry**: APIs de servicios internos (matrícula, CRM)

**Tecnología**: LangChain Tools, Tavily API (web search)

---

### Memoria Compartida

**Componente**: Redis + PostgreSQL

**Propósito**: Coordinación entre agentes y contexto conversacional

**Usos**:
- Sesiones de conversación multi-turno
- Handoffs entre agentes (pasar contexto)
- Compartir insights descubiertos
- Historial de recomendaciones

---

## Capa 5: Fuentes de Datos

### Sistemas de la Universidad

**Integraciones**:
- **CRM**: Datos de leads y estudiantes potenciales
- **LMS**: Historial académico y rendimiento
- **Sistema de Matrícula**: Proceso de inscripción

**Uso**: Sincronización de datos de estudiantes y cursos

---

### Mercado Laboral

**Fuentes externas**:
- **LinkedIn API**: Ofertas de empleo globales
- **InfoJobs API**: Mercado laboral español
- **Web Scraping**: Otras fuentes (Indeed, Glassdoor)

**Frecuencia**: Actualización diaria de tendencias

---

## Flujos de Interacción

### Flujo 1: Recomendación Completa

```
1. Usuario: "Quiero orientación para ser Data Scientist"
   ↓
2. Orquestador → Student Coach Agent
   - Hace preguntas para clarificar (background, restricciones)
   ↓
3. Usuario: "Tengo Grado en Economía, experiencia 1 año analista"
   ↓
4. Orquestador → Skills Assessment Agent
   - Evalúa: Excel ✓, SQL ✓, Python básico, ML ✗
   ↓
5. Orquestador → Ejecución Paralela:
   - Market Intelligence Agent: Analiza demanda de Data Scientists
   - Course Expert Agent: Busca cursos compatibles con perfil
   ↓
6. Orquestador → Career Advisor Agent
   - Recibe: skills_assessment + market_analysis + available_courses
   - Diseña 3 trayectorias alternativas (conservadora, equilibrada, ambiciosa)
   - Calcula ROI de cada una
   ↓
7. Retorna al usuario:
   "Basándome en tu perfil, te recomiendo 3 trayectorias:

   [Trayectoria 1 - Bootcamp] ...
   [Trayectoria 2 - Máster] ← RECOMENDADA
   [Trayectoria 3 - Máster + Especialización] ...

   Perspectivas: Demanda ALTA, Salario 50-70K EUR, ROI 300% en 4 años"
```

---

### Flujo 2: Conversación Multi-Turno

```
Turno 1: Usuario → "Estoy pensando en algo con datos"
         ↓ Student Coach Agent: Hace preguntas clarificadoras

Turno 2: Usuario → "Me interesa análisis y sacar insights"
         ↓ Student Coach → Skills Assessment: Evalúa nivel técnico

Turno 3: Usuario → "Uso SQL y Excel, Python básico"
         ↓ Skills Assessment → Career Advisor: Genera recomendación

Turno 4: Usuario → "Me interesa el Máster. ¿Coste?"
         ↓ Course Expert: Detalles del Máster

Turno 5: Usuario → "Quiero solicitar beca"
         ↓ Enrollment Agent: Inicia proceso de matrícula
```

---

## Capacidades Diferenciales

### vs. Arquitectura Tradicional

| Aspecto | Tradicional | Multi-Agente |
|---------|-------------|--------------|
| **Modularidad** | Baja | Alta (agentes independientes) |
| **Especialización** | Genérica | Expertos por dominio |
| **Adaptabilidad** | Requiere código | Agentes se adaptan |
| **Observabilidad** | Limitada | Razonamiento transparente |
| **Escalabilidad** | Todo junto | Por agente individual |
| **Autonomía** | Baja | Alta (agentes deciden) |

---

### Ventajas Clave

1. **Expertise Especializado**: Cada agente es experto en su dominio
2. **Razonamiento Explicable**: Se puede auditar decisión de cada agente
3. **Mejora Continua**: Agentes aprenden y mejoran independientemente
4. **Flexibilidad**: Fácil agregar nuevos agentes sin cambiar el core
5. **Escalabilidad**: Escalar solo agentes bajo alta demanda
6. **Colaboración Inteligente**: Agentes comparten conocimiento

---

## Sistemas Externos

### Google Gemini

**Rol**: LLM externo de última generación para potenciar capacidades avanzadas de los agentes.

**Uso por agente**:

1. **Career Advisor Agent**:
   - Razonamiento complejo para diseño de trayectorias multi-nivel
   - Análisis de trade-offs entre opciones
   - Generación de explicaciones detalladas

2. **Market Intelligence Agent**:
   - Análisis predictivo de tendencias laborales
   - Correlación de múltiples señales del mercado
   - Identificación de competencias emergentes

3. **Student Coach Agent**:
   - Generación de respuestas empáticas y contextualizadas
   - Adaptación de tono según perfil del estudiante
   - Detección de preocupaciones no explícitas

4. **Skills Assessment Agent**:
   - Evaluación matizada de nivel de competencias
   - Análisis de experiencia laboral compleja
   - Extracción de skills desde descripciones narrativas

5. **Course Expert Agent**:
   - Comprensión profunda de syllabi y programas
   - Comparación semántica de contenidos
   - Generación de summaries de cursos

**Ventajas de Gemini**:
- ✅ Razonamiento avanzado multimodal
- ✅ Contexto largo (2M tokens) para analizar múltiples documentos
- ✅ Latencia baja y alta calidad de respuestas
- ✅ Multilenguaje nativo (español/inglés)

**Estrategia híbrida**:
- **Modelos locales**: Tareas simples, privacidad crítica, alto volumen
- **Gemini**: Razonamiento complejo, análisis profundo, decisiones críticas

---

## Stack Tecnológico

### AI/ML
- **Orquestación**: LangGraph, Temporal.io, CrewAI
- **LLMs Externos**: Google Gemini Pro/Ultra
- **LLMs Locales**: Llama 3, Mixtral (para privacidad y volumen)
- **Embeddings**: Sentence Transformers (multilenguaje)
- **Frameworks**: LangChain, LlamaIndex

### Backend
- **Lenguaje**: Python 3.11+
- **APIs**: FastAPI
- **Procesamiento**: Celery, Apache Airflow

### Datos
- **Relacional**: PostgreSQL 16 + pgvector
- **NoSQL**: MongoDB (ofertas de empleo)
- **Vector DB**: Qdrant
- **Graph DB**: Neo4j
- **Cache**: Redis

### Frontend
- **Web**: React + TypeScript
- **Mobile**: React Native

### Infraestructura
- **Cloud**: AWS / Azure / GCP
- **Containers**: Docker + Kubernetes
- **Observabilidad**: LangSmith, Prometheus, Grafana

---

## Estimaciones

### Tiempo de Implementación

**Fase 1 - MVP (4-5 meses)**:
- 3 agentes core (Coach, Course Expert, Career Advisor)
- Orquestador básico
- Web application
- Recomendaciones simples

**Fase 2 - Agentes Avanzados (3 meses)**:
- Market Intelligence Agent
- Skills Assessment Agent
- Sistema de memoria
- Integraciones externas

**Fase 3 - Optimización (3 meses)**:
- Mobile app
- Compresión de modelos
- Analytics avanzado
- A/B testing

**Total**: 10-11 meses

---

### Costes Estimados (Mensual)

**Infraestructura Cloud**:
- Compute (LLMs + Services): $3,800/mes
- Databases: $1,050/mes
- Storage: $300/mes
- Observabilidad: $350/mes

**Total**: **~$5,500/mes**

(30-40% más que arquitectura tradicional, justificado por mayor autonomía y calidad)

---

## Propuesta de Valor Inorganic

### Expertise en IA Generativa
- Experiencia en implementación de sistemas multi-agente
- Conocimiento de LangChain, LangGraph, CrewAI
- Desarrollo con LLMs locales y comerciales

### Consultoría End-to-End
- Diseño de arquitectura
- Implementación técnica
- Despliegue en cloud
- Soporte y evolución

### Enfoque en ROI
- Modelos locales para reducir costes de inferencia (70-75% ahorro)
- Compresión de modelos para mayor velocidad
- Escalabilidad optimizada por agente

---

## Próximos Pasos

1. **Workshop de validación** (2h)
   - Presentación de arquitectura
   - Demo de PoC de agente individual
   - Q&A técnico

2. **PoC - Career Advisor Agent** (4 semanas)
   - Implementación de 1 agente funcional
   - Integración con datos reales de Universidad
   - Evaluación de calidad de recomendaciones

3. **Roadmap detallado** (1 semana)
   - Planificación de sprints
   - Definición de hitos
   - Estimación refinada

4. **Kick-off del proyecto** (si aprobado)

---

## Contacto

**Inorganic**
alexia@inorganic.com | mikel@inorganic.com

**Propuesta para**: Universidad Europea
**Fecha**: Enero 2026
**Versión**: 1.0
