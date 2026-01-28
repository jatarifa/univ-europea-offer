# Arquitectura del Recomendador de Trayectorias Académico-Profesionales

## Visión General

Sistema inteligente que combina el análisis del perfil del estudiante con tendencias del mercado laboral para recomendar trayectorias formativas personalizadas (Grado → Máster → Certificaciones).

## Componentes Principales

### 1. Capa de Presentación

#### Web Application (React + TypeScript)
- Interfaz principal para estudiantes y futuros alumnos
- Visualización de trayectorias recomendadas con gráficos interactivos
- Dashboard personalizado con KPIs del mercado laboral
- Comparador de trayectorias alternativas

#### Mobile App (React Native)
- Acceso móvil a recomendaciones
- Notificaciones push sobre nuevas formaciones relevantes
- Modo offline para consultar recomendaciones guardadas

#### Chatbot Interface (WhatsApp/Telegram)
- Canal conversacional natural
- Integrado con Conversational AI Service
- Permite consultas rápidas y orientación inicial

### 2. Capa de API

#### API Gateway (Kong / AWS API Gateway)
- **Autenticación y autorización**: OAuth 2.0 / JWT
- **Rate limiting**: Prevención de abuso
- **Routing**: Enrutamiento inteligente a microservicios
- **Logging y monitorización**: Centralización de logs
- **Transformación de requests/responses**

### 3. Capa de Servicios de Negocio

#### Recommendation Engine (Python + FastAPI)
**Responsabilidad principal**: Motor de recomendaciones

**Funcionalidades**:
- Análisis del perfil del estudiante (competencias actuales, intereses, background)
- Matching con requisitos del mercado laboral
- Generación de trayectorias personalizadas multi-nivel (Grado → Máster → Certificación)
- Scoring de compatibilidad entre perfil y trayectoria
- Explicabilidad de recomendaciones (por qué se sugiere cada trayectoria)

**Algoritmos**:
- Collaborative Filtering para encontrar perfiles similares
- Content-Based Filtering basado en competencias
- Hybrid Recommender combinando múltiples señales
- Búsqueda semántica en Vector Database

#### Conversational AI Service (Python + LangChain)
**Responsabilidad**: Asistente conversacional inteligente

**Funcionalidades**:
- Guía al usuario en la exploración de trayectorias
- Resuelve dudas sobre formaciones específicas
- Propone formaciones complementarias
- Puede iniciar proceso de matrícula
- Contexto conversacional persistente

**Tecnología**:
- LangChain para orquestación de LLMs
- Modelos locales (según requisitos de privacidad del PDF)
- RAG (Retrieval Augmented Generation) con Vector Database
- Function calling para integración con otros servicios

#### Job Market Analyzer (Python + Scrapy + Airflow)
**Responsabilidad**: Extracción y análisis de ofertas de empleo

**Funcionalidades**:
- Scraping periódico de LinkedIn, InfoJobs y otras fuentes
- Extracción de competencias requeridas usando NLP
- Análisis de tendencias (competencias emergentes, salarios, ubicaciones)
- Detección de cambios en demanda del mercado
- Generación de insights sobre sectores en crecimiento

**Pipeline**:
```
LinkedIn/InfoJobs → Extracción → Limpieza → NLP (Competencias)
  → Análisis de Tendencias → Almacenamiento → Indexación Vectorial
```

**Tecnología**:
- Airflow para orquestación de jobs periódicos (diarios/semanales)
- Scrapy para web scraping robusto
- spaCy/BERT para extracción de entidades y competencias
- Pandas para análisis de datos

#### Student Profile Service (Python + FastAPI)
**Responsabilidad**: Gestión de perfiles de estudiantes

**Funcionalidades**:
- CRUD de perfiles de estudiantes
- Gestión de competencias actuales y objetivos profesionales
- Historial de formaciones completadas
- Preferencias de aprendizaje (modalidad, horarios, presupuesto)
- Integración con LMS para obtener rendimiento académico
- Sincronización con CRM

#### Course Catalog Service (Python + FastAPI)
**Responsabilidad**: Gestión del catálogo formativo

**Funcionalidades**:
- Catálogo de Grados, Másteres, Certificaciones
- Metadata de cada formación (competencias desarrolladas, duración, precio, modalidad)
- Requisitos previos y dependencias entre formaciones
- Información de empleabilidad por formación
- Indexación en Vector Database para búsqueda semántica

