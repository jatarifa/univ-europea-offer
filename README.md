# Recomendador de Trayectorias - Universidad Europea

Propuesta de arquitectura para el caso de uso "Recomendador de Trayectorias Académico-Profesionales" basado en el documento de casos de IA de Inorganic.

## Contenido

### Arquitectura Tradicional (Microservicios)
- `recomendador-trayectorias-c4.puml` - Diagrama C4 de contenedores en PlantUML
- `recomendador-trayectorias-c4.png` - Diagrama exportado (407 KB)
- `arquitectura-recomendador.md` - Documentación detallada de la arquitectura

### Arquitectura Multi-Agente IA (Recomendado)
- `recomendador-agentes-simple-c4.puml` - Diagrama C4 simplificado ✨ **RECOMENDADO**
- `recomendador-agentes-simple-c4.png` - Diagrama exportado (197 KB)
- `propuesta-arquitectura-agentes.md` - Propuesta ejecutiva para consultoría **LEER PRIMERO**

### Arquitectura Multi-Agente IA (Versión Detallada)
- `recomendador-trayectorias-agents-c4.puml` - Diagrama C4 detallado
- `recomendador-trayectorias-agents-c4.png` - Diagrama exportado (733 KB)
- `arquitectura-multi-agente.md` - Documentación técnica completa (solo si necesitas máximo detalle)

### Documentos de Referencia
- `Casos de IA para la Universidad Europea-2.pdf` - Documento original con los casos de uso

## Propuesta Recomendada: Arquitectura Multi-Agente IA

### Enfoque
Sistema inteligente basado en **5 agentes IA especializados** que colaboran para ofrecer recomendaciones personalizadas de trayectorias académico-profesionales.

### Capas del Sistema
1. **Canales de Interacción**: Web, Mobile, WhatsApp
2. **Orquestador de Agentes**: Coordinación inteligente
3. **Agentes Especializados**: 5 expertos por dominio
4. **Recursos**: Base de conocimiento y herramientas
5. **Fuentes de Datos**: Universidad + Mercado laboral

### Agentes IA Incluidos
1. **Career Advisor Agent**: Diseña trayectorias académicas personalizadas
2. **Market Intelligence Agent**: Analiza tendencias laborales en tiempo real
3. **Course Expert Agent**: Experto en catálogo de formaciones
4. **Student Coach Agent**: Acompañamiento empático y personalizado
5. **Skills Assessment Agent**: Evalúa competencias y detecta gaps

### Ventajas Clave
- ✅ **Modular y extensible**: Fácil agregar nuevos agentes
- ✅ **Especialización por dominio**: Cada agente es experto en su área
- ✅ **Razonamiento explicable**: Trazabilidad completa de decisiones
- ✅ **Autonomía**: Agentes se adaptan automáticamente
- ✅ **Escalabilidad**: Por agente individual

### Estimaciones
- **Tiempo MVP**: 4-5 meses (3 agentes core)
- **Tiempo completo**: 10-11 meses
- **Coste mensual**: ~$5,500 (infraestructura cloud)

### vs. Arquitectura Tradicional
| Aspecto | Tradicional | Multi-Agente |
|---------|-------------|--------------|
| Modularidad | Baja | ⭐⭐⭐⭐⭐ Alta |
| Autonomía | Baja | ⭐⭐⭐⭐⭐ Alta |
| Observabilidad | Limitada | ⭐⭐⭐⭐⭐ Completa |
| Coste | $3,500/mes | $5,500/mes |
| Tiempo MVP | 3-4 meses | 4-5 meses |

## Visualizar el Diagrama PlantUML

### Opción 1: VS Code (Recomendado)

1. Instala la extensión [PlantUML](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml)
2. Abre el archivo `recomendador-trayectorias-c4.puml`
3. Presiona `Alt+D` (o `Cmd+D` en Mac) para ver el preview
4. O haz clic derecho → "Preview Current Diagram"

### Opción 2: PlantUML Online

1. Visita https://www.plantuml.com/plantuml/uml/
2. Copia el contenido de `recomendador-trayectorias-c4.puml`
3. Pégalo en el editor online
4. El diagrama se generará automáticamente

### Opción 3: Exportar como PNG/SVG

Con la extensión de VS Code instalada:

```bash
# Exportar como PNG
plantuml recomendador-trayectorias-c4.puml -tpng

# Exportar como SVG
plantuml recomendador-trayectorias-c4.puml -tsvg
```

### Opción 4: Docker

```bash
docker run --rm -v $(pwd):/data plantuml/plantuml recomendador-trayectorias-c4.puml
```

## Estructura del Diagrama

El diagrama C4 de contenedores muestra:

### Actores
- **Estudiante/Futuro Alumno**: Busca orientación académica
- **Alumno Actual**: Busca formación complementaria
- **Alumni**: Busca formación continua

### Contenedores (Aplicaciones)
1. **Web Application** (React)
2. **Mobile App** (React Native)
3. **Chatbot Interface** (WhatsApp/Telegram)

### Contenedores (Backend)
4. **API Gateway** (Kong/AWS)
5. **Recommendation Engine** (Python/FastAPI) - Motor principal
6. **Conversational AI Service** (Python/LangChain) - Chatbot
7. **Job Market Analyzer** (Python/Scrapy) - Análisis de ofertas
8. **Student Profile Service** (Python/FastAPI)
9. **Course Catalog Service** (Python/FastAPI)
10. **ML Training Pipeline** (MLflow)
11. **Model Serving** (TensorFlow/TorchServe)

### Contenedores (Datos)
12. **Student Database** (PostgreSQL)
13. **Courses Database** (PostgreSQL)
14. **Job Market Database** (MongoDB)
15. **Vector Database** (Pinecone/Qdrant)
16. **Cache** (Redis)
17. **Event Bus** (Kafka/RabbitMQ)

### Sistemas Externos
- **LinkedIn API** - Ofertas de empleo
- **InfoJobs API** - Ofertas de empleo
- **Sistema de Matrícula** - Universidad Europea
- **CRM Universidad** - Gestión de leads
- **LMS Universidad** - Historial académico

## Arquitectura Detallada

Ver `arquitectura-recomendador.md` para:

- Descripción completa de cada componente
- Flujos principales del sistema
- Stack tecnológico recomendado
- Consideraciones de escalabilidad, seguridad y resiliencia
- Estimación de recursos cloud
- Plan de implementación por fases

## Casos de Uso del PDF

El documento original propone 6 casos de uso de IA:

1. ✅ **Recomendador de Trayectorias** (este proyecto)
2. Asistente Conversacional de Admisión
3. Generador de Contenido
4. Asistente de Secretaría
5. Asistente de Citación
6. Asistente de Corrección

## Contacto

Propuesta de Inorganic para Universidad Europea
- alexia@inorganic.com
- mikel@inorganic.com
