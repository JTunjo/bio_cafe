# Solicitud de Cotización (RFQ)
## Biorreactor Ultra para Fermentación Controlada de Café de Especialidad

**Versión:** Ultra  
**Aplicación:** Poscosecha — café cereza y despulpado  
**Nivel:** Estado del arte. Plataforma de producción y conocimiento.

---

## 1. Contexto y objetivo

Se solicita cotización para un sistema de biorreactor de alto desempeño orientado a la producción reproducible de café de especialidad y a la generación de conocimiento de proceso. El sistema debe permitir no solo ejecutar fermentaciones controladas, sino **comprender qué ocurre dentro del biorreactor** en tiempo real, con resolución suficiente para correlacionar dinámica fermentativa con perfil de taza.

El principio de diseño central es **precisión sobre completitud**: el sistema debe medir pocas variables pero con exactitud, confiabilidad y resolución temporal que justifiquen su uso en toma de decisiones reales.

---

## 2. Alcance del suministro

- Diseño, fabricación e integración del sistema completo
- Sistema de control con firmware documentado y actualizable
- Sensores, instrumentación y actuadores según especificación
- Instalación, calificación de desempeño (PQ) y capacitación
- Documentación técnica completa (diagramas P&ID, esquemas eléctricos, protocolos de calibración)
- Soporte técnico con cobertura mínima de 3 años y tiempo de respuesta garantizado

---

## 3. Modos de operación

El sistema debe soportar en el mismo vessel, sin reconfiguraciones físicas:

- **SSF** (estado sólido, sin agua libre)
- **SSF con humectación controlada**
- **Fermentación sumergida**
- **SIAF** (Self-Induced Anaerobic Fermentation) — modo primario de operación

**Requisito de hermeticidad SIAF:** el sistema debe poder verificar hermeticidad mediante presurización con CO₂ a 0.3 bar sin pérdida detectable en 10 minutos. La válvula de alivio debe ser de precisión, con set-point configurable entre 0.05 y 0.5 bar, que permita salida de CO₂ producido sin entrada de O₂ en ninguna condición operativa.

---

## 4. Instrumentación

El sistema se basa en **cuatro variables primarias** medidas con precisión de grado analítico. Estas son las únicas variables que el sistema necesita medir para caracterizar completamente el proceso fermentativo de café.

| Variable | Justificación | Precisión requerida | Frecuencia mínima |
|---|---|---|---|
| **Temperatura** (2 puntos: superior e inferior del lecho) | Termometría del proceso + detección de gradientes | ±0.2 °C | 30 s |
| **pH** (lixiviado) | Indicador primario de avance fermentativo y punto de arresto | ±0.02 pH | 60 s |
| **Presión en headspace** | Trazador de actividad metabólica en SIAF; proxy de CO₂ acumulado | ±0.005 bar | 30 s |
| **ORP / Potencial redox** (lixiviado) | Indicador del estado redox del consorcio microbiano; complementario al pH | ±2 mV | 60 s |

**Sobre la exclusión de otros sensores:**

- **CO₂ en headspace**: reemplazado por presión de alta resolución, que en condiciones SIAF es equivalente y más robusto frente a matrices heterogéneas de café.
- **Oxígeno disuelto (DO)**: redundante en condiciones SIAF verificadas. Si la hermeticidad se cumple, el DO cae a cero en los primeros minutos y no aporta información dinámica relevante durante el proceso.
- **Conductividad, turbidez, NIR en línea**: instrumentación de mayor complejidad con calibración específica por origen y variedad de café — su valor requiere un programa de calibración que está fuera del alcance de este sistema.

Los sensores deben ser de grado industrial (no de laboratorio portátil), con cuerpo en acero inoxidable AISI 316L, compatibles con CIP, y con salida digital (protocolo abierto: Modbus, HART o equivalente documentado).

### 4.2 Arquitectura de medición: inmersión directa vs. cámara de muestreo

El proveedor debe proponer y justificar la arquitectura de medición para pH, ORP y presión. Se aceptan dos arquitecturas:

**Opción A — Inmersión directa:** sensores instalados directamente en el vessel, en contacto con el sustrato o lixiviado.

