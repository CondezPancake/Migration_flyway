# MindConnect - Migración MySQL con Spring Boot y Flyway

Este proyecto crea el esquema MySQL de MindConnect a partir del diagrama entidad-relación suministrado. Spring Boot inicia Flyway y Flyway ejecuta las 52 migraciones SQL ubicadas en `src/main/resources/db/migration`.

## Paso a paso

### 1. Requisitos

- JDK 21.
- MySQL 8.
- PowerShell en Windows. El proyecto incluye Maven Wrapper, por lo que no es necesario instalar Maven globalmente.

Comprueba que Java apunte a un JDK y no a un JRE:

```powershell
java -version
javac -version
```

### 2. Crear la base de datos

En MySQL crea la base antes de iniciar la aplicación:

```sql
CREATE DATABASE IF NOT EXISTS migracion_spring
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
```

### 3. Configurar las credenciales locales

Copia el archivo de ejemplo:

```powershell
Copy-Item .env.example .env
```

Edita `.env` con las credenciales de tu instalación local. El archivo no se versiona porque está incluido en `.gitignore`.

```properties
DB_URL=jdbc:mysql://localhost:3306/migracion_spring
DB_USERNAME=root
DB_PASSWORD=tu_contrasena
```

`application.properties` importa ese archivo y entrega las variables a Spring Boot. No se utiliza `@Value`, porque no hay código Java que necesite consumir la configuración directamente.

### 4. Ejecutar las migraciones y las pruebas

Desde la raíz del proyecto ejecuta:

```powershell
.\mvnw.cmd clean test
```

La prueba levanta el contexto de Spring Boot. Flyway valida las migraciones existentes y aplica las pendientes en `migracion_spring`.

Si Java 25 está instalado como JRE y Java 21 como JDK, define el JDK para la terminal actual antes de ejecutar Maven:

```powershell
$env:JAVA_HOME='C:\Program Files\Eclipse Adoptium\jdk-21.0.12.101-hotspot'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\mvnw.cmd clean test
```

### 5. Verificar el resultado

Conéctate a MySQL y consulta el historial de Flyway:

```sql
USE migracion_spring;

SELECT installed_rank, version, description, success
FROM flyway_schema_history
ORDER BY installed_rank;
```

El resultado debe mostrar las versiones `1` a `52` con `success = 1`.

También puedes iniciar la aplicación directamente:

```powershell
.\mvnw.cmd spring-boot:run
```

### 6. Crear una migración nueva

Después de aplicar una versión, no se modifica su archivo SQL. Para un cambio nuevo se crea otro archivo, por ejemplo:

```text
src/main/resources/db/migration/V53__add_example_column.sql
```

Después se ejecuta otra vez:

```powershell
.\mvnw.cmd test
```

Flyway guarda el checksum de cada archivo en `flyway_schema_history`. Cambiar una migración aplicada genera un error de validación. `repair` solo debe usarse si se conoce con certeza la causa del cambio y el esquema ya es correcto.

## Configuración del proyecto

La configuración está en `src/main/resources/application.properties`:

- `spring.config.import`: carga `.env` si existe.
- `spring.datasource.*`: define la conexión MySQL mediante variables `DB_*`.
- `spring.flyway.locations`: usa `classpath:db/migration`.
- `spring.flyway.validate-migration-naming`: exige nombres de migración válidos.

Las dependencias relevantes son `spring-boot-starter-flyway`, `flyway-mysql` y `mysql-connector-j`. No se incluyó JPA ni se crearon entidades, repositorios o controladores porque el alcance actual es la migración de base de datos. Esos componentes solo serían necesarios si el proyecto también debe exponer una API o implementar operaciones CRUD.

## Tablas del esquema

El esquema contiene 52 tablas.

### Ubicación y catálogos base

| Tabla | Propósito |
| --- | --- |
| `countries` | Países. |
| `state_regions` | Regiones o departamentos de un país. |
| `city_municipalities` | Ciudades o municipios de una región. |
| `document_types` | Tipos de documento. |
| `genders` | Catálogo de género y sexo biológico. |
| `relationship_types` | Tipos de parentesco o relación. |
| `professional_types` | Tipos de profesionales. |
| `studies` | Estudios o formación académica. |

### Personas y contactos

| Tabla | Propósito |
| --- | --- |
| `professionals` | Profesionales que atienden pacientes. |
| `patients` | Datos generales de pacientes. |
| `contacts` | Contactos de pacientes. |
| `phone_contacts` | Teléfonos de contactos. |
| `email_contacts` | Correos de contactos. |
| `patient_contacts` | Relación entre paciente y contacto. |
| `patient_allergies` | Alergias del paciente. |
| `professional_studies` | Estudios registrados por profesional. |

### Historia clínica y tratamientos

| Tabla | Propósito |
| --- | --- |
| `clinical_record_statuses` | Estados de historias clínicas. |
| `clinical_records` | Historias clínicas de pacientes. |
| `encounter_types` | Tipos de atención. |
| `encounter_modalities` | Modalidad de atención. |
| `encounter_statuses` | Estados de atención. |
| `encounters` | Consultas o encuentros clínicos. |
| `risk_levels` | Niveles de riesgo. |
| `risk_assessments` | Evaluaciones de riesgo. |
| `clinical_notes` | Notas clínicas. |
| `mental_status_exams` | Exámenes de estado mental. |
| `treatment_statuses` | Estados de tratamientos. |
| `treatment_plans` | Planes de tratamiento. |
| `treatment_goal_statuses` | Estados de objetivos terapéuticos. |
| `treatment_goals` | Objetivos de un tratamiento. |
| `medication_routes` | Vías de administración de medicamentos. |
| `assessment_types` | Tipos de evaluación. |
| `consent_types` | Tipos de consentimiento. |
| `diagnostic_systems` | Sistemas de diagnóstico. |

