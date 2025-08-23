# AsistenteCordinación — Modelo de Datos (v1)
**Fecha:** 2025-08-23 

Este documento describe el **modelo de datos** refinado para AsistenteCordinación en **formato de tablas**. Incluye catálogos, estructura académica, materias/NRC, profesores/aulas, usuarios/alumnos/historial, métricas y estimaciones.

> **Convenciones:** PostgreSQL; PK=Primary Key; FK=Foreign Key; UNQ=Índice único.  
> **Notas:** Reglas como *no solapes* y *límite de 4 intentos* se aplican en lógica de aplicación o políticas/trigger, no como constraint duro aquí.

---

## 0) Resumen de tablas
- **Catálogos:** `divisiones`, `tipos_aula`, `semestres`
- **Estructura académica:** `carreras`, `versiones_malla`, `malla_materias`, `equivalencias_materias`
- **Materias y NRC:** `materias`, `nrc` *(con aula, cupos y horario_semana_json)*
- **Profesores y aulas:** `profesores`, `profesor_materia`, `disponibilidad_profesor`, `aulas`, `disponibilidad_aula`
- **Usuarios, alumnos e historial:** `usuarios`, `alumnos`, `historial_alumno`
- **Métricas:** `metrica_complejidad_materia`, `metrica_score_profesor`, `metrica_confiabilidad_alumno`, `materia_similar_referencia`
- **Estimaciones:** `estimaciones`, `estimacion_compartida`, `estimacion_detalle_materia`

---

## 1) Catálogos básicos

### divisiones
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Identificador de división |
| nombre | TEXT | UNQ | Nombre de la división |

### tipos_aula
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Identificador de tipo de aula |
| nombre | TEXT | UNQ | Ej.: Aula Normal, Lab. Computación, Lab. Electrónica |

### semestres
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| codigo | TEXT | PK | ‘AAAA-A’ / ‘AAAA-B’ |
| fecha_inicio | DATE |  | Inicio de periodo |
| fecha_fin | DATE |  | Fin de periodo |

---

## 2) Estructura académica (carreras, malla, equivalencias)

### carreras
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id carrera |
| codigo | TEXT | UNQ | Código oficial |
| nombre | TEXT |  | Nombre |
| division_id | BIGINT | FK→divisiones.id | División |
| duracion_semestres | INT |  | Duración del plan |
| creditos_para_titularse | INT |  | Créditos para titulación |
| capacidad_ingreso_sem | INT |  | Cupos de nuevo ingreso por semestre |

### versiones_malla
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id versión |
| carrera_id | BIGINT | FK→carreras.id | Carrera |
| version | TEXT | UNQ(carrera_id,version) | Etiqueta de versión |
| vigente_desde | TEXT | FK→semestres.codigo | Desde |
| vigente_hasta | TEXT | FK→semestres.codigo | Hasta (NULL=vigente) |

### malla_materias
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| version_malla_id | BIGINT | PK/FK→versiones_malla.id | Versión |
| clave_materia | TEXT | PK | Clave materia |
| semestre_sugerido | INT |  | Semestre sugerido |

### equivalencias_materias
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id equivalencia |
| carrera_id | BIGINT | FK→carreras.id | Ámbito por carrera |
| clave_origen | TEXT |  | Materia A |
| clave_equivalente | TEXT |  | Equivalente de A |
| vigente_desde | TEXT | FK→semestres.codigo | Desde |
| vigente_hasta | TEXT | FK→semestres.codigo | Hasta |

---

## 3) Materias y NRC

### materias
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| clave | TEXT | PK | Clave única institucional |
| nombre | TEXT |  | Nombre |
| creditos | INT |  | Créditos |
| calificacion_minima_aprob | NUMERIC(4,2) |  | Mínimo aprobatorio por materia |
| tipo_aula_requerida_id | BIGINT | FK→tipos_aula.id | Tipo requerido |

### nrc *(único global; consolidado)*
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| nrc | TEXT | PK/UNQ | Identificador global de NRC |
| clave_materia | TEXT | FK→materias.clave | Materia |
| profesor_id | BIGINT | FK→profesores.id | Profesor “fijo” del NRC |
| aula_id | BIGINT | FK→aulas.id | Aula asignada al NRC |
| cupos | INT |  | Cupos fijos del NRC |
| horario_semana_json | JSONB |  | Bloques semanales recurrentes (ver Anexo A) |
| activo | BOOLEAN |  | Activo/Inactivo |

> **Regla:** si maestro/horario/aula/cupos cambian, se genera **nuevo NRC** (alta desde BD principal).

---

## 4) Profesores y aulas

### profesores
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id profesor |
| nombre | TEXT |  | Nombre |
| division_id | BIGINT | FK→divisiones.id | División |
| limite_horas_semanales | INT |  | Carga máxima semanal |

### profesor_materia
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| profesor_id | BIGINT | PK/FK→profesores.id | Profesor |
| clave_materia | TEXT | PK/FK→materias.clave | Materia habilitada |

### disponibilidad_profesor
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id disp. |
| profesor_id | BIGINT | FK→profesores.id | Profesor |
| dia_semana | SMALLINT |  | 1..7 |
| hora_inicio | TIME |  | Inicio |
| hora_fin | TIME |  | Fin |

### aulas
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id aula |
| edificio | TEXT |  | Edificio |
| numero | TEXT | UNQ(edificio,numero) | Número |
| tipo_aula_id | BIGINT | FK→tipos_aula.id | Tipo |
| capacidad | INT |  | Capacidad nominal |

