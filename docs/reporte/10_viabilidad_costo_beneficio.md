<!-- Ruta: docs/reporte/10_viabilidad_costo_beneficio.md · Responsable: A (Stephanny Elizabeth Torres Romo) · Fase 1 -->

# Bloque de Ingeniería de Software y SQLite

## Estudio de viabilidad

### Planteamiento del problema

Una comercializadora de electrodomésticos de tamaño pequeño o mediano en Monclova, Coahuila, que vende en mostrador y entrega a domicilio, suele llevar el inventario en hojas de cálculo y planear los repartos a mano. Ese esquema genera tres problemas recurrentes: (a) ventas de productos sin existencia real porque el inventario no se descuenta en el momento de la venta, (b) despachos sin un criterio explícito de urgencia y (c) rutas de reparto que dependen de la experiencia del chofer y no se ajustan cuando una vialidad se cierra.

ElectroLogística es una aplicación de escritorio que integra ventas, inventario por almacén, despacho priorizado, asignación de carga a vehículos y cálculo de rutas, con SQLite como fuente de verdad. Para el análisis se define un **caso de referencia** (Tabla 1) que sirve de base a las cifras de beneficio. No describe a una empresa real: es un escenario supuesto, plausible para el mercado local, y cada valor puede sustituirse por datos de un cliente concreto sin cambiar el método.

**Tabla 1**  
*Caso de referencia supuesto para el análisis*

| Parámetro | Valor supuesto | Razonamiento |
|---|---|---|
| Almacenes | 2 centrales + 1 sucursal | Escala de una cadena local pequeña |
| Vehículos de reparto | 5 camionetas | Coherente con los datos de prueba (`DML.sql`) |
| Recorrido por vehículo | 120 km/día | Repartos urbanos y a colonias periféricas de Monclova y Frontera |
| Días de operación | 26 días/mes | Lunes a sábado |
| Rendimiento de combustible | 8 km/L | Camioneta de reparto cargada |
| Precio de gasolina regular | $24.00 MXN/L | Precio de referencia supuesto; verificar en estaciones de Monclova |
| Planeación manual de rutas | 2 h/día de un auxiliar de logística | Asignar pedidos y ordenar paradas a mano |
| Incidencias por stock erróneo | 2 por mes, $1,500 MXN cada una | Margen perdido + reenvío o cancelación |

### Viabilidad técnica

**Tabla 2**  
*Evaluación de la viabilidad técnica*

| Aspecto | Evaluación | Conclusión |
|---|---|---|
| Lenguaje y plataforma | C# sobre .NET 10 (LTS, soporte de Microsoft hasta noviembre de 2028), Windows Forms para la ventana, consola multiplataforma | Tecnología madura y gratuita, documentada en Microsoft Learn |
| Base de datos | SQLite embebida mediante `Microsoft.Data.Sqlite`. No requiere servidor, admite transacciones ACID, llaves foráneas, restricciones CHECK y respaldo en línea (SQLite Consortium, s. f.) | Adecuada para un solo puesto de trabajo o pocos usuarios con escrituras poco concurrentes |
| Estructuras de datos | Pila, montículo binario, ABB, grafo con Dijkstra y ordenamientos implementados desde cero; complejidades conocidas (Cormen et al., 2022) | Volúmenes esperados (≤ 100,000 productos, ≤ 1,000 nodos viales) dentro de lo que se procesa en milisegundos |
| Hardware mínimo | Equipo con Windows 10/11 de 64 bits, 8 GB de RAM y 1 GB libre en disco | Los equipos del equipo y los de una tienda típica lo cumplen |
| Competencias del equipo | Programación orientada a objetos, estructuras de datos y SQL vistos en el plan de estudios; .NET y SQLite se aprenden con la documentación oficial | Riesgo de curva de aprendizaje mitigado en la matriz de riesgos (R-01) |
| Limitaciones | SQLite admite un escritor a la vez; la app de ventana solo corre en Windows | Aceptables para el alcance; la migración a un servidor (PostgreSQL) y la web quedan como trabajo futuro |