### 4. Capa de Machine Learning

#### ML Training Pipeline (Python + MLflow + Scikit-learn)
**Responsabilidad**: Entrenamiento y versionado de modelos

**Modelos**:
1. **Modelo de Matching**: Predice compatibilidad perfil-trayectoria
2. **Modelo de Empleabilidad**: Predice probabilidad de empleo por trayectoria
3. **Modelo de Extracción de Competencias**: Extrae competencias de ofertas de empleo
4. **Embedding Models**: Genera embeddings de perfiles, cursos y ofertas

**Tecnología**:
- MLflow para tracking de experimentos y model registry
- Scikit-learn para modelos tradicionales de ML
- Transformers (BERT/RoBERTa) para modelos de NLP
- Reentrenamiento periódico basado en eventos

#### Model Serving (TensorFlow Serving / TorchServe)
**Responsabilidad**: Servir modelos en producción

**Características**:
- Inferencia de baja latencia (<100ms)
- Versionado de modelos
- A/B testing de modelos
- Monitorización de drift
- Escalado horizontal

### 5. Capa de Datos

#### Student Database (PostgreSQL)
**Contenido**:
- Perfiles de estudiantes
- Competencias y objetivos profesionales
- Historial de recomendaciones
- Interacciones con el sistema

**Esquema principal**:
```sql
students (id, name, email, profile_data, created_at, updated_at)
student_competencies (student_id, competency_id, level, verified)
student_goals (student_id, goal_type, target_role, target_salary)
student_interactions (student_id, recommendation_id, action, timestamp)
```

#### Courses Database (PostgreSQL)
**Contenido**:
- Catálogo de formaciones
- Competencias desarrolladas por cada formación
- Requisitos y dependencias
- Metadata (precio, duración, modalidad)

**Esquema principal**:
```sql
courses (id, name, type, level, duration, price, modality)
course_competencies (course_id, competency_id, proficiency_level)
course_prerequisites (course_id, prerequisite_course_id)
competencies (id, name, category, description)
```

#### Job Market Database (MongoDB)
**Contenido**:
- Ofertas de empleo extraídas
- Tendencias del mercado laboral
- Análisis de competencias demandadas
- Información salarial por rol

**Estructura de documentos**:
```json
{
  "job_id": "...",
  "title": "Data Scientist",
  "company": "...",
  "location": "Madrid",
  "salary_range": {"min": 40000, "max": 60000},
  "competencies": [
    {"name": "Python", "importance": "required"},
    {"name": "Machine Learning", "importance": "required"}
  ],
  "extracted_at": "2026-01-28",
  "source": "linkedin"
}
```

**Justificación MongoDB**: Esquema flexible para diferentes formatos de ofertas de empleo de múltiples fuentes.

#### Vector Database (Pinecone / Qdrant)
**Contenido**:
- Embeddings de perfiles de estudiantes
- Embeddings de formaciones
- Embeddings de ofertas de empleo
- Embeddings de competencias

**Uso**:
- Búsqueda semántica de trayectorias similares
- Matching perfil-oferta-formación
- RAG para el chatbot conversacional

#### Cache (Redis)
**Contenido**:
- Recomendaciones recientes (TTL: 24h)
- Perfiles de usuario activos (TTL: 1h)
- Tendencias del mercado agregadas (TTL: 7 días)
- Sesiones de chatbot (TTL: 30 min)

**Estrategia**: Cache-aside pattern

#### Event Bus (Apache Kafka / RabbitMQ)
**Eventos**:
- `job_market.offers.new`: Nueva oferta detectada
- `student.profile.updated`: Perfil de estudiante actualizado
- `recommendation.generated`: Nueva recomendación generada
- `ml.model.updated`: Modelo ML actualizado
- `course.catalog.updated`: Catálogo de cursos actualizado

**Uso**:
- Procesamiento asíncrono
- Desacoplamiento de servicios
- Trigger para reentrenamiento de modelos

### 6. Sistemas Externos

#### LinkedIn API
- Extracción de ofertas de empleo
- Análisis de competencias demandadas
- Rate limiting: Respetar límites de la API

