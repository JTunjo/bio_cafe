# Referencia Técnica — Sistema Actual de Fermentación Controlada
## Biorreactor de Caneca con Instrumentación IoT

**Documento:** Línea base técnica  
**Propósito:** Descripción del sistema en operación actual. Sirve como referencia de punto de partida para la evaluación de sistemas Advanced y Ultra.  
**Estado:** En operación

---

## 1. Descripción general

El sistema consiste en una caneca plástica de **200 L** adaptada como biorreactor anaerobio de baja instrumentación, con control de temperatura por PID, monitoreo de variables de proceso mediante sensores embebidos, y gestión de recetas y visualización a través de una aplicación web local. Opera principalmente en modo **SIAF** (fermentación anaerobia autoinducida), con mejor desempeño documentado en **fermentación sumergida** que en estado sólido.

---

## 2. Vessel y contención

| Atributo | Especificación |
|---|---|
| Material | Caneca plástica (HDPE o equivalente), 200 L |
| Hermeticidad | Tapa con sello de cierre |
| Válvula de alivio | Airlock de fermentación estándar tipo cervecero — permite salida de CO₂ sin entrada de O₂ |
| Agitación / recirculación | Ninguna — proceso completamente estático |
| Modo de operación primario | SIAF sumergido |
| Modo de operación secundario | SIAF estado sólido (desempeño inferior documentado) |

**Nota operativa:** La ausencia de mecanismo de agitación o recirculación limita la homogeneidad térmica y química del sustrato, especialmente en estado sólido. En fermentación sumergida, la convección natural del lixiviado compensa parcialmente esta limitación.

---

## 3. Sistema de calentamiento

| Atributo | Especificación |
|---|---|
| Elemento calefactor | Resistencia de silicona flexible, instalada en serpentina de acero inoxidable interior |
| Función de la serpentina | Distribución del calor desde la resistencia hacia el lixiviado / sustrato |
| Enfriamiento | Pasivo — temperatura ambiente. Sin sistema activo de enfriamiento |
| Control | PID sobre temperatura — lazo cerrado único |

**Limitación:** La ausencia de enfriamiento activo impide el arresto controlado de la fermentación por reducción rápida de temperatura. El proceso se detiene por agotamiento de sustratos o por intervención manual (apertura del vessel / descarga).

---

## 4. Instrumentación

### 4.1 Sensores instalados

| Variable | Sensor | Principio | Ubicación | Observaciones |
|---|---|---|---|---|
| Temperatura de resistencia | No especificado | Contacto directo | Sobre la resistencia de silicona | Función de protección — evita sobrecalentamiento local del elemento calefactor |
| Temperatura headspace | DS18B20 | Digital 1-Wire, semiconductor | Headspace (fase gaseosa) | Resolución 0.0625 °C; precisión ±0.5 °C típica |
| Temperatura interna sustrato | DS18B20 | Digital 1-Wire, semiconductor | Sumergido en sustrato o lixiviado | Mismo sensor, segundo punto de medición |
| pH | Sonda analógica genérica (módulo tipo pH-4502C o equivalente) | Electrodo de vidrio + módulo de acondicionamiento | Sumergido en lixiviado | Ver nota técnica 1 |
| CO₂ headspace | CCS811 | MOX (Metal Oxide Semiconductor) | Headspace | Ver nota técnica 2 — limitación crítica |
| Sólidos disueltos / EC | Sensor TDS/EC genérico | Conductividad eléctrica | Sumergido en lixiviado | Proxy de concentración de solutos en lixiviado |

---

### 4.2 Notas técnicas críticas sobre sensores

**Nota 1 — Sensor de pH genérico (módulo analógico):**
Los módulos analógicos de pH tipo 4502C presentan deriva significativa con temperatura y humedad, y requieren calibración frecuente (recomendada cada 2–3 lotes o ante cambio de temperatura ambiente > 5 °C). La precisión real en campo es típicamente ±0.1–0.2 pH, inferior a la precisión nominal del electrodo en condiciones de laboratorio. En el contexto de esta aplicación, el pH es aceptable como indicador de tendencia pero no como valor absoluto de alta confianza sin calibración reciente verificada.

**Nota 2 — Sensor de CO₂: CCS811 (limitación crítica):**
El CCS811 **no mide CO₂ real**. Es un sensor de calidad de aire que detecta compuestos orgánicos volátiles (VOCs) y calcula un valor de **CO₂ equivalente (eCO₂)** mediante algoritmo interno. En el headspace de una fermentación de café, el ambiente está cargado de etanol, ésteres y otros VOCs producidos por el metabolismo microbiano — exactamente los compuestos que confunden al CCS811 y generan lecturas de eCO₂ artificialmente elevadas e incorrelacionadas con el CO₂ real.