**Dictamen técnico: viable.** Todo el desarrollo puede hacerse con herramientas gratuitas, y las restricciones de SQLite no afectan al alcance de un puesto de venta con reparto propio.

### Viabilidad económica

#### Supuestos de costo

**Tabla 3**  
*Supuestos de costo y forma de verificarlos*

| Concepto | Supuesto | Fuente o razonamiento | Cómo verificar |
|---|---|---|---|
| Tarifa de desarrollador junior | **$125 MXN/h** (costo para la empresa) | Sueldo bruto supuesto de $16,000 a $20,000 MXN mensuales para un desarrollador junior .NET en la región Monclova–Saltillo, dividido entre 176 h/mes ≈ $91–114/h, más ≈ 25–30 % de cargas (IMSS, INFONAVIT, aguinaldo, vacaciones y prima vacacional según la Ley Federal del Trabajo) | Capturar tres vacantes reales con fecha en OCC Mundial o Indeed México y recalcular |
| Horas de desarrollo | **354 h** en total (Tabla 4) | Estimación ascendente por fase y perfil a partir de los entregables de la guía | Comparar con las horas reales registradas en los commits y en el seguimiento semanal |
| Equipo de cómputo | 3 laptops propias de ≈ $15,000 MXN cada una; se imputa depreciación | Tasa de depreciación fiscal del 30 % anual para computadoras personales (Ley del Impuesto sobre la Renta, art. 34, fracc. VII): $15,000 × 0.30 ÷ 12 = $375 por equipo y mes | Factura o precio de mercado del equipo |
| Energía eléctrica | 65 W promedio por laptop × 354 h ≈ 23 kWh, más ≈ 7 kWh de router y periféricos ≈ **30 kWh** a $2.80 MXN/kWh | Tarifa doméstica de CFE en su escalón intermedio-excedente; valor supuesto | Recibo de CFE de cada integrante |
| Internet | 30 % de un plan de $500 MXN/mes en cada uno de los 3 hogares = **$450 MXN** | Proporción de uso dedicada al proyecto durante un mes | Recibo del proveedor |
| Licencias | **$0** | .NET SDK (MIT), Visual Studio Community (gratuita para uso académico e individual), SQLite (dominio público), Microsoft.Data.Sqlite (MIT), xUnit (Apache 2.0), Git y GitHub (plan gratuito), Python y matplotlib, Pandoc, Mermaid | Términos de licencia de cada producto |
| Hosting | **$0** | Aplicación de escritorio; el repositorio vive en GitHub sin costo | No aplica |
| Capacitación del cliente | 8 h × $125 = **$1,000 MXN** | Dos sesiones de 4 h con vendedores, almacén y despacho | Lista de asistencia |
| Mantenimiento | 4 h/mes × $125 = **$500 MXN/mes** | Correcciones menores, respaldos y ajustes de catálogo | Bitácora de soporte |
| Contingencia | **10 %** de la inversión | Holgura habitual para desviaciones de estimación (Pressman y Maxim, 2020) | No aplica |

**Tabla 4**  
*Estimación de horas por fase y perfil*

| Fase | A | B | C | Total |
|---|---:|---:|---:|---:|
| 1. Viabilidad, requisitos y gestión | 16 | 16 | 16 | 48 |
| 2. Arquitectura, base de datos y TDA | 18 | 18 | 20 | 56 |
| 3. Despacho, ventas y auditoría | 14 | 22 | 22 | 58 |
| 4. Catálogo e indexación con ABB | 12 | 20 | 14 | 46 |
| 5. Red logística y ruteo | 26 | 10 | 12 | 48 |
| 6. Ordenamiento y rendimiento | 18 | 12 | 16 | 46 |
| 7. Integración y cierre | 16 | 18 | 18 | 52 |
| **Total** | **120** | **116** | **118** | **354** |

