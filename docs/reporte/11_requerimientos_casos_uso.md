<!-- Ruta: docs/reporte/11_requerimientos_casos_uso.md · Responsable: B (Bárbara Sofía Pérez Romo) · Fase 1 -->

## Especificación de requerimientos

Los requerimientos siguen la clasificación de Sommerville (2016): los funcionales describen lo que el sistema hace y los no funcionales las propiedades que debe cumplir. La prioridad usa MoSCoW: **M** = obligatorio, **S** = debería, **C** = podría. Cada requerimiento indica el componente del contrato técnico que lo implementa y la prueba que lo verifica, para mantener la trazabilidad entre requerimientos, diseño, código y pruebas.

### Requerimientos funcionales

**Tabla 11**  
*Requerimientos funcionales de venta y carrito/pedido*

| ID | Requerimiento | Prioridad | Implementa | Verifica |
|---|---|:-:|---|---|
| RF-01 | Registrar una venta con uno o más renglones (SKU, cantidad) para un cliente, un almacén de origen, una urgencia de 1 a 5 y un tipo de entrega (estándar, express o programada) | M | `ServicioVentas.RegistrarVenta`, `IRegistroVentasDao.RegistrarVenta` | `RegistroVentasDaoTests` |
| RF-02 | Validar antes de confirmar que cada SKU existe, está activo y tiene stock suficiente en el almacén elegido; si un renglón falla, no se registra nada | M | `SolicitudVenta.Validar`, `IInventarioDao.Descontar` | `RegistroVentasDaoTests`, `CasosLimiteTests` |
| RF-03 | Guardar el precio unitario vigente al momento de la venta en el detalle del pedido | M | `detalle_pedido.precio_unitario_centavos` | `RegistroVentasDaoTests` |
| RF-04 | Generar folio de pedido y número de guía únicos al confirmar la venta | M | `GeneradorFolios` | `RegistroVentasDaoTests` |
| RF-05 | Cancelar un pedido no despachado y devolver su stock | M | `ServicioVentas.CancelarPedido` | `DeshacerPersistenteTests` |
| RF-06 | Deshacer la última operación revertible (venta, cancelación, devolución o reasignación) aplicando su operación inversa, incluso después de cerrar y reabrir la aplicación | M | `ServicioDeshacer`, `PilaDeshacer`, `bitacora_operaciones` | `DeshacerPersistenteTests` |
| RF-07 | Recibir solicitudes de venta en un buffer de recepción y procesarlas en orden de llegada, avisando cuando el buffer esté lleno | S | `ServicioRecepcion`, `ColaCircular`, `ListaEnlazadaCola` | `ColaCircularTests`, `ListaEnlazadaColaTests` |
| RF-08 | Consultar un pedido con sus renglones, totales e historial de estados | M | `ServicioVentas.ObtenerPedido`, `Historial` | `IntegracionFlujoCompletoTests` |

**Tabla 12**  
*Requerimientos funcionales de almacén y catálogo*

| ID | Requerimiento | Prioridad | Implementa | Verifica |
|---|---|:-:|---|---|
| RF-09 | Buscar un electrodoméstico por SKU en memoria en O(log n) | M | `ServicioCatalogo.BuscarPorSku`, `ArbolBinarioBusqueda` | `ArbolBinarioBusquedaTests` |
| RF-10 | Filtrar el catálogo por rango de precio, marca, categoría y texto del nombre mediante consultas parametrizadas | M | `ICatalogoDao.Filtrar`, `v_catalogo` | `CatalogoDaoTests` |
| RF-11 | Listar el catálogo en recorrido inorden (orden de SKU), preorden y postorden | S | `ServicioCatalogo.Recorrer*` | `ArbolBinarioBusquedaTests` |
| RF-12 | Consultar existencias de un producto en cada almacén | M | `ServicioCatalogo.Existencias` | `CatalogoDaoTests` |
| RF-13 | Dar de alta un producto con stock inicial, cambiar su precio conservando el historial y darlo de baja | M | `ServicioCatalogo.AgregarProducto`, `CambiarPrecio`, `DarDeBaja` | `CatalogoDaoTests` |
| RF-14 | Registrar la devolución de un pedido entregado y reingresar su stock | M | `ServicioVentas.DevolverPedido` | `DeshacerPersistenteTests` |

