# Prompts — Migración de base de datos LTI
## Desarrollado por: VACR

## Prompt 1 — Expansión del schema.prisma
Usado en Cursor (modo Agent):

"Eres un experto en bases de datos PostgreSQL y Prisma ORM.

Tengo el siguiente schema.prisma actual con estos modelos:
Candidate, Education, WorkExperience, Resume

Necesito expandirlo agregando los siguientes modelos del ERD:

erDiagram
     COMPANY { int id PK, string name }
     EMPLOYEE { int id PK, int company_id FK, string name, 
       string email, string role, boolean is_active }
     POSITION { int id PK, int company_id FK, 
       int interview_flow_id FK, string title, text description,
       string status, boolean is_visible, string location,
       text job_description, text requirements, 
       text responsibilities, numeric salary_min, 
       numeric salary_max, string employment_type, 
       text benefits, text company_description, 
       date application_deadline, string contact_info }
     INTERVIEW_FLOW { int id PK, string description }
     INTERVIEW_STEP { int id PK, int interview_flow_id FK, 
       int interview_type_id FK, string name, int order_index }
     INTERVIEW_TYPE { int id PK, string name, text description }
     APPLICATION { int id PK, int position_id FK, 
       int candidate_id FK, date application_date, 
       string status, text notes }
     INTERVIEW { int id PK, int application_id FK, 
       int interview_step_id FK, int employee_id FK, 
       date interview_date, string result, int score, 
       text notes }

Requisitos de buenas prácticas:
1. Mantén los modelos existentes sin modificarlos
2. Agrega índices en todas las foreign keys
3. Agrega índices compuestos donde tenga sentido 
   (ej: candidate_id + position_id en Application)
4. Usa tipos de datos apropiados para PostgreSQL:
   - salary_min/max como Decimal
   - fechas como DateTime
   - textos largos como String sin límite
   - strings cortos con @db.VarChar()
5. Aplica normalización: no repitas datos
6. Agrega relaciones bidireccionales correctas en Prisma
7. Usa nombres en camelCase para los modelos Prisma

Genera el schema.prisma completo actualizado manteniendo 
el datasource y generator existentes."

## Resultado
- 8 nuevos modelos agregados
- Índices en todas las FKs
- Índices compuestos en application e interviewStep
- Migración aplicada exitosamente
- BD verificada en PGAdmin