**Opción B — Cámara de muestreo externa (flow cell):** el lixiviado se extrae continuamente mediante bomba peristáltica hacia una cámara externa donde están los sensores, y se reincorpora al vessel de forma continua y cuantificada.

Si el proveedor propone la Opción B, debe especificar obligatoriamente:

- Retardo de medición entre condición real en vessel y lectura del sensor (en segundos o minutos)
- Diseño del circuito de reincorporación y su impacto sobre el balance de masa y la integridad de la presión del headspace en modo SIAF — este punto es crítico y debe resolverse sin comprometer la presión como variable de trazado
- Caudal de extracción y reincorporación, y cómo se gestiona en SSF donde el lixiviado disponible es limitado
- Procedimiento de limpieza CIP de la cámara y las líneas de muestreo
- Impacto sobre frecuencia de calibración comparado con inmersión directa

La temperatura y la presión del headspace se miden siempre mediante sensores instalados directamente en el vessel, independientemente de la arquitectura elegida para los demás sensores.

---

## 5. Control de proceso

### 5.1 Temperatura

- Control activo por camisa o serpentín con calentamiento y enfriamiento independientes
- Gradiente máximo admisible entre zona superior e inferior: **±0.5 °C** en estado estable
- Rampa de enfriamiento para arresto: mínimo **1 °C/min** hasta temperatura objetivo
- Preacondicionamiento térmico previo a carga, configurable en receta

### 5.2 Atmósfera

- Inyección de CO₂ para establecimiento de anaerobiosis inicial
- Inyección de N₂ como gas de purga
- Válvula de alivio de precisión (ver sección 3)
- **Sin inyección de O₂:** no compatible con el perfil de proceso objetivo

### 5.3 Mezcla y manejo de fases

- Agitación de baja cizalla para SSF (volteo o paletas), sin daño al grano
- Recirculación de lixiviados con caudal configurable
- Modo estático como opción explícita en receta
- Separación y drenaje independiente de fase líquida

### 5.4 Inoculación de cultivos iniciadores

- Puerto de inoculación aséptico sin ruptura de hermeticidad
- Soporte para inoculación secuencial en dos tiempos (definida en receta)
- Registro automático: ID de inóculo, volumen, timestamp

---

## 6. Recetas y lógica de control

Las recetas son el núcleo del sistema. Deben ser definibles externamente, versionables y reproducibles de forma exacta.

### 6.1 Estructura

Cada receta define una secuencia de etapas. Cada etapa especifica:

- Setpoints de temperatura, presión máxima de headspace, intensidad de mezcla
- **Condición de avance:** tiempo transcurrido **O** variable alcanzando umbral (pH, presión, ORP)
- Acciones automáticas ante desviación de setpoint: alerta, pausa, arresto

### 6.2 Arresto automático basado en proceso

El sistema debe poder ejecutar arresto de fermentación (enfriamiento activo inmediato) cuando:

- pH desciende por debajo de umbral definido en receta
- Presión supera límite de seguridad
- Cualquier combinación de condiciones configurada por el usuario

Este es el diferenciador funcional más importante del sistema frente a equipos básicos.

### 6.3 Gestión de recetas

- Formato estructurado (JSON o YAML), importable desde fuente externa
- Biblioteca local con control de versiones (nombre, versión, autor, fecha)
- Registro inmutable de qué receta ejecutó cada lote

---

## 7. Datos y conectividad

El sistema es ante todo una **plataforma de datos de proceso**. La calidad del dato es más importante que la cantidad de variables.

### 7.1 Adquisición

- Registro continuo a las frecuencias definidas en sección 4
- Marca de tiempo con sincronización NTP y resolución de 1 segundo
- Escritura en almacenamiento no volátil local; buffer mínimo de **30 días**
- Operación completamente autónoma sin internet

### 7.2 Estructura del registro

```json
{
  "batch_id": "string",
  "recipe_id": "nombre:versión",
  "timestamp": "ISO 8601",
  "sensors": {
    "temperature_top_C": float,
    "temperature_bottom_C": float,
    "pH": float,
    "pressure_bar": float,
    "ORP_mV": float
  },
  "process_state": {
    "mode": "SSF | submerged | SIAF",
    "stage_name": "string",
    "stage_elapsed_min": int
  }
}
```