Las 118 h promedio por integrante, repartidas en los 32 días del cronograma, equivalen a unas 3.7 h diarias, compatibles con la carga del semestre.

#### Costos

**Tabla 5**  
*Inversión inicial y costo de operación (MXN)*

| Concepto | Cálculo | Importe |
|---|---|---:|
| Desarrollo | 354 h × $125 | 44,250 |
| Depreciación de equipo | 3 × $375 × 1 mes | 1,125 |
| Energía eléctrica | 30 kWh × $2.80 | 84 |
| Internet | 3 × $500 × 30 % | 450 |
| Licencias y hosting | — | 0 |
| Capacitación | 8 h × $125 | 1,000 |
| Subtotal | | 46,909 |
| Contingencia 10 % | | 4,691 |
| **Inversión inicial** | | **51,600** |
| **Costo anual de operación** | $500 × 12 (mantenimiento) | **6,000** |

#### Beneficios

**Tabla 6**  
*Beneficios mensuales estimados (MXN)*

| Beneficio | Cálculo | Importe mensual |
|---|---|---:|
| Ahorro de combustible por rutas de menor costo (10 %) | 5 × 120 km × 26 d = 15,600 km ÷ 8 km/L = 1,950 L × $24 = $46,800 × 10 % | 4,680 |
| Horas de planeación ahorradas | De 2 h a 0.5 h diarias: 1.5 h × 26 d × $70/h (auxiliar de logística con ≈ $12,000 MXN/mes ÷ 176 h) | 2,730 |
| Menos incidencias de stock (−50 %) | 2 × $1,500 × 50 % | 1,500 |
| **Total** | | **8,910** |
| **Total anual** | × 12 | **106,920** |

El 10 % de ahorro en combustible es un supuesto conservador: la ruta propuesta encadena caminos mínimos de Dijkstra con la heurística del vecino más cercano, que no garantiza el óptimo pero sí evita los recorridos redundantes de una planeación manual (Cormen et al., 2022).

#### Indicadores

Con una tasa de descuento supuesta del 12 % anual (costo de oportunidad de una PyME: tasa de referencia más una prima de riesgo) y un horizonte de 3 años, el factor de valor presente de una anualidad es (1 − 1.12⁻³) ÷ 0.12 = 2.4018.

**Tabla 7**  
*Indicadores económicos (horizonte de 3 años)*

| Indicador | Cálculo | Resultado |
|---|---|---:|
| Beneficio neto anual | 106,920 − 6,000 | $100,920 |
| Periodo de recuperación | 51,600 ÷ (100,920 ÷ 12) | 6.1 meses |
| ROI del primer año | (100,920 − 51,600) ÷ 51,600 | 95.6 % |
| VP de beneficios | 106,920 × 2.4018 | $256,804 |
| VP de costos | 51,600 + 6,000 × 2.4018 | $66,011 |
| VPN | 256,804 − 66,011 | $190,793 |
| Relación beneficio/costo | 256,804 ÷ 66,011 | 3.89 |

**Tabla 8**  
*Análisis de sensibilidad*

| Escenario | Beneficio anual | Recuperación | B/C (3 años) |
|---|---:|---:|---:|
| Base | $106,920 | 6.1 meses | 3.89 |
| Beneficios −50 % | $53,460 | 13.0 meses | 1.95 |
| Tarifa de desarrollo +40 % ($175/h; inversión $71,070) | $106,920 | 8.5 meses | 3.00 |
| Pesimista combinado (−50 % y +40 %) | $53,460 | 18.0 meses | 1.50 |

El aumento de tarifa se aplica solo a las horas de desarrollo: (354 × 175 + 1,125 + 84 + 450 + 1,000) × 1.10 = $71,070. En el pesimista combinado, VP de costos = 71,070 + 14,411 = 85,481 y VP de beneficios = 53,460 × 2.4018 = 128,402, de modo que B/C = 1.50. La recuperación sale de 71,070 ÷ ((53,460 − 6,000) ÷ 12) = 18.0 meses.

