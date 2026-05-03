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


## Inserts para test
-- 1. Insertar empresa
INSERT INTO company (name, description) 
VALUES ('TechCorp SL', 'Empresa de tecnología líder en España');

-- 2. Insertar empleado (reclutador)
INSERT INTO employee (company_id, name, email, role, is_active)
VALUES (1, 'María López', 'maria.lopez@techcorp.com', 'recruiter', true);

-- 3. Insertar tipo de entrevista
INSERT INTO "interviewType" (name, description)
VALUES ('Técnica', 'Entrevista de habilidades técnicas');

-- 4. Insertar flujo de entrevista
INSERT INTO "interviewFlow" (description)
VALUES ('Proceso estándar de selección tech');

-- 5. Insertar paso del flujo
INSERT INTO "interviewStep" (interview_flow_id, interview_type_id, name, order_index)
VALUES (1, 1, 'Entrevista técnica inicial', 1);

-- 6. Insertar posición
INSERT INTO position (company_id, interview_flow_id, title, status, 
  is_visible, location, salary_min, salary_max, employment_type)
VALUES (1, 1, 'Backend Developer', 'open', true, 
  'Madrid', 35000, 55000, 'full-time');

-- 7. Insertar candidato
INSERT INTO "Candidate" ("firstName", "lastName", email, phone)
VALUES ('Carlos', 'Martínez', 'carlos@email.com', '612345678');

-- 8. Insertar aplicación
INSERT INTO application (position_id, candidate_id, status)
VALUES (1, 1, 'in_progress');

-- 9. Insertar entrevista
INSERT INTO interview (application_id, interview_step_id, 
  employee_id, interview_date, result, score)
VALUES (1, 1, 1, '2026-05-10 10:00:00', 'pending', null);

## Select para test
-- 1. Ver todos los candidatos con sus aplicaciones y posición
SELECT 
  c."firstName", 
  c."lastName", 
  c.email,
  p.title as posicion,
  a.status as estado_aplicacion,
  a.application_date
FROM "Candidate" c
JOIN application a ON a.candidate_id = c.id
JOIN position p ON p.id = a.position_id;

-- 2. Ver el flujo completo de entrevistas de un candidato
SELECT 
  c."firstName",
  c."lastName",
  p.title as posicion,
  ist.name as paso_entrevista,
  ist.order_index,
  i.interview_date,
  i.result,
  i.score,
  e.name as entrevistador
FROM interview i
JOIN application a ON a.id = i.application_id
JOIN "Candidate" c ON c.id = a.candidate_id
JOIN position p ON p.id = a.position_id
JOIN "interviewStep" ist ON ist.id = i.interview_step_id
JOIN employee e ON e.id = i.employee_id;

-- 3. Posiciones abiertas con rango salarial
SELECT 
  p.title,
  p.location,
  p.employment_type,
  p.salary_min,
  p.salary_max,
  co.name as empresa,
  COUNT(a.id) as total_aplicaciones
FROM position p
JOIN company co ON co.id = p.company_id
LEFT JOIN application a ON a.position_id = p.id
WHERE p.status = 'open' AND p.is_visible = true
GROUP BY p.id, co.name;