La estructura debe ser estable entre versiones de firmware. Cualquier cambio en el schema debe ser versionado y documentado.

### 7.3 Acceso externo

- **API REST** para consulta en tiempo real e histórico
- **MQTT** para telemetría hacia sistemas externos (IoT / plataformas de análisis)
- Exportación por lote: serie temporal completa en CSV y JSON
- Sin dependencia de servicios de nube propietarios del proveedor para acceso a datos propios

### 7.4 Reporte de lote

Al cierre de cada lote, el sistema genera automáticamente:

- Serie temporal completa de las 5 variables medidas
- Estadísticos por etapa: media, máximo, mínimo, desviación estándar
- Log de eventos: desvíos, alertas, intervenciones del operador, arresto
- Identificador único del lote (código vinculable a resultado de catación)

---

## 8. Interfaz de operación

- Pantalla táctil local mínimo 10"
- Dashboard único con todas las variables en tiempo real y tendencia de las últimas 24 h
- Flujos guiados para inicio de lote, calibración, CIP y arresto manual
- Acceso remoto vía navegador web (red local) con capacidades idénticas a HMI local
- **Sin dependencia de app móvil** como interfaz primaria

---

## 9. Mantenimiento y calibración

### 9.1 Calibración

- Acceso directo a todos los sensores sin desmontaje
- Protocolos guiados desde HMI: pH (buffer 2 puntos), ORP (solución de referencia), presión (referencia certificada)
- Registro inmutable de cada calibración: fecha, usuario, valores pre/post
- Alertas de recalibración por tiempo definido por el proveedor según sensor

### 9.2 Limpieza CIP

- Ciclos de limpieza sin desmontaje completo
- Cobertura de: vessel, líneas, sensores
- Drenaje completo verificable
- Log de cada ciclo CIP

### 9.3 Componentes críticos

El proveedor debe especificar:

- Vida útil esperada de cada sensor (horas de operación o número de ciclos)
- Disponibilidad de repuestos en Colombia (stock local o tiempo de importación garantizado)
- Piezas que el operador puede reemplazar en campo sin herramienta especializada

---

## 10. Infraestructura y condiciones de operación

- Temperatura ambiente: 5–40 °C; HR: 20–95% sin condensación
- Gabinete de control: protección **IP65 mínimo**
- **UPS integrado:** autonomía mínima de **2 horas** para adquisición de datos y protocolo de arresto controlado
- El proveedor debe especificar: suministro eléctrico, consumo de agua, drenaje, presión y caudal de gases

---

## 11. Requisito de integración proceso–calidad de taza

El sistema debe soportar el cierre del ciclo proceso–resultado sensorial:

- El ID de lote debe ser vinculable externamente a una hoja de catación SCA
- El proveedor debe documentar el formato de exportación de datos con suficiente detalle para que un sistema externo construya esta correlación
- No se requiere integración nativa con software de catación, pero el formato de datos debe permitirla sin transformaciones propietarias

---

## 12. Información requerida al proveedor

1. Descripción de la solución y diagrama P&ID simplificado
2. Ficha técnica de cada sensor: marca, modelo, principio de medición, rango, precisión, vida útil
3. Descripción del sistema de control y lógica de recetas (lenguaje o entorno utilizado)
4. Documentación de la API y protocolo MQTT (endpoints, autenticación, schema)
5. Requerimientos de infraestructura (eléctrico, agua, gases, espacio)
6. Plan de mantenimiento: frecuencias, componentes, disponibilidad de repuestos en Colombia
7. Procedimiento de calificación de desempeño (PQ) al momento de instalación
8. Costos detallados (equipo, instalación, calificación, capacitación, soporte anual)
9. Tiempo de entrega e instalación
10. Referencias verificables en contexto de poscosecha de café o fermentación de alimentos

---

## 13. Criterios de evaluación

| Criterio | Peso |
|---|---|
| Precisión y confiabilidad de los cuatro sensores primarios | 25% |
| Robustez del modo SIAF y hermeticidad verificable | 20% |
| Calidad del sistema de datos (estructura, API, autonomía offline) | 20% |
| Funcionalidad de recetas con lógica event-driven y arresto automático | 20% |
| Soporte local, repuestos y plan de mantenimiento | 15% |