### disponibilidad_aula
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id disp. |
| aula_id | BIGINT | FK→aulas.id | Aula |
| dia_semana | SMALLINT |  | 1..7 |
| hora_inicio | TIME |  | Inicio |
| hora_fin | TIME |  | Fin |

---

## 5) Usuarios, alumnos e historial

### usuarios
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id usuario |
| nombre | TEXT |  | Nombre |
| rol | TEXT |  | ‘coordinador’/‘analista’/‘admin’ |

### alumnos
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id alumno (pseudónimo) |
| fecha_ingreso | DATE |  | Ingreso |
| carrera_id | BIGINT | FK→carreras.id | Carrera actual |
| estatus | TEXT |  | ‘activo’/‘baja_institucional’ (por 4º intento fallido) |

### historial_alumno
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id registro |
| alumno_id | BIGINT | FK→alumnos.id | Alumno |
| clave_materia | TEXT | FK→materias.clave | Materia |
| nrc | TEXT | FK→nrc.nrc | NRC cursado |
| semestre | TEXT | UNQ(alumno_id,clave_materia,semestre), FK→semestres.codigo | Periodo |
| calificacion | NUMERIC(5,2) |  | Calificación numérica (si aplica) |
| aprobado | BOOLEAN |  | Resultado (según mínimo por materia) |
| modo_aprobacion | TEXT |  | ‘ordinario’/‘extraordinario’ (solo si aprobado) |
| intento_n | SMALLINT |  | 1..4 |

---

## 6) Métricas (por semestre)

### metrica_complejidad_materia
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| clave_materia | TEXT | PK/FK→materias.clave | Materia |
| semestre | TEXT | PK/FK→semestres.codigo | Periodo |
| valor | NUMERIC(6,4) |  | Escala TBD |
| n_muestra | INT |  | N alumnos |
| ventana_usada | SMALLINT |  | 3 |
| decaimiento | NUMERIC(3,2) |  | 0.9 |
| inicializada_por_similar | BOOLEAN |  | Flag de inicialización |
| materia_similar_ref | TEXT |  | Clave referencia |
| fecha_calculo | TIMESTAMPTZ |  | Timestamp |

### metrica_score_profesor
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| profesor_id | BIGINT | PK/FK→profesores.id | Profesor |
| semestre | TEXT | PK/FK→semestres.codigo | Periodo |
| valor | NUMERIC(6,4) |  | Escala TBD |
| n_muestra | INT |  | N registros |
| ventana_usada | SMALLINT |  | 3 |
| decaimiento | NUMERIC(3,2) |  | 0.9 |
| fecha_calculo | TIMESTAMPTZ |  | Timestamp |

### metrica_confiabilidad_alumno
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| alumno_id | BIGINT | PK/FK→alumnos.id | Alumno |
| semestre | TEXT | PK/FK→semestres.codigo | Periodo |
| indice | NUMERIC(6,4) |  | Escala TBD |
| num_aprobadas | INT |  | Conteo |
| num_no_aprobadas | INT |  | Conteo |
| num_extraordinarios | INT |  | Conteo |
| fecha_calculo | TIMESTAMPTZ |  | Timestamp |

### materia_similar_referencia
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id |
| carrera_id | BIGINT | FK→carreras.id | Ámbito carrera |
| clave_materia | TEXT |  | Materia destino |
| clave_materia_similar | TEXT |  | Materia “seed” |
| usuario_id | BIGINT | FK→usuarios.id | Coordinador que define |
| fecha_asignacion | TIMESTAMPTZ |  | Fecha |

---

## 7) Estimaciones (compartibles, versionadas)

### estimaciones
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id estimación |
| carrera_id | BIGINT | FK→carreras.id | Ámbito de carrera |
| usuario_id | BIGINT | FK→usuarios.id | Propietario |
| semestre_objetivo | TEXT | FK→semestres.codigo | Periodo objetivo |
| raiz_id | BIGINT |  | Id primera versión de la serie |
| version | INT |  | Nº versión (>=1) |
| estado | TEXT |  | ‘borrador’/‘final’ |
| escenario_activo | TEXT |  | ‘p50’/‘p80’/‘p95’ |
| parametros_json | JSONB |  | Políticas/escenarios |
| titulo | TEXT |  | Título |
| notas | TEXT |  | Notas |
| creada_en | TIMESTAMPTZ |  | Creación |

### estimacion_compartida
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| estimacion_id | BIGINT | PK/FK→estimaciones.id | Estimación |
| usuario_id | BIGINT | PK/FK→usuarios.id | Usuario |
| permiso | TEXT |  | ‘lectura’/‘edicion’ *(borrar: solo propietario)* |

### estimacion_detalle_materia
| Columna | Tipo | Clave | Descripción |
|---|---|---|---|
| id | BIGSERIAL | PK | Id detalle |
| estimacion_id | BIGINT | FK→estimaciones.id | Estimación |
| clave_materia | TEXT | FK→materias.clave | Materia |
| demanda_p50 | INT |  | p50 |
| demanda_p80 | INT |  | p80 |
| demanda_p95 | INT |  | p95 |
| cupos_sugeridos | INT |  | Cupos propuestos |
| propuesta_horario_json | JSONB |  | Propuesta de bloques/huecos |
| notas | TEXT |  | Notas |
| (UNQ estimacion_id, clave_materia) | — | UNQ | Único por materia/estimación |

---

## Anexo A — Ejemplo `horario_semana_json` (NRC)
```json
[
  { "dia_semana": 1, "hora_inicio": "08:00", "hora_fin": "10:00" },
  { "dia_semana": 3, "hora_inicio": "10:00", "hora_fin": "12:00" }
]
```
