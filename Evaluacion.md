[INFORME DE FIN DE UNIDAD.pdf](https://github.com/user-attachments/files/34690061/INFORME.DE.FIN.DE.UNIDAD.pdf)

# Informe

*Título*

Diseño e implementación de una base de datos relacional para la gestión del triaje
de pacientes en el servicio de emergencias de un centro de salud.


### Diagrama E-R


<img width="888" height="792" alt="image" src="https://github.com/user-attachments/assets/227a69cb-4cac-45b0-a9ce-ff58a43a8a94" />


### Diagrama Relacional


<img width="1016" height="792" alt="image" src="https://github.com/user-attachments/assets/024f7681-cd79-4292-97e4-d64596e9d27a" />


### Código SQL


CREATE TABLE PACIENTE (
 id_paciente INT NOT NULL AUTO_INCREMENT,
 cedula VARCHAR(10) NOT NULL,
 nombre VARCHAR(50) NOT NULL,
 apellido VARCHAR(50) NOT NULL,
 fecha_nacimiento DATE NOT NULL,
 telefono VARCHAR(15),
 PRIMARY KEY (id_paciente),
 CONSTRAINT uq_paciente_cedula UNIQUE (cedula),
 CONSTRAINT ck_paciente_cedula CHECK (CHAR_LENGTH(cedula) = 10)
);

CREATE TABLE MEDICO (
 id_medico INT NOT NULL AUTO_INCREMENT,
 nombre VARCHAR(50) NOT NULL,
 apellido VARCHAR(50) NOT NULL,
 especialidad VARCHAR(50) NOT NULL,
 PRIMARY KEY (id_medico)
);

CREATE TABLE TRIAJE (
 id_triaje INT NOT NULL AUTO_INCREMENT,
 id_paciente INT NOT NULL,
 id_medico INT NOT NULL,
 fecha_evaluacion DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
 nivel_prioridad VARCHAR(20) NOT NULL,
 observacion VARCHAR(255),
 PRIMARY KEY (id_triaje),
 CONSTRAINT ck_triaje_nivel CHECK (nivel_prioridad IN ('Rojo', 'Amarillo',
'Verde')),
 CONSTRAINT fk_triaje_paciente FOREIGN KEY (id_paciente)
 REFERENCES PACIENTE(id_paciente)
 ON UPDATE CASCADE ON DELETE RESTRICT,
 CONSTRAINT fk_triaje_medico FOREIGN KEY (id_medico)
 REFERENCES MEDICO(id_medico)
 ON UPDATE CASCADE ON DELETE RESTRICT
);


<img width="869" height="855" alt="image" src="https://github.com/user-attachments/assets/3dad02ed-4bc1-4e6d-9a70-cca0a4df500b" />



### Inserción de Datos 

-- Datos de PACIENTE
INSERT INTO PACIENTE (cedula, nombre, apellido, fecha_nacimiento, telefono)
VALUES
('1104567890', 'Juan', 'Pérez', '1990-05-12', '0991234567'),
('1109876543', 'María', 'Gómez', '1985-08-22', '0987654321'),
('1101234567', 'Carlos', 'Loaiza', '2000-01-15', '0971122334'),
('1103334445', 'Ana', 'Torres', '1995-11-30', '0964455667'),
('1106667778', 'Luis', 'Benítez', '1978-04-05', '0957788990'),
('1150012345', 'Valentina', 'Chamba', '2018-03-09', '0983344556');

-- Datos de MEDICO
INSERT INTO MEDICO (nombre, apellido, especialidad) VALUES
('Roberto', 'Cueva', 'Emergenciólogo'),
('Sofía', 'Mendoza', 'Medicina General'),
('Diego', 'Sarango', 'Pediatría'),
('Elena', 'Ríos', 'Enfermería de Triaje'),
('Mateo', 'Vargas', 'Medicina General');

-- Datos de TRIAJE
INSERT INTO TRIAJE (id_paciente, id_medico, fecha_evaluacion, nivel_prioridad,
observacion) VALUES
(1, 1, '2026-11-02 08:15:00', 'Rojo', 'Dificultad respiratoria severa'),
(2, 2, '2026-11-02 09:40:00', 'Verde', 'Consulta por dolor articular
leve'),
(3, 1, '2026-11-03 14:20:00', 'Amarillo', 'Fiebre alta persistente y
deshidratación'),
(4, 1, '2026-11-04 11:05:00', 'Rojo', 'Traumatismo por caída de altura'),
(5, 2, '2026-11-05 16:30:00', 'Verde', 'Control sintomático leve'),
(1, 4, '2026-11-06 10:10:00', 'Amarillo', 'Control posterior: saturación de
oxígeno en 91%'),
(3, 5, '2026-11-07 18:45:00', 'Verde', 'Cefalea leve sin signos de
alarma'),
(2, 1, '2026-11-08 07:50:00', 'Amarillo', 'Dolor torácico atípico con signos
vitales estables'),
(6, 3, '2026-11-08 12:25:00', 'Amarillo', 'Fiebre de 39 °C y vómitos');


### Consultas

Consulta 1 — Selección (σ) con condición compuesta
Objetivo: Obtener las evaluaciones de triaje de prioridad Rojo o Amarillo realizadas a
partir del 4 de noviembre de 2026, es decir, los casos graves o urgentes más recientes que
requieren seguimiento.
Álgebra relacional:
σ (nivel_prioridad = 'Rojo' ∨ nivel_prioridad = 'Amarillo') ∧ fecha_evaluacion
≥ '2026-11-04' (TRIAJE)
SQL:
SELECT *
FROM TRIAJE
WHERE nivel_prioridad IN ('Rojo', 'Amarillo')
 AND fecha_evaluacion >= '2026-11-04';


<img width="1321" height="339" alt="image" src="https://github.com/user-attachments/assets/c4480e79-486c-4f3f-8d55-6836544d653d" />



Consulta 2 — Proyección (π) combinada con selección (σ)
Objetivo: Obtener la cédula, el nombre, el apellido y el teléfono de los pacientes nacidos
antes de 1990, por ejemplo, para un programa de seguimiento de adultos de mayor edad.
Álgebra relacional:
π cedula, nombre, apellido, telefono (σ fecha_nacimiento < '1990-01-01'
(PACIENTE))
SQL:
SELECT cedula, nombre, apellido, telefono
FROM PACIENTE
WHERE fecha_nacimiento < '1990-01-01';



<img width="1105" height="421" alt="image" src="https://github.com/user-attachments/assets/b5fd3f6a-32bf-4005-8fe0-2137c90669a3" />




Consulta 3 — Diferencia ( ) −
Objetivo: Identificar a los pacientes que nunca han recibido una clasificación de prioridad
Rojo. Se obtiene restando, del conjunto de todos los pacientes, el conjunto de los que
tienen al menos un triaje rojo.
Álgebra relacional:
π cedula, nombre, apellido ( PACIENTE ⋈ ( π id_paciente (PACIENTE) − π
id_paciente (σ nivel_prioridad = 'Rojo' (TRIAJE)) ) )
SQL:
SELECT P.cedula, P.nombre, P.apellido
FROM PACIENTE P
WHERE P.id_paciente NOT IN (
 SELECT T.id_paciente
 FROM TRIAJE T
 WHERE T.nivel_prioridad = 'Rojo'
);



<img width="1028" height="568" alt="image" src="https://github.com/user-attachments/assets/2a0d1670-e9e3-47cd-8f50-5c602d43aa51" />




Consulta 4 — Reunión natural / JOIN ( ) de tres tablas con selección ⋈
Objetivo: Consultar, para todos los casos que no son de prioridad Verde, el nombre del
paciente, el profesional que lo evaluó con su especialidad, la fecha, el nivel de prioridad y
la observación.
Álgebra relacional:
π P.nombre, P.apellido, M.nombre, M.apellido, M.especialidad,
T.fecha_evaluacion, T.nivel_prioridad, T.observacion
 ( σ T.nivel_prioridad ≠ 'Verde'
 ( ( ρP(PACIENTE) ⋈ P.id_paciente = T.id_paciente ρT(TRIAJE) )
 ⋈ T.id_medico = M.id_medico ρM(MEDICO) ) )
Observación: se usa reunión con condición explícita (theta-join) y renombrado (ρ) en lugar
de reunión natural pura, porque PACIENTE y MEDICO comparten los atributos «nombre» y
«apellido»; una reunión natural los igualaría también y devolvería resultados incorrectos.
SQL:
SELECT P.nombre AS paciente_nombre, P.apellido AS paciente_apellido,
 M.nombre AS profesional_nombre, M.apellido AS profesional_apellido,
 M.especialidad, T.fecha_evaluacion, T.nivel_prioridad, T.observacion
FROM TRIAJE T
JOIN PACIENTE P ON T.id_paciente = P.id_paciente
JOIN MEDICO M ON T.id_medico = M.id_medico
WHERE T.nivel_prioridad <> 'Verde'
ORDER BY FIELD(T.nivel_prioridad, 'Rojo', 'Amarillo'), T.fecha_evaluacion;




<img width="1371" height="570" alt="image" src="https://github.com/user-attachments/assets/506ceed1-ab06-48cf-b2d6-ee6821ec193e" />





Consulta 5 — Agrupamiento y agregación (γ) con reunión y condición de
grupo
Objetivo: Obtener cuántos triajes ha realizado cada profesional, mostrando solo a
quienes han realizado dos o más evaluaciones, para analizar la carga de trabajo del
servicio.
Álgebra relacional:
σ total_triajes ≥ 2 ( γ id_medico, nombre, apellido, especialidad ;
COUNT(id_triaje) → total_triajes ( MEDICO ⋈ TRIAJE ) )
SQL:
SELECT M.id_medico, M.nombre, M.apellido, M.especialidad,
 COUNT(T.id_triaje) AS total_triajes
FROM MEDICO M
JOIN TRIAJE T ON M.id_medico = T.id_medico
GROUP BY M.id_medico, M.nombre, M.apellido, M.especialidad
HAVING COUNT(T.id_triaje) >= 2
ORDER BY total_triajes DESC;




<img width="1298" height="508" alt="image" src="https://github.com/user-attachments/assets/eb420538-ac73-4ad9-8816-c855d179fa17" />