#### InfoJobs API
- Ofertas de empleo del mercado español
- Información salarial
- Tendencias por sector

#### Sistema de Matrícula (Universidad Europea)
- Inicio de proceso de matrícula desde recomendación
- Sincronización de estudiantes matriculados
- API REST existente de la Universidad

#### CRM Universidad
- Captura de leads desde recomendaciones
- Seguimiento de conversión
- Segmentación de audiencias

#### LMS Universidad
- Historial académico de estudiantes
- Rendimiento en asignaturas
- Competencias verificadas

## Flujos Principales

### Flujo 1: Generación de Recomendación

```
1. Usuario solicita recomendación (Web/App/Chatbot)
2. API Gateway autentica y enruta a Recommendation Engine
3. Recommendation Engine:
   a. Obtiene perfil del estudiante (Profile Service → PostgreSQL)
   b. Busca perfiles similares (Vector DB - búsqueda semántica)
   c. Consulta tendencias laborales (Job Market DB)
   d. Consulta catálogo de cursos (Courses DB)
   e. Ejecuta modelo de matching (Model Serving)
   f. Genera trayectorias candidatas
   g. Scoring y ranking de trayectorias
   h. Almacena en caché (Redis)
4. Retorna recomendaciones al usuario con explicación
```

### Flujo 2: Actualización de Tendencias Laborales

```
1. Airflow dispara job diario (3:00 AM)
2. Job Market Analyzer:
   a. Scraping de LinkedIn API
   b. Scraping de InfoJobs API
   c. Limpieza y normalización de datos
   d. Extracción de competencias con NLP
   e. Análisis de tendencias
   f. Almacenamiento en MongoDB
   g. Indexación en Vector DB
   h. Publicación de evento en Kafka
3. Kafka notifica a Recommendation Service
4. Recommendation Service invalida caché relevante
5. ML Training Pipeline evalúa necesidad de reentrenamiento
```

### Flujo 3: Conversación con Chatbot

```
1. Usuario envía mensaje por WhatsApp
2. Chatbot Interface recibe mensaje
3. API Gateway enruta a Conversational AI Service
4. AI Service:
   a. Recupera contexto de conversación (Redis)
   b. Procesa mensaje con LLM local
   c. Ejecuta RAG sobre Vector DB (documentación de cursos)
   d. Si necesita recomendaciones, llama a Recommendation Engine
   e. Si usuario quiere matricularse, llama a Sistema de Matrícula
   f. Genera respuesta
   g. Actualiza contexto conversacional
5. Retorna respuesta al usuario
```

### Flujo 4: Reentrenamiento de Modelos

```
1. Evento en Kafka dispara reentrenamiento (semanal o threshold de nuevos datos)
2. ML Training Pipeline:
   a. Extrae datos de Student DB, Courses DB, Job Market DB
   b. Preprocesamiento y feature engineering
   c. División train/validation/test
   d. Entrenamiento de modelo
   e. Evaluación de métricas
   f. Registro en MLflow
   g. Si mejora métricas, despliega a Model Serving
   h. A/B testing con modelo anterior
3. Model Serving sirve nuevo modelo
4. Monitorización de drift y performance
```

## Consideraciones de Diseño

### Escalabilidad
- **Servicios stateless**: Permiten escalado horizontal
- **Caché Redis**: Reduce carga en bases de datos
- **Event-driven architecture**: Procesamiento asíncrono
- **Vector DB**: Búsquedas semánticas eficientes a escala
- **CDN**: Para contenido estático (web/mobile)

### Seguridad
- **API Gateway**: Autenticación centralizada (OAuth 2.0 / JWT)
- **HTTPS**: Todo el tráfico cifrado
- **Rate limiting**: Prevención de abuso
- **Sanitización de inputs**: Prevención de inyecciones
- **GDPR compliance**: Consentimiento para datos personales
- **Modelos locales**: Privacidad de datos según requisitos del PDF

### Resiliencia
- **Circuit breakers**: Prevención de cascading failures
- **Retries con backoff**: Manejo de fallos transitorios
- **Health checks**: Monitorización proactiva
- **Fallbacks**: Recomendaciones básicas si falla ML
- **Message queues**: Garantía de procesamiento de eventos

### Observabilidad
- **Logging centralizado**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Métricas**: Prometheus + Grafana
- **Tracing distribuido**: Jaeger / OpenTelemetry
- **Alerting**: PagerDuty / Slack notifications
- **Model monitoring**: MLflow + custom dashboards

### Performance
- **Targets**:
  - Latencia API < 200ms (p95)
  - Latencia recomendaciones < 500ms (p95)
  - Latencia chatbot < 1s (p95)
  - Throughput: 1000 req/s
- **Optimizaciones**:
  - Caché Redis agresivo
  - Compresión de responses (gzip)
  - Database indexing optimizado
  - Connection pooling
  - Modelos comprimidos (según PDF: mejor velocidad)

## Stack Tecnológico Recomendado

### Backend
- **Lenguaje**: Python 3.11+
- **Frameworks**: FastAPI (APIs), LangChain (AI), Scrapy (scraping)
- **Orquestación**: Airflow
- **Message Broker**: Apache Kafka

### Frontend
- **Web**: React 18 + TypeScript + Vite
- **Mobile**: React Native + Expo
- **UI Library**: Material-UI / Chakra UI

### Datos
- **RDBMS**: PostgreSQL 15
- **NoSQL**: MongoDB 7
- **Vector DB**: Qdrant (self-hosted) o Pinecone (managed)
- **Caché**: Redis 7
- **Message Queue**: Kafka 3.x

### Machine Learning
- **Training**: Scikit-learn, XGBoost, PyTorch, Transformers
- **NLP**: spaCy, BERT, LLMs locales
- **Serving**: TorchServe o FastAPI custom
- **MLOps**: MLflow, DVC

### Infraestructura
- **Cloud**: AWS / Azure / GCP
- **Containers**: Docker + Kubernetes
- **CI/CD**: GitHub Actions / GitLab CI
- **IaC**: Terraform
- **Monitoring**: Prometheus + Grafana + ELK

## Estimación de Recursos

### Infraestructura Cloud (AWS)

**Compute**:
- API Gateway: Serverless (AWS API Gateway)
- Services: 6x EC2 t3.large (8 vCPU total, 16 GB RAM)
- ML Training: p3.2xlarge (on-demand, 2-4h/week)
- Model Serving: 2x g4dn.xlarge (GPU inference)

**Databases**:
- RDS PostgreSQL: db.r5.xlarge (4 vCPU, 32 GB RAM)
- DocumentDB (MongoDB): db.r5.large (2 vCPU, 16 GB RAM)
- ElastiCache Redis: cache.r5.large (2 vCPU, 13 GB RAM)
- Qdrant: 2x t3.medium (self-hosted en K8s)

**Storage**:
- S3: 500 GB (modelos, datasets, backups)
- EBS: 2 TB (databases)

**Networking**:
- Data transfer: ~2 TB/mes

**Estimación mensual**: ~$3,500 - $5,000 USD

## Fases de Implementación

### Fase 1: MVP (3-4 meses)
- API Gateway + servicios básicos
- Recommendation Engine con algoritmo simple
- Student Profile Service + Course Catalog Service
- Web Application básica
- PostgreSQL databases
- Integración con 1 fuente laboral (InfoJobs)

### Fase 2: ML & Analytics (2-3 meses)
- ML Training Pipeline
- Model Serving
- Job Market Analyzer completo (LinkedIn + InfoJobs)
- Vector Database + búsqueda semántica
- Dashboards de analytics

### Fase 3: Conversational AI (2 meses)
- Conversational AI Service
- Chatbot Interface (WhatsApp)
- RAG implementation
- Integración con Sistema de Matrícula

### Fase 4: Optimización & Escala (2 meses)
- Mobile App
- Redis caché optimizado
- Event Bus (Kafka)
- Performance tuning
- A/B testing framework
- Monitoring avanzado

**Tiempo total estimado**: 9-11 meses

## Próximos Pasos

1. **Validación de arquitectura** con stakeholders técnicos
2. **Definición de APIs** (OpenAPI/Swagger specs)
3. **Diseño de base de datos** (ERD detallado)
4. **Prototipo de Recommendation Engine** (Jupyter Notebook)
5. **PoC de Job Market Analyzer** (scraping básico)
6. **Setup de infraestructura** (Terraform + K8s)
7. **Sprint 0**: Setup de repositorios, CI/CD, estándares de código