Consecuencias prácticas:
- Los valores de CO₂ registrados por este sistema **no son comparables** con mediciones de CO₂ real (sensores NDIR como MH-Z19, SCD30, o Sensirion SCD4x).
- La curva de "CO₂" en el historial de lotes de este sistema refleja principalmente la actividad de VOCs, no la cinética de producción de CO₂.
- Para correlacionar estos datos con los de sistemas Advanced o Ultra (que usan presión o CO₂ NDIR), se requiere una campaña de medición paralela con sensor de referencia.

**Recomendación de mejora inmediata de bajo costo:** reemplazar el CCS811 por un sensor NDIR de bajo costo (MH-Z19B ~15 USD, SCD40 ~35 USD) para obtener medición real de CO₂ en headspace sin cambiar ningún otro componente del sistema.

---

## 5. Sistema de control y computación

| Componente | Especificación |
|---|---|
| Microcontrolador de campo | STM32 (familia STM32, modelo específico no documentado) |
| Función del STM32 | Lectura de sensores, ejecución de PID de temperatura, control de actuadores, comunicación con gateway |
| Computación central | Raspberry Pi 3 Model B+ |
| Función de la Raspberry Pi | Procesamiento de datos, gestión de recetas, almacenamiento local, servidor web |
| Framework de aplicación | Flask (Python) — servidor web local |
| Comunicación campo → gateway | LoRaWAN |
| Acceso de operador | Aplicación web vía WiFi desde dispositivo móvil |

**Arquitectura de comunicación:**
```
Sensores → STM32 → LoRaWAN → Raspberry Pi 3B+ → Flask App → Navegador móvil (WiFi)
```

---

## 6. Gestión de recetas

| Atributo | Especificación |
|---|---|
| Tipo de receta | Multietapa con duraciones definidas |
| Transición entre etapas | Manual |
| Transición por variable (event-driven) | No disponible — solo por tiempo |
| Controles de receta | Solo por temperatura (event-driven) |
| Configuración | Desde aplicación web en dispositivo móvil |
| Visualización | Dashboard en navegador móvil vía WiFi |

---

## 7. Capacidades y limitaciones consolidadas

### Lo que el sistema hace bien
- Control de temperatura por PID con elemento calefactor dedicado
- Registro de múltiples variables simultáneas
- Acceso remoto desde móvil sin infraestructura adicional
- Bajo costo de fabricación y componentes accesibles localmente
- LoRaWAN permite despliegue en campo sin WiFi en el punto del vessel

### Limitaciones documentadas

| Limitación | Impacto | Solución en Advanced/Ultra |
|---|---|---|
| Sin enfriamiento activo | No permite arresto controlado de fermentación | Camisa/serpentín con enfriamiento activo |
| CO₂ medido con CCS811 (eCO₂, no real) | Datos de CO₂ no confiables ni comparables | Sensor NDIR o presión de alta resolución |
| pH con módulo analógico genérico | Precisión real ±0.1–0.2 pH; alta deriva | Electrodo industrial con salida digital |
| Sin ORP | No hay caracterización del estado redox | ORP en Advanced y Ultra |
| Sin DO | No hay información de oxigenación | DO en Advanced y propagador de Ultra |
| Proceso estático | Gradientes térmicos y químicos en SSF | Recirculación y/o agitación en Advanced/Ultra |
| Desempeño inferior en SSF | Limitación en agitación, enfriado de lixiviados y recirculación | Mecanismo de humectación y mezcla en Advanced/Ultra |
| Transición de etapas solo por tiempo | No hay arresto automático por pH o presión | Lógica event-driven en Advanced/Ultra |
| Vessel plástico | Limpieza más difícil; no apto para CIP con químicos agresivos | Vessel AISI 316L en Advanced/Ultra |

---

## 8. Observaciones para uso como referencia comparativa

Este sistema representa el **mínimo funcional viable** para fermentación SIAF instrumentada. Ha demostrado capacidad para producir café de especialidad en modo sumergido, lo que valida el concepto de fermentación controlada con instrumentación básica.

Los sistemas Advanced y Ultra propuestos en este proceso de RFQ deben evaluarse en términos de qué limitaciones específicas de este sistema corrigen, y si esa corrección justifica el diferencial de costo en el contexto de producción objetivo.

Las variables de mayor impacto potencial en calidad de taza que este sistema no captura correctamente son, en orden de prioridad estimada:

1. **CO₂ real** (el CCS811 no lo mide)
2. **Arresto por pH** (no hay lógica event-driven)
3. **Enfriamiento activo** (no hay arresto térmico controlado)
4. **ORP** (no hay caracterización del estado redox)
