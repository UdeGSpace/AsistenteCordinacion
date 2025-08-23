###Asistente Coordinación

##Diseño del esquema de datos inicial 

Catalogo de Materias
[
	{
		Clave - Identificador de materia, una clave puede tener muchas secciones
		NRC - Identificador de sección 
		Semestre - {fecha inicio - fecha de terminación}
		Horario - Dias y horas en las que se imparte la clase. {Dias de la semana, hora de inicio y fin}
		Carreras [] - Lista de carreras que pueden compartir esta materia entre si 
		Créditos int - Cantidad de créditos que da la materia 
		Lab - Tipo de aula que necesita 
		Complejidad - numero calculado con base al histórico de calificaciones obtenidas por los alumnos y la relación entre alumnos que cursaron y alumnos aprobados 
        calificacion minima
	}
]

Catalog de carreras
[
	{
		Id - Codigo de carrera
		Division - Grupo al que pertenece la carrera
		Duración - Cantidad de semestres 
		Malla Curricular [] - lista de las materias necesarias para que el alumnos culmine su carrera, cada materia es representada por la Clave del catálogo de materias
		Créditos necesarios - Cantidad de créditos que el alumno debe conseguir para poder titularse 
		Capacidad - Cantidad maxima de nuevos alumnos que la carrera acepta cada semestre
	
	}
]

Catalog de Aulas
[
	{
		ID - Nombre del edificio y el numero de aula
		Tipo de aula - lista de los tipos de aulas disponibles, ejemplo lab de computación, lab de electrónica, aula normal
		Capacidad - Cantidad de alumnos 
		Disponibilidad - lista de horarios en los que opera
	}
]

Catalog de Profesores
[
	{
		ID - Identificador de profesor
		Divisiones - grupo de carreras donde el profesor imparte clases, puedes ser mas de 1
		materias - lista de materias que el profesor puede impartir, esta lista usara Clave del catálogo de materias
		Disponibilidad - días y horas en las que el profesor puede impartir clase
		LimiteHoras - Cantidad de horas a la semana que el profesor puede impartir clases
		Score - Valor calculado en base al porcentaje de alumnos aprobados vs la complejidad de materia 
	}
]
Alumno
{
	id - numero de identificación no relacionado con el código del alumno, este id se usara únicamente en el sistema y no tiene relación con la información confidencial del alumno
	Fecha de ingreso - Date 
	IndiceConfiabilidad int - Calculado con base a las materias cursadas, complejidad de materias cursadas, calificación obtenida, cantidad de veces que repitió una materia
	Materias Array - [] (Lista de NRC que el alumno a cursado con la calificación obtenida)
	carrera - id de carrera a la que pertenece	
}