**Tabla 13**  
*Requerimientos funcionales de despacho y reparto*

| ID | Requerimiento | Prioridad | Implementa | Verifica |
|---|---|:-:|---|---|
| RF-15 | Mantener la cola de pedidos pendientes ordenada por prioridad (urgencia y tipo de entrega; a igualdad, el más antiguo primero) | M | `ColaDespacho`, `MonticuloBinario`, `CalculadoraPrioridad` | `MonticuloBinarioTests`, `ColaDespachoTests` |
| RF-16 | Asignar un lote de pedidos de mayor prioridad a los vehículos con First-Fit Decreasing, respetando capacidad de volumen y de peso | M | `ServicioAsignacion`, `FirstFitDecreasing` | `FirstFitDecreasingTests` |
| RF-17 | Reasignar un envío a otro vehículo con capacidad disponible | M | `ServicioDespacho.ReasignarEnvio` | `DeshacerPersistenteTests` |
| RF-18 | Rastrear un envío por número de guía | M | `ServicioCatalogo.BuscarGuia` | `ArbolBinarioBusquedaTests` |
| RF-19 | Calcular el camino de menor costo entre dos nodos de la red y mostrar la traza de Dijkstra en tabla, exportable a CSV | M | `Dijkstra`, `ExportadorTrazaDijkstra` | `DijkstraTests` |
| RF-20 | Planificar la ruta de un vehículo con varias paradas (vecino más cercano sobre distancias de Dijkstra) y guardarla en `ruta_paradas` | M | `ServicioRutas.PlanificarRutaVehiculo` | `VecinoMasCercanoTests`, `RutaDaoTests` |
| RF-21 | Informar de forma explícita los destinos inalcanzables | M | `ResultadoCamino.Inalcanzable` | `ReconstructorCaminoTests` |
| RF-22 | Generar la hoja de ruta del transportista con el recorrido completo, costo y distancia de cada tramo | M | `GeneradorHojaDeRuta` | `ReconstructorCaminoTests` |
| RF-23 | Cerrar, reabrir o cambiar el costo de un tramo y recalcular únicamente las rutas afectadas | M | `ServicioRutas.CerrarTramo`, `CambiarCostoTramo`, `RuteadorDinamico` | `RuteadorDinamicoTests` |
| RF-24 | Marcar pedidos en ruta y entregados | S | `ServicioDespacho.MarcarEnRuta`, `MarcarEntregado` | `IntegracionFlujoCompletoTests` |
| RF-25 | Respaldar la base de datos y verificar su integridad | S | `ServicioRespaldo` | `RespaldoTests` |
| RF-26 | Ejecutar escenarios de prueba y benchmarks desde la consola y exportar resultados a CSV | M | `Electro.Consola`, `BancoBenchmarks`, `ExportadorCsv` | `EstadisticaCsvTests` |

### Requerimientos no funcionales

**Tabla 14**  
*Requerimientos no funcionales*

| ID | Categoría | Requerimiento medible | Verifica |
|---|---|---|---|
| RNF-01 | Integridad | Ninguna operación deja datos a medias: toda escritura ocurre en una transacción y la memoria solo cambia después del COMMIT | `RecuperacionTransaccionTests` |
| RNF-02 | Integridad | Las reglas RI-01 a RI-12 se garantizan en la base (PK, FK, CHECK, índices únicos) y no solo en el código | `RestriccionesCheckTests`, `IntegridadReferencialTests` |
| RNF-03 | Rendimiento | Búsqueda por SKU en memoria con altura del ABB ≤ ⌈log₂(n+1)⌉ tras la carga; con 100,000 productos la altura es ≤ 17 | `ConstruccionBalanceadaTests` |
| RNF-04 | Rendimiento | Inserción y extracción de la cola de despacho en O(log n); eliminación de un pedido concreto en O(log n) | `MonticuloBinarioTests` |
| RNF-05 | Rendimiento | Ordenamiento de 100,000 pedidos en O(n log n) también con claves muy repetidas (urgencia) | `QuickSort3ViasTests`, benchmark |
| RNF-06 | Recuperación | Tras un cierre inesperado, al reabrir se reconstruyen ABB, cola, pila y grafo desde SQLite y coinciden con la base | `CargaMemoriaTests` |
| RNF-07 | Memoria | Los nodos eliminados de pilas, colas, montículos y árboles quedan sin referencias y el GC los libera; toda conexión, comando, transacción y lector de SQLite se libera con `using` | `MemoriaTests`, procedimiento de Visual Studio |
| RNF-08 | Robustez | Ninguna estructura desborda la pila de llamadas con 100,000 elementos, incluso con datos ordenados | `QuickSort3ViasTests`, `ArbolBinarioBusquedaTests` |
| RNF-09 | Mantenibilidad | Toda la lógica vive en `Electro.Core`; App y Consola solo llaman a servicios. `Electro.App` no contiene SQL ni lógica de negocio | `ArquitecturaTests` |
| RNF-10 | Restricción de implementación | `Electro.Core` no usa colecciones de `System.Collections*` (salvo `IEnumerable<T>`) ni LINQ | `ArquitecturaTests` |
| RNF-11 | Portabilidad | Core, Consola y Tests compilan en Windows, Linux y macOS (`net10.0`); App requiere Windows | `dotnet build` |
| RNF-12 | Usabilidad | La interfaz está en español, con una pestaña por proceso; cada error muestra un mensaje claro (stock insuficiente, entrada inválida, operación no reversible) | Revisión con escenarios de la Tabla 17 |
| RNF-13 | Disponibilidad | Opera sin conexión a internet | Ejecución sin red |
| RNF-14 | Seguridad y privacidad | Solo se guardan los datos personales necesarios para la entrega y los datos de prueba son ficticios | Revisión del DML |
| RNF-15 | Calidad | Cada fase de código (3 a 6) entrega pruebas unitarias; la suite completa pasa con `dotnet test` antes de cada merge a `main` | Historial de PR |

### Actores

**Tabla 15**  
*Actores del sistema*

| Actor | Tipo | Descripción |
|---|---|---|
| Vendedor | Primario | Registra, cancela y deshace ventas |
| Almacenista | Primario | Consulta y mantiene el catálogo, registra devoluciones |
| Despachador | Primario | Prioriza, asigna y reasigna envíos; planifica rutas; gestiona tramos |
| Transportista | Primario | Usa la hoja de ruta y reporta entregas |
| Gerente | Primario | Respalda y verifica la base |
| Probador (QA) | Secundario | Ejecuta escenarios y benchmarks desde la consola |
| SQLite | Sistema externo | Persistencia y fuente de verdad |

### Catálogo de casos de uso

El diagrama está en `docs/diagramas/casos_uso.md`.

**Tabla 16**  
*Casos de uso por proceso*

| ID | Caso de uso | Proceso | Actor | RF |
|---|---|---|---|---|
| CU-01 | Registrar venta | Venta | Vendedor | RF-01–04 |
| CU-02 | Cancelar pedido | Venta | Vendedor | RF-05 |
| CU-03 | Deshacer última operación | Venta / auditoría | Vendedor, Despachador | RF-06 |
| CU-04 | Recibir y procesar solicitudes | Venta | Vendedor | RF-07 |
| CU-05 | Buscar producto por SKU y filtrar catálogo | Almacén | Almacenista | RF-09–12 |
| CU-06 | Administrar producto | Almacén | Almacenista | RF-13 |
| CU-07 | Registrar devolución | Almacén | Almacenista | RF-14 |
| CU-08 | Despachar lote priorizado a vehículos | Despacho | Despachador | RF-15, RF-16 |
| CU-09 | Reasignar envío | Despacho | Despachador | RF-17 |
| CU-10 | Rastrear paquete por guía | Paquetería | Despachador, Vendedor | RF-18 |
| CU-11 | Planificar ruta y generar hoja de ruta | Paquetería | Despachador, Transportista | RF-19–22 |
| CU-12 | Cerrar tramo o cambiar su costo (ruteo dinámico) | Paquetería | Despachador | RF-23 |
| CU-13 | Marcar en ruta y entregado | Paquetería | Transportista | RF-24 |
| CU-14 | Respaldar y verificar la base | Administración | Gerente | RF-25 |
| CU-15 | Ejecutar escenarios y benchmarks | Pruebas | Probador | RF-26 |

### Especificación de casos de uso principales

#### CU-01 Registrar venta

- **Actor:** Vendedor. **Precondiciones:** el cliente y el almacén existen; la aplicación cargó la memoria desde SQLite.
- **Flujo principal:**
  1. El vendedor elige cliente, almacén, urgencia y tipo de entrega.
  2. Agrega renglones por SKU; el sistema muestra cada producto desde el ABB.
  3. Confirma la venta.
  4. El sistema abre una transacción: valida stock, inserta pedido y detalle con el precio vigente, descuenta inventario, registra el estado `PENDIENTE` en el historial, crea el envío con guía y escribe la operación `VENTA` en la bitácora.
  5. Tras el COMMIT, el sistema encola el pedido, inserta la guía en el ABB de guías y apila la operación.
  6. Muestra folio, guía y total.
- **Flujos alternos:** 2a. El SKU no existe o está dado de baja: se rechaza el renglón. 4a. El stock no alcanza: ROLLBACK, se informa el SKU, lo solicitado y lo disponible, y la memoria queda igual. 4b. Error de base de datos: ROLLBACK y mensaje; no se registra nada.
- **Postcondición:** la venta existe en SQLite y en las estructuras en memoria, o no existe en ninguna de las dos.

#### CU-03 Deshacer última operación

- **Actor:** Vendedor o Despachador. **Precondición:** la pila de deshacer no está vacía.
- **Flujo principal:**
  1. El actor ve la operación del tope (tipo, folio, fecha).
  2. Confirma.
  3. El sistema ejecuta la operación inversa en una transacción propia y marca el registro como `REVERTIDA`.
  4. Tras el COMMIT hace pop y actualiza cola, ABB de guías y cargas de vehículo.
- **Flujos alternos:** 1a. Pila vacía: el botón está deshabilitado y la consola informa «no hay operaciones». 3a. El pedido ya va en ruta o fue entregado: `OperacionNoReversibleException`, sin cambios. 3b. Deshacer una cancelación sin stock suficiente: ROLLBACK, se informa el faltante y la operación permanece en la pila.
- **Postcondición:** el estado es el mismo que antes de la operación revertida; esto se cumple aunque la aplicación se haya cerrado entre la operación y el deshacer.

#### CU-07 Registrar devolución (almacén)

- **Actor:** Almacenista. **Precondición:** el pedido está `ENTREGADO`.
- **Flujo principal:**
  1. El almacenista busca el pedido por guía (ABB) o por folio.
  2. Captura el motivo.
  3. En una transacción, el sistema reingresa el stock al almacén de origen, cambia pedido y envío a `DEVUELTO`, registra el historial y escribe `DEVOLUCION` en la bitácora.
  4. Tras el COMMIT apila la operación.
- **Flujos alternos:** 1a. La guía no existe: mensaje «guía no encontrada». 3a. El pedido no está entregado: se rechaza con su estado actual.

#### CU-08 Despachar lote priorizado a vehículos

- **Actor:** Despachador. **Precondición:** hay pedidos `PENDIENTE` y vehículos activos.
- **Flujo principal:**
  1. El despachador indica el tamaño máximo del lote.
  2. El sistema toma de la cola los pedidos de mayor prioridad, sin extraerlos todavía.
  3. Aplica First-Fit Decreasing: ordena los paquetes por volumen descendente y coloca cada uno en el primer vehículo con volumen y peso libres suficientes.
  4. En una transacción inserta las asignaciones y cambia los pedidos a `ASIGNADO`.
  5. Tras el COMMIT quita de la cola los asignados y suma la carga a cada vehículo.
  6. Muestra la asignación y los paquetes que no cupieron, que siguen en la cola.
- **Flujos alternos:** 2a. Cola vacía: mensaje y fin. 3a. Ningún paquete cabe: se informa y no se escribe nada.

#### CU-11 Planificar ruta y generar hoja de ruta (paquetería)

- **Actor:** Despachador; Transportista como receptor. **Precondición:** el vehículo tiene envíos asignados.
- **Flujo principal:**
  1. El despachador elige el vehículo.
  2. El sistema ejecuta Dijkstra desde el almacén base y, en cada paso, elige como siguiente parada el destino pendiente más cercano (vecino más cercano), ejecutando Dijkstra desde la parada actual.
  3. Reconstruye recursivamente cada camino desde el arreglo de predecesores.
  4. Guarda la ruta y sus paradas en una transacción.
  5. Muestra la hoja de ruta con el orden, el recorrido nodo a nodo, el costo y la distancia por tramo y los acumulados.
- **Flujos alternos:** 2a. Un destino es inalcanzable: se guarda como parada `INALCANZABLE` al final y se resalta en la hoja de ruta. 2b. El vehículo no tiene envíos: mensaje y fin.

#### CU-12 Cerrar tramo o cambiar su costo (ruteo dinámico)

- **Actor:** Despachador. **Precondición:** existen rutas planificadas.
- **Flujo principal:**
  1. El despachador selecciona un tramo y elige cerrar, reabrir o capturar un nuevo costo.
  2. El sistema persiste el cambio del tramo (transacción 1) y, tras el COMMIT, actualiza el grafo.
  3. Determina qué rutas resultan afectadas según los árboles de caminos mínimos que usaron.
  4. Recalcula solo esas rutas, las guarda como nuevas, marca las anteriores como `RECALCULADA` (transacción 2) y actualiza el registro en memoria.
  5. Muestra cada ruta anterior junto a la nueva, con la diferencia de costo.
- **Flujos alternos:** 1a. El costo es ≤ 0 o no es numérico: `DatosInvalidosException`, sin cambios. 3a. Ninguna ruta afectada: se informa «0 rutas recalculadas». 4a. Una parada queda inalcanzable: se marca `INALCANZABLE`.

### Escenarios de prueba derivados

**Tabla 17**  
*Escenarios de aceptación (se ejecutan en la consola y se automatizan en `Electro.Tests`)*

| ID | Escenario | Datos de entrada | Resultado esperado | CU |
|---|---|---|---|---|
| ESC-01 | Venta exitosa | Cliente 1, almacén 1, 2 SKU con stock | Folio y guía nuevos, stock descontado, pedido en la cola | CU-01 |
| ESC-02 | Venta con stock insuficiente | Cantidad mayor a la existencia | Mensaje de faltante; base y memoria sin cambios | CU-01 |
| ESC-03 | Deshacer venta tras reiniciar | Venta, cerrar, reabrir, deshacer | Stock restaurado, pedido `CANCELADO`, fuera de la cola, bitácora `REVERTIDA` | CU-03 |
| ESC-04 | Deshacer con pila vacía | Sin operaciones aplicadas | Mensaje «no hay operaciones»; sin excepción no controlada | CU-03 |
| ESC-05 | Prioridad de despacho | Pedidos con urgencias 5/express, 5/estándar, 3/express | Orden: 5-express, 5-estándar, 3-express | CU-08 |
| ESC-06 | FFD con capacidad limitada | 6 paquetes, 2 vehículos pequeños | Ninguno excede volumen ni peso; los sobrantes quedan en la cola | CU-08 |
| ESC-07 | Búsqueda de guía inexistente | `EL000000000000` | «Guía no encontrada» | CU-10 |
| ESC-08 | Ruta con destino inalcanzable | Vehículo con envío al domicilio aislado del DML | Parada marcada `INALCANZABLE` | CU-11 |
| ESC-09 | Cierre de tramo usado por una ruta | Cerrar un tramo de la ruta planificada | Solo esa ruta se recalcula; costo nuevo ≥ anterior | CU-12 |
| ESC-10 | Baja de costo en un tramo ajeno | Bajar el costo de un tramo que no mejora ninguna ruta | 0 rutas recalculadas | CU-12 |
| ESC-11 | Buffer circular lleno | Capacidad 4, 5 solicitudes | La quinta se rechaza; `Rechazados = 1` | CU-04 |
| ESC-12 | Respaldo e integridad | Ejecutar respaldo | Archivo creado; `integrity_check = ok`; 0 violaciones de FK | CU-14 |