**Dictamen económico: viable.** Incluso en el escenario pesimista la relación beneficio/costo supera 1 y la inversión se recupera en unos 18 meses, dentro del horizonte de evaluación de 3 años. Como proyecto académico el desembolso real del equipo se limita a energía e internet (≈ $534 MXN); el resto es costo de oportunidad.

### Viabilidad operativa

**Tabla 9**  
*Usuarios, cambios en su trabajo y aceptación esperada*

| Usuario | Qué hace en el sistema | Cambio respecto al proceso actual | Aceptación esperada |
|---|---|---|---|
| Vendedor | Registra ventas con validación de stock en línea; cancela y deshace | Deja de consultar existencias por teléfono | Alta: menos rechazos al cliente |
| Almacenista | Consulta el catálogo por SKU, filtra por marca y precio, registra devoluciones | Búsqueda inmediata en lugar de hojas de cálculo | Alta |
| Despachador | Ve la cola de pedidos por prioridad, asigna lotes a vehículos y reasigna | El criterio de prioridad queda explícito y es el mismo para todos | Media: requiere confiar en el criterio automático; se mitiga mostrando el puntaje |
| Transportista | Recibe la hoja de ruta con el orden de paradas | Sigue un orden calculado | Media: se permite recalcular al cerrar un tramo |
| Gerente | Respaldos, verificación de integridad, indicadores | Nuevo control | Alta |

Factores a favor: interfaz en español con pestañas por proceso, operación sin conexión a internet (base local), capacitación breve (Tabla 3) y la posibilidad de deshacer errores incluso después de cerrar la aplicación. Factores en contra: resistencia al cambio de quien planea rutas por experiencia y dependencia de un solo equipo con Windows. Ambos se tratan en la matriz de riesgos (R-09 y R-11).

**Dictamen operativo: viable**, condicionado a la capacitación y a un periodo de operación en paralelo de una semana.

### Viabilidad legal

**Tabla 10**  
*Marco legal aplicable y cumplimiento*

| Marco | Implicación para el sistema | Medida adoptada |
|---|---|---|
| Ley Federal de Protección de Datos Personales en Posesión de los Particulares (DOF, 20 de marzo de 2025) | Nombre, teléfono, correo y domicilio de clientes son datos personales; se requiere aviso de privacidad, finalidad limitada y medidas de seguridad | Solo se guardan los datos indispensables para la entrega; la base local se respalda cifrada fuera del equipo; el README incluye un modelo de aviso de privacidad para el comercio; los datos de prueba son ficticios |
| Ley Federal de Protección al Consumidor | Información veraz de precios, respeto al precio ofertado y atención de devoluciones y garantías | Se guarda el precio unitario al momento de la venta (`detalle_pedido`), el historial de precios y el flujo de devolución con su historial de estados |
| Ley Federal del Derecho de Autor | El software es obra protegida de sus autores | El repositorio declara autoría por integrante; el registro ante INDAUTOR es opcional |
| Licencias de terceros | MIT (.NET, Microsoft.Data.Sqlite), dominio público (SQLite), Apache 2.0 (xUnit), licencia de matplotlib; términos de Visual Studio Community | Todas permiten uso académico y comercial; se conservan avisos de licencia; no se distribuye Visual Studio |
| Obligaciones fiscales (CFDI) | Una venta real requiere factura electrónica emitida mediante un proveedor autorizado | Fuera de alcance: el sistema registra la venta y deja el CFDI como integración futura |

**Dictamen legal: viable**, siempre que el comercio publique el aviso de privacidad y que la facturación se resuelva con su sistema actual.

### Conclusión del estudio de viabilidad

Los cuatro dictámenes son favorables. El proyecto usa herramientas gratuitas, recupera la inversión estimada en unos 6 meses en el caso base (unos 18 en el pesimista), encaja en la operación de un comercio local con una capacitación breve y cumple el marco legal con medidas concretas. Se aprueba continuar con el análisis de requerimientos y el diseño.
