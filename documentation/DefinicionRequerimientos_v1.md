| ID     | Tipo         | Descripción                                                                                
| Criterios de aceptación                                                                                                                     
|
| ------ | ------------ | 
------------------------------------------------------------------------------------------ 
| 
------------------------------------------------------------------------------------------------------------------------------------------- 
|
| RF-01  | Funcional    | Gestionar catálogos de divisiones, 
tipos de aula y semestres                               | Los 
catálogos se crean, editan, consultan y eliminan; Bloqueo de 
eliminación si hay referencias activas; Disponibles en listas 
desplegables |
| RF-02  | Funcional    | Registrar carreras, versiones de 
malla, materias por malla y equivalencias                 | 
Cada carrera tiene ≥1 versión de malla; Las versiones 
incluyen vigencia (inicio/fin); Se pueden definir 
equivalencias entre materias        |
| RF-03  | Funcional    | Administrar materias con clave 
única, créditos y tipo de aula requerido                    | 
No hay claves de materia duplicadas; Cada materia referencia 
un tipo de aula válido; CRUD completo                                          
|
| RF-04  | Funcional    | Crear y gestionar NRC con profesor, 
aula, cupos, horarios y estado activo                  | NRC 
con identificador único; Sin solapamiento de horarios por 
aula/profesor; Estado activo controla inscribibilidad                         
|
| RF-05  | Funcional    | Registrar profesores con 
disponibilidad, límite de horas y materias impartibles            
| No se asignan NRC fuera de disponibilidad; No exceder límite 
semanal; Validación contra materias habilitadas                                
|
| RF-06  | Funcional    | Registrar aulas con tipo, capacidad 
y disponibilidad                                       | No 
asignar NRC que supere capacidad; Validar contra 
disponibilidad; CRUD completo                                                           
|
| RF-07  | Funcional    | Administrar usuarios con roles 
(coordinador, analista, admin)                              | 
Cada usuario tiene rol; Permisos restringidos al rol; Intentos 
de acceso no autorizado son denegados y registrados                          
|
| RF-08  | Funcional    | Registrar alumnos, su carrera y su 
historial de materias con resultados e intentos         | 
Historial por periodo y materia; Máximo cuatro intentos por 
materia; Importación/exportación del historial                                  
|
| RF-09  | Funcional    | Calcular y almacenar métricas de 
complejidad, desempeño docente y confiabilidad de alumnos | 
Cálculo automático a partir de datos históricos; Métricas 
consultables y exportables; Recalculo programado                                  
|
| RF-10  | Funcional    | Crear, versionar y compartir 
estimaciones de demanda con escenarios                        
| Estimaciones con versión, autor y fecha; Compartibles por 
rol autorizado; Historial de cambios visible                                      
|
| RNF-01 | No Funcional | Utilizar PostgreSQL y mantener 
integridad referencial                                      | 
Todas las tablas con PK/FK; Constraints y cascadas definidas; 
Transacciones mantienen consistencia                                          
|
| RNF-02 | No Funcional | Garantizar la unicidad de NRC y 
claves de materias                                         | 
Índices/constraints UNIQUE creados; Inserciones duplicadas son 
rechazadas                                                                   
|
| RNF-03 | No Funcional | Implementar control de acceso basado 
en roles (RBAC)                                       | 
Usuarios solo acceden a recursos permitidos; Auditoría de 
accesos; Pruebas de autorización superadas                                        
|
| RNF-04 | No Funcional | Aplicar reglas de negocio en la 
lógica (solapes y límite de intentos)                      | 
Validaciones previas a guardar; Errores claros al usuario; 
Cobertura en pruebas unitarias/integración                                       
|
| RNF-05 | No Funcional | Soportar vigencias temporales para 
versiones de malla y equivalencias                      | No 
se pueden usar versiones vencidas en nuevos registros; 
Selección filtra por vigencia activa                                              
|
| RNF-06 | No Funcional | Permitir escalabilidad para 
múltiples carreras, materias y estimaciones                    
| Capaz de manejar ≥10,000 alumnos, 500 materias y 50 
carreras sin degradación notable; Índices y planes de consulta 
optimizados              |
| RNF-07 | No Funcional | Registrar versiones y cambios en las 
estimaciones para auditoría                           | 
Bitácora con autor, timestamp y cambio; Recuperación de 
versión anterior (rollback) disponible                                              
|