### Chat, IA y escalamiento

| Tabla | Propósito |
| --- | --- |
| `conversations_statuses` | Estados de conversaciones. |
| `priorities` | Prioridades de conversaciones. |
| `sender_types` | Tipos de remitente o participante. |
| `message_types` | Tipos de mensajes. |
| `chat_conversations` | Conversaciones de chat. |
| `chat_participants` | Participantes de una conversación. |
| `chat_messages` | Mensajes enviados. |
| `provider_models_ai` | Proveedores de modelos de IA. |
| `ai_models` | Modelos de IA disponibles. |
| `chat_conversation_ai_settings` | Configuración de IA por conversación. |
| `ai_runs_statuses` | Estados de ejecuciones de IA. |
| `chat_ai_runs` | Ejecuciones de IA sobre una conversación. |
| `chat_ai_run_metrics` | Métricas y costo de una ejecución. |
| `chat_ai_run_errors` | Errores de una ejecución. |
| `escalations_statuses` | Estados de escalamiento. |
| `chat_escalations` | Escalamientos de conversaciones. |
| `chat_escalation_assignments` | Profesionales asignados a escalamiento. |
| `chat_escalation_status_history` | Historial de estados de escalamiento. |

## Cardinalidades principales

| Relación | Cardinalidad | Explicación |
| --- | --- | --- |
| `countries` → `state_regions` | 1:N | Un país tiene varias regiones. |
| `state_regions` → `city_municipalities` | 1:N | Una región tiene varias ciudades. |
| `city_municipalities` → `patients`, `professionals`, `contacts` | 1:N | Una ciudad puede estar asociada a muchas personas. |
| `document_types` → `patients`, `professionals` | 1:N | Un tipo de documento se usa en varios registros. |
| `patients` → `clinical_records` | 1:N | Un paciente puede tener varias historias clínicas. |
| `clinical_records` → `encounters` | 1:N | Una historia clínica registra varias atenciones. |
| `professionals` → `encounters` | 1:N | Un profesional puede atender varios encuentros. |
| `encounters` → `risk_assessments`, `clinical_notes`, `mental_status_exams` | 1:N | Un encuentro puede registrar estos elementos clínicos. |
| `encounters` → `treatment_plans` → `treatment_goals` | 1:N y 1:N | Un encuentro puede generar planes; cada plan, varios objetivos. |
| `patients` ↔ `contacts` | N:M | Se resuelve mediante `patient_contacts`. |
| `contacts` → `phone_contacts`, `email_contacts` | 1:N | Un contacto puede registrar varios teléfonos y correos. |
| `chat_conversations` → `chat_participants`, `chat_messages` | 1:N | Una conversación tiene participantes y mensajes. |
| `chat_messages` → `chat_ai_runs` | 1:N | Un mensaje puede desencadenar ejecuciones de IA. |
| `provider_models_ai` → `ai_models` | 1:N | Un proveedor ofrece varios modelos. |
| `chat_conversations` → `chat_escalations` | 1:N | Una conversación puede requerir varios escalamientos. |
| `chat_escalations` → asignaciones e historial | 1:N | Un escalamiento puede tener varios responsables y cambios de estado. |

## Adaptaciones necesarias para MySQL

El diagrama de referencia emplea tipos propios de PostgreSQL o tiene algunos campos incompletos. Las migraciones aplican estas equivalencias:

| Referencia | Implementación MySQL | Motivo |
| --- | --- | --- |
| `UUID` | `CHAR(36)` | MySQL no posee el tipo `UUID` nativo. |
| `TIMESTAMPTZ` | `TIMESTAMP` | MySQL no implementa `TIMESTAMPTZ`. |
| `JSONB` | `JSON` | `JSONB` es específico de PostgreSQL. |
| `provider_models_ai.razon_social VARCHAR` | `VARCHAR(150)` | MySQL exige longitud para `VARCHAR`. |
| `ai_models.provider_model_id VARCHAR(50)` | `CHAR(36)` | Debe coincidir con la PK UUID del proveedor para crear la FK. |

Además, `Column1` y `Column2` no se incluyeron en `phone_contacts`, porque el diagrama no les asigna tipo ni propósito. Se conserva `treatment_goal_id` como nombre de columna, aunque su relación apunta al catálogo `treatment_goal_statuses`, tal como se muestra en el diagrama.

## Pendientes funcionales

El proyecto crea la estructura, pero no inserta datos en los catálogos. Si se requieren tipos de documento, estados, prioridades, modalidades o géneros iniciales, deben agregarse en una nueva migración, por ejemplo `V53__seed_catalog_data.sql`.

Si el objetivo incluye mover información real desde otra base de datos, también falta un proceso de migración de datos: extracción, transformación, carga, validación y respaldo. Por tratarse de datos clínicos, esos datos no deben quedar incluidos en el repositorio ni en migraciones SQL versionadas.

Para entornos distintos de desarrollo, se recomienda usar un usuario MySQL específico para migraciones en lugar de `root`, limitar sus privilegios y almacenar las credenciales fuera del repositorio.
