# Entrega 0 – Contrato técnico del proyecto ElectroLogística

Proyecto integrador AED-1026 Estructura de Datos + Ingeniería de Software  
Instituto Tecnológico de Monclova · Ingeniería en Informática · 3.er semestre · Docente: Rubén Riojas

| Perfil | Integrante | Matrícula | Enfoque |
|---|---|---|---|
| A | Stephanny Elizabeth Torres Romo | I2505386 | Arquitectura, grafos y rendimiento |
| B | Bárbara Sofía Pérez Romo | [MATRÍCULA] | Despacho, estructuras lineales, ABB y QA |
| C | José Luis García Robinson | I22050340 | Persistencia SQLite y gestión |

Este documento fija los nombres que no cambiarán en las entregas siguientes: carpetas, archivos, namespaces, tablas, columnas, restricciones, índices y firmas públicas. Si más adelante hay que cambiar un nombre, lo aviso y listo los archivos afectados.

---

## 0. Observaciones sobre la guía y el prompt

1. **Urgencia y QuickSort de 3 vías.** La guía (Fase 6) dice que la urgencia «tiene pocos valores repetidos». El prompt dice «pocos valores distintos y muchos repetidos». Uso la del prompt porque es la que justifica técnicamente la partición en 3 vías; la frase de la guía se entiende como «pocos valores, que se repiten».
2. **`IEnumerable<T>`.** Vive en `System.Collections.Generic`, pero el prompt exige exponerlo con `yield`. Es la única referencia permitida a ese namespace en Core: solo `IEnumerable<T>`/`IEnumerator<T>`. Una prueba de arquitectura (Fase 7) verifica que no aparezca ninguna colección prohibida ni LINQ.
3. **Grafo cargado desde SQLite.** El prompt pide reconstruir «tramos habilitados». Se cargan **todos** los tramos y los cerrados quedan con `Habilitada = false`. Dijkstra los ignora, así que el grafo efectivo es el de tramos habilitados, y además se puede reabrir un tramo desde la app sin volver a leer la base.
4. **Matrícula de A.** `I2505386` tiene un dígito menos que `I22050340`. Revísala; la dejo tal como me la diste.

---

## 1. Árbol completo de carpetas y archivos

Leyenda: **[A] [B] [C]** = dueño del archivo; sus commits los hace ese integrante. `(F n)` = fase/entrega donde aparece. Los archivos marcados ♻ se vuelven a entregar completos en una fase posterior al crecer el sistema; siempre conservan su nombre y sus firmas públicas previas.

```text
ElectroLogistica/
├─ ElectroLogistica.sln                                  [A] (F2)
├─ Directory.Build.props                                 [A] (F2)  net10.0, Nullable, TreatWarningsAsErrors
├─ .gitignore                                            [A] (F2)  NO excluye datos/electro.db
├─ .gitattributes                                        [A] (F2)  *.db binary, *.sql eol=lf
├─ README.md                                             [B] (E8)
├─ sql/
│  ├─ DDL.sql                                            [C] (F2)
│  ├─ DML.sql                                            [C] (F2)
│  └─ ESQUEMA_FINAL.sql                                  [C] (F7)
├─ datos/
│  └─ electro.db                                         [C] (F2) generado con `Electro.Consola --crear-bd`
├─ resultados/
│  └─ .gitkeep                                           [A] (F6)  aquí caen CSV y PNG medidos
├─ scripts/
│  ├─ graficas.py                                        [A] (F6)
│  └─ requirements.txt                                   [A] (F6)
├─ docs/
│  ├─ 00-Contrato.md                                     [A] (E0)
│  ├─ diagramas/
│  │  ├─ arquitectura.md                                 [A] (F2)
│  │  ├─ clases.md                                       [A] (F2) ♻F7
│  │  ├─ casos_uso.md                                    [B] (F1)
│  │  ├─ entidad_relacion.md                             [C] (F2)
│  │  └─ gantt.md                                        [C] (F1)
│  ├─ esquema/Esquema_SQLite.md                          [C] (F7)
│  ├─ git/Flujo_Git.md                                   [A] (E8)
│  ├─ memoria/Procedimiento_Diagnostico_VS.md            [A] (F7)
│  ├─ video/Guion_Video.md                               [C] (E10)
│  └─ reporte/
│     ├─ metadata.yaml                                   [C] (E9)
│     ├─ 00_portada.md                                   [C] (E9)
│     ├─ 01_resumen_ejecutivo.md                         [C] (E9)
│     ├─ 10_viabilidad_costo_beneficio.md                [A] (F1)
│     ├─ 11_requerimientos_casos_uso.md                  [B] (F1)
│     ├─ 12_integridad_riesgos_gantt.md                  [C] (F1)
│     ├─ 13_arquitectura_uml.md                          [A] (F2) ♻F7
│     ├─ 14_modelo_relacional.md                         [C] (F2)
│     ├─ 15_plan_pruebas_resultados.md                   [B] (F7)
│     ├─ 20_estructuras_lineales_buffer.md               [B] (F2)
│     ├─ 21_despacho_deshacer_ffd.md                     [B] (F3)  incluye subsección FFD de [A]
│     ├─ 22_abb_catalogo.md                              [B] (F4)  incluye subsecciones de [A] y [C]
│     ├─ 23_grafo_dijkstra_ruteo.md                      [A] (F5)
│     ├─ 24_ordenamiento_rendimiento.md                  [A] (F6)
│     ├─ 25_memoria_recursos.md                          [A] (F7)
│     ├─ 30_conclusiones.md                              [C] (E9)
│     ├─ 40_referencias.md                               [B] (E9)
│     ├─ 50_cumplimiento_rubricas.md                     [A] (E9)
│     └─ capturas/.gitkeep                               [A] (F7)
├─ src/
│  ├─ Electro.Core/
│  │  ├─ Electro.Core.csproj                             [A] (F2)  embebe sql/DDL.sql y sql/DML.sql
│  │  ├─ Comun/
│  │  │  ├─ Excepciones.cs                               [A] (F2)
│  │  │  ├─ Validar.cs                                   [A] (F2)
│  │  │  └─ Reloj.cs                                     [A] (F2)
│  │  ├─ Contratos/
│  │  │  ├─ IComparador.cs                               [A] (F2)
│  │  │  ├─ IPosicionMonticulo.cs                        [A] (F2)
│  │  │  ├─ IColaPrioridad.cs                            [A] (F2)
│  │  │  ├─ IPila.cs                                     [A] (F2)
│  │  │  ├─ IBufferPedidos.cs                            [A] (F2)
│  │  │  ├─ ParClaveValor.cs                             [A] (F2)
│  │  │  ├─ IArbolBusqueda.cs                            [A] (F2)
│  │  │  └─ IOrdenador.cs                                [A] (F2)
│  │  ├─ Dominio/
│  │  │  ├─ Logistica/
│  │  │  │  ├─ TipoNodo.cs                               [A] (F2)
│  │  │  │  ├─ NodoDistribucion.cs                       [A] (F2)
│  │  │  │  ├─ AristaVial.cs                             [A] (F2)
│  │  │  │  ├─ Almacen.cs                                [A] (F2)
│  │  │  │  ├─ Vehiculo.cs                               [A] (F2)
│  │  │  │  └─ FlotaVehicular.cs                         [A] (F2)
│  │  │  └─ Ventas/
│  │  │     ├─ Enumeraciones.cs                          [B] (F2)
│  │  │     ├─ Electrodomestico.cs                       [B] (F2)
│  │  │     ├─ Cliente.cs                                [B] (F2)
│  │  │     ├─ LineaPedido.cs                            [B] (F2)
│  │  │     ├─ Pedido.cs                                 [B] (F2)
│  │  │     ├─ ResumenPedido.cs                          [B] (F2)
│  │  │     ├─ Paquete.cs                                [B] (F2)
│  │  │     └─ SolicitudVenta.cs                         [B] (F2)
│  │  ├─ Estructuras/
│  │  │  ├─ Lineales/
│  │  │  │  ├─ NodoSimple.cs                             [B] (F2)
│  │  │  │  ├─ ArregloDinamico.cs                        [B] (F2)
│  │  │  │  ├─ ColaCircular.cs                           [B] (F2)
│  │  │  │  ├─ ListaEnlazadaCola.cs                      [B] (F2)
│  │  │  │  ├─ MedidorMemoria.cs                         [B] (F2)
│  │  │  │  └─ PilaArreglo.cs                            [B] (F3)
│  │  │  ├─ Monticulo/
│  │  │  │  └─ MonticuloBinario.cs                       [B] (F3)
│  │  │  └─ Arboles/
│  │  │     ├─ NodoArbol.cs                              [B] (F4)
│  │  │     ├─ PasoConstruccionAbb.cs                    [B] (F4)
│  │  │     ├─ ComparadoresClave.cs                      [B] (F4)
│  │  │     └─ ArbolBinarioBusqueda.cs                   [B] (F4)
│  │  ├─ Despacho/
│  │  │  ├─ CalculadoraPrioridad.cs                      [B] (F3)
│  │  │  ├─ ElementoDespacho.cs                          [B] (F3)
│  │  │  ├─ ComparadorPrioridadDespacho.cs               [B] (F3)
│  │  │  ├─ ColaDespacho.cs                              [B] (F3)
│  │  │  ├─ TipoOperacion.cs                             [B] (F2, adelantado: lo usa ConversorValores)
│  │  │  ├─ OperacionRegistrada.cs                       [B] (F3)
│  │  │  ├─ CargasOperacion.cs                           [B] (F3)
│  │  │  ├─ SerializadorCargas.cs                        [B] (F3)
│  │  │  └─ PilaDeshacer.cs                              [B] (F3)
│  │  ├─ Asignacion/
│  │  │  ├─ ComparadorPaqueteVolumenDesc.cs              [A] (F3)
│  │  │  ├─ ResultadoAsignacion.cs                       [A] (F3)
│  │  │  └─ FirstFitDecreasing.cs                        [A] (F3)
│  │  ├─ Caminos/
│  │  │  ├─ ResultadoCamino.cs                           [B] (F5)
│  │  │  ├─ ReconstructorCamino.cs                       [B] (F5)
│  │  │  ├─ HojaDeRuta.cs                                [B] (F5)
│  │  │  └─ GeneradorHojaDeRuta.cs                       [B] (F5)
│  │  ├─ Grafos/
│  │  │  ├─ GrafoLogistico.cs                            [A] (F5)
│  │  │  ├─ EntradaDijkstra.cs                           [A] (F5)
│  │  │  ├─ PasoTrazaDijkstra.cs                         [A] (F5)
│  │  │  ├─ ResultadoDijkstra.cs                         [A] (F5)
│  │  │  ├─ Dijkstra.cs                                  [A] (F5)
│  │  │  ├─ ExportadorTrazaDijkstra.cs                   [A] (F5)
│  │  │  ├─ RutaPlanificada.cs                           [A] (F5)
│  │  │  ├─ VecinoMasCercano.cs                          [A] (F5)
│  │  │  ├─ ResultadoRecalculo.cs                        [A] (F5)
│  │  │  └─ RuteadorDinamico.cs                          [A] (F5)
│  │  ├─ Ordenamiento/
│  │  │  ├─ CriterioOrden.cs                             [B] (F6)
│  │  │  ├─ ComparadoresPedido.cs                        [B] (F6)
│  │  │  ├─ ComparadorInverso.cs                         [B] (F6)
│  │  │  ├─ FabricaComparadores.cs                       [B] (F6)
│  │  │  ├─ QuickSort3Vias.cs                            [B] (F6)
│  │  │  └─ MergeSort.cs                                 [C] (F6)
│  │  ├─ Rendimiento/
│  │  │  ├─ InsertionSort.cs                             [A] (F6)
│  │  │  ├─ Estadistica.cs                               [A] (F4)
│  │  │  ├─ ConfiguracionBenchmark.cs                    [A] (F4)
│  │  │  ├─ MedicionBenchmark.cs                         [A] (F4)
│  │  │  ├─ GeneradorDatos.cs                            [A] (F4) ♻F6
│  │  │  ├─ CurvaTeorica.cs                              [A] (F4)
│  │  │  ├─ ExportadorCsv.cs                             [A] (F4) ♻F6
│  │  │  ├─ BenchmarkBusquedaAbb.cs                      [A] (F4)
│  │  │  └─ BancoBenchmarks.cs                           [A] (F6)
│  │  ├─ Datos/
│  │  │  ├─ IFabricaConexiones.cs                        [C] (F2)
│  │  │  ├─ FabricaConexiones.cs                         [C] (F2)
│  │  │  ├─ EjecutorTransaccional.cs                     [C] (F2)
│  │  │  ├─ InicializadorBaseDatos.cs                    [C] (F2)
│  │  │  ├─ ConversorValores.cs                          [C] (F2)
│  │  │  ├─ GeneradorFolios.cs                           [C] (F3)
│  │  │  ├─ FiltroCatalogo.cs                            [C] (F4)
│  │  │  ├─ Dao/
│  │  │  │  ├─ ContratosDao.cs                           [C] (F3) ♻F4 ♻F5
│  │  │  │  ├─ RegistrosDatos.cs                         [C] (F3) ♻F5  DTO de lectura
│  │  │  │  ├─ ResultadosPersistencia.cs                 [C] (F3)
│  │  │  │  ├─ ClienteDao.cs                             [C] (F3)
│  │  │  │  ├─ InventarioDao.cs                          [C] (F3)
│  │  │  │  ├─ PedidoDao.cs                              [C] (F3)
│  │  │  │  ├─ EnvioDao.cs                               [C] (F3)
│  │  │  │  ├─ FlotaDao.cs                               [C] (F3)
│  │  │  │  ├─ BitacoraDao.cs                            [C] (F3)
│  │  │  │  ├─ RegistroVentasDao.cs                      [C] (F3)
│  │  │  │  ├─ CatalogoDao.cs                            [C] (F4)
│  │  │  │  ├─ RedDao.cs                                 [C] (F5)
│  │  │  │  └─ RutaDao.cs                                [C] (F5)
│  │  │  ├─ Sincronizacion/
│  │  │  │  ├─ EstadoMemoria.cs                          [C] (F3) ♻F4 ♻F5
│  │  │  │  └─ CargadorMemoria.cs                        [C] (F3) ♻F4 ♻F5
│  │  │  ├─ Respaldo/
│  │  │  │  └─ ServicioRespaldo.cs                       [C] (F7)
│  │  │  └─ Rendimiento/
│  │  │     ├─ BenchmarkCargaMasiva.cs                   [C] (F6)
│  │  │     └─ BenchmarkIndices.cs                       [C] (F6)
│  │  └─ Servicios/
│  │     ├─ SistemaElectro.cs                            [A] (F2 mínimo, lo usa Electro.App) ♻F3 ♻F4 ♻F5 ♻F7
│  │     ├─ ServicioAsignacion.cs                        [A] (F3)
│  │     ├─ ServicioRutas.cs                             [A] (F5)
│  │     ├─ ServicioDespacho.cs                          [B] (F3)
│  │     ├─ ServicioDeshacer.cs                          [B] (F3)
│  │     ├─ ServicioRecepcion.cs                         [B] (F3)
│  │     ├─ ServicioVentas.cs                            [C] (F3)
│  │     └─ ServicioCatalogo.cs                          [C] (F4)
│  ├─ Electro.App/                                       (net10.0-windows, WinForms; solo compila en Windows)
│  │  ├─ Electro.App.csproj                              [A] (F2)
│  │  ├─ Program.cs                                      [A] (F2)
│  │  ├─ IPestanaModulo.cs                               [A] (F2)
│  │  ├─ FormPrincipal.cs                                [A] (F2)  descubre pestañas por reflexión
│  │  └─ Pestanas/
│  │     ├─ PestanaEstado.cs                             [A] (F2)
│  │     ├─ PestanaVentas.cs                             [C] (F3)
│  │     ├─ PestanaDespacho.cs                           [B] (F3)  despacho + deshacer + FFD
│  │     ├─ PestanaInventario.cs                         [B] (F4)
│  │     └─ PestanaRutas.cs                              [A] (F5)  cerrar/abrir tramo, cambiar costo, ver ruta nueva
│  └─ Electro.Consola/
│     ├─ Electro.Consola.csproj                          [A] (F2)
│     ├─ Program.cs                                      [A] (F2)  modo menú o modo CLI por argumento
│     ├─ IEscenarioConsola.cs                            [A] (F2)
│     ├─ ContextoConsola.cs                              [A] (F2)
│     ├─ MenuPrincipal.cs                                [A] (F2)  descubre escenarios por reflexión
│     └─ Escenarios/
│        ├─ EscenarioCrearBaseDatos.cs                   [C] (F2)  --crear-bd
│        ├─ EscenarioBuffers.cs                          [B] (F2)  --buffers
│        ├─ EscenarioVentasDeshacer.cs                   [B] (F3)  --ventas
│        ├─ EscenarioAsignacionFfd.cs                    [A] (F3)  --ffd
│        ├─ EscenarioCatalogoAbb.cs                      [B] (F4)  --catalogo
│        ├─ EscenarioBenchmarkAbb.cs                     [A] (F4)  --bench-abb
│        ├─ EscenarioTrazaDijkstra.cs                    [A] (F5)  --traza-dijkstra
│        ├─ EscenarioRuteoDinamico.cs                    [A] (F5)  --ruteo-dinamico
│        ├─ EscenarioDatosAleatorios.cs                  [B] (F6)  --aleatorios <n> <semilla>
│        ├─ EscenarioBenchmarkOrdenamiento.cs            [A] (F6)  --bench-orden
│        ├─ EscenarioBenchmarkSqlite.cs                  [C] (F6)  --bench-sqlite
│        ├─ EscenarioTransaccionInterrumpida.cs          [C] (F7)  --transaccion-interrumpida <ruta.db>
│        └─ EscenarioRespaldo.cs                         [C] (F7)  --respaldo <destino.db>
└─ tests/
   └─ Electro.Tests/
      ├─ Electro.Tests.csproj                            [B] (F2)  referencia Core y Consola
      ├─ Infraestructura/
      │  ├─ BaseDatosTemporal.cs                         [C] (F2)
      │  └─ RutasRepositorio.cs                          [B] (F2)
      ├─ Fase2/
      │  ├─ ArregloDinamicoTests.cs                      [B]
      │  ├─ ColaCircularTests.cs                         [B]
      │  ├─ ListaEnlazadaColaTests.cs                    [B]
      │  ├─ DominioLogisticaTests.cs                     [A]
      │  ├─ DominioVentasTests.cs                        [B]
      │  └─ EsquemaSqliteTests.cs                        [C]
      ├─ Fase3/
      │  ├─ PilaArregloTests.cs                          [B]
      │  ├─ MonticuloBinarioTests.cs                     [B]
      │  ├─ ColaDespachoTests.cs                         [B]
      │  ├─ PilaDeshacerTests.cs                         [B]
      │  ├─ FirstFitDecreasingTests.cs                   [A]
      │  ├─ RegistroVentasDaoTests.cs                    [C]
      │  └─ DeshacerPersistenteTests.cs                  [B]
      ├─ Fase4/
      │  ├─ ArbolBinarioBusquedaTests.cs                 [B]
      │  ├─ ConstruccionBalanceadaTests.cs               [B]
      │  ├─ CatalogoDaoTests.cs                          [C]
      │  └─ BenchmarkBusquedaAbbTests.cs                 [A]
      ├─ Fase5/
      │  ├─ GrafoLogisticoTests.cs                       [A]
      │  ├─ DijkstraTests.cs                             [A]
      │  ├─ ReconstructorCaminoTests.cs                  [B]
      │  ├─ VecinoMasCercanoTests.cs                     [A]
      │  ├─ RuteadorDinamicoTests.cs                     [A]
      │  └─ RutaDaoTests.cs                              [C]
      ├─ Fase6/
      │  ├─ ComparadoresTests.cs                         [B]
      │  ├─ QuickSort3ViasTests.cs                       [B]
      │  ├─ MergeSortTests.cs                            [C]
      │  ├─ InsertionSortTests.cs                        [A]
      │  ├─ EstadisticaCsvTests.cs                       [A]
      │  └─ BenchmarkSqliteTests.cs                      [C]
      └─ Fase7/
         ├─ CasosLimiteTests.cs                          [B]
         ├─ IntegracionFlujoCompletoTests.cs             [B]
         ├─ ArquitecturaTests.cs                         [A]
         ├─ MemoriaTests.cs                              [A]
         ├─ CargaMemoriaTests.cs                         [C]
         ├─ IntegridadReferencialTests.cs                [C]
         ├─ RestriccionesCheckTests.cs                   [C]
         ├─ RecuperacionTransaccionTests.cs              [C]
         └─ RespaldoTests.cs                             [C]
```

---

## 2. Namespaces por perfil

| Namespace | Dueño | Contenido |
|---|---|---|
| `Electro.Core.Comun` | A | Excepciones, validación, reloj inyectable |
| `Electro.Core.Contratos` | A | Interfaces abstractas compartidas (Fase 2) |
| `Electro.Core.Dominio.Logistica` | A | TDA nodo de distribución, arista vial, almacén, vehículo, flota |
| `Electro.Core.Dominio.Ventas` | B | TDA electrodoméstico, paquete, cliente, pedido |
| `Electro.Core.Estructuras.Lineales` | B | Arreglo dinámico, buffers, pila |
| `Electro.Core.Estructuras.Monticulo` | B | Montículo binario |
| `Electro.Core.Estructuras.Arboles` | B | ABB |
| `Electro.Core.Despacho` | B | Prioridad, cola de despacho, pila de deshacer, cargas útiles |
| `Electro.Core.Caminos` | B | Reconstrucción recursiva del camino, hoja de ruta |
| `Electro.Core.Ordenamiento` | B (C: `MergeSort`) | Comparadores, QuickSort, MergeSort |
| `Electro.Core.Asignacion` | A | First-Fit Decreasing |
| `Electro.Core.Grafos` | A | Grafo, Dijkstra, vecino más cercano, ruteo dinámico |
| `Electro.Core.Rendimiento` | A | Banco de benchmarks, estadística, CSV, InsertionSort |
| `Electro.Core.Datos` (+ `.Dao`, `.Sincronizacion`, `.Respaldo`, `.Rendimiento`) | C | SQLite, DAO, carga a memoria, respaldo, benchmarks SQLite |
| `Electro.Core.Servicios` | A/B/C por archivo | Fachada y casos de uso; único punto de entrada para App y Consola |
| `Electro.App`, `Electro.App.Pestanas` | A/B/C por archivo | UI WinForms |
| `Electro.Consola`, `Electro.Consola.Escenarios` | A/B/C por archivo | Menú, escenarios y benchmarks |
| `Electro.Tests.*` | dueño del módulo probado | Pruebas xUnit |

**Dependencias permitidas** (las verifica `ArquitecturaTests`):  
`App → Core.Servicios` (+ tipos de dominio, resultados y `FiltroCatalogo`). App **no** referencia `Microsoft.Data.Sqlite` (Core lo declara con `PrivateAssets="compile"`), no crea DAO, fábricas de conexión ni transacciones, y no contiene SQL.  
`Consola → Core` (cualquier namespace público).  
`Core.Datos → Dominio, Estructuras, Despacho, Grafos` (para cargar memoria).  
Estructuras y algoritmos **no** dependen de `Datos`.

---

## 3. Esquema SQLite (nombres definitivos)

Convenciones: tablas y columnas en `snake_case`; PK `id_<entidad>` `INTEGER PRIMARY KEY`; dinero en **centavos** `INTEGER` (exacto, sin redondeo de `REAL`); fechas `TEXT` ISO-8601 `yyyy-MM-ddTHH:mm:ss`; booleanos `INTEGER` 0/1; restricciones con nombre `ck_<tabla>_<regla>`, FK `fk_<tabla>_<referida>`. Cada conexión ejecuta `PRAGMA foreign_keys = ON;`. Las FK usan `ON DELETE RESTRICT` (nunca se borra historia; se cambian estados).

### 3.1 Tablas

**T1 `nodos_red`**: nodos del grafo logístico.

| Columna | Tipo | Restricción |
|---|---|---|
| id_nodo | INTEGER | PK |
| codigo | TEXT | NOT NULL, UNIQUE |
| nombre | TEXT | NOT NULL |
| tipo | TEXT | NOT NULL, `ck_nodos_red_tipo` IN ('ALMACEN_CENTRAL','CENTRO_ACOPIO','DOMICILIO') |
| codigo_postal | TEXT | NOT NULL, `ck_nodos_red_cp` length = 5 |

**T2 `almacenes`**: almacenes centrales y sucursales.

| Columna | Tipo | Restricción |
|---|---|---|
| id_almacen | INTEGER | PK |
| codigo | TEXT | NOT NULL, UNIQUE |
| nombre | TEXT | NOT NULL |
| tipo | TEXT | NOT NULL, `ck_almacenes_tipo` IN ('CENTRAL','SUCURSAL') |
| id_nodo | INTEGER | NOT NULL, UNIQUE, `fk_almacenes_nodos_red` |

**T3 `electrodomesticos`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_electrodomestico | INTEGER | PK |
| sku | TEXT | NOT NULL, `ck_electrodomesticos_sku` length BETWEEN 4 AND 20 |
| nombre | TEXT | NOT NULL |
| marca | TEXT | NOT NULL |
| categoria | TEXT | NOT NULL |
| peso_kg | REAL | NOT NULL, `ck_electrodomesticos_peso` > 0 |
| volumen_m3 | REAL | NOT NULL, `ck_electrodomesticos_volumen` > 0 |
| activo | INTEGER | NOT NULL DEFAULT 1, `ck_electrodomesticos_activo` IN (0,1) |
| fecha_alta | TEXT | NOT NULL |

**T4 `historial_precios`**: el precio vigente es la fila con `vigente_hasta IS NULL` (sin columna redundante de precio en T3, para cumplir 3FN).

| Columna | Tipo | Restricción |
|---|---|---|
| id_historial_precio | INTEGER | PK |
| id_electrodomestico | INTEGER | NOT NULL, `fk_historial_precios_electrodomesticos` |
| precio_centavos | INTEGER | NOT NULL, `ck_historial_precios_precio` > 0 |
| vigente_desde | TEXT | NOT NULL |
| vigente_hasta | TEXT | NULL, `ck_historial_precios_rango` (vigente_hasta IS NULL OR vigente_hasta >= vigente_desde) |

**T5 `inventario`**: existencias por almacén.

| Columna | Tipo | Restricción |
|---|---|---|
| id_almacen | INTEGER | PK(1), `fk_inventario_almacenes` |
| id_electrodomestico | INTEGER | PK(2), `fk_inventario_electrodomesticos` |
| stock | INTEGER | NOT NULL, `ck_inventario_stock` >= 0 |
| stock_minimo | INTEGER | NOT NULL DEFAULT 0, `ck_inventario_stock_minimo` >= 0 |

**T6 `clientes`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_cliente | INTEGER | PK |
| nombre | TEXT | NOT NULL, `ck_clientes_nombre` length(trim(nombre)) > 0 |
| telefono | TEXT | NOT NULL, `ck_clientes_telefono` length = 10 |
| correo | TEXT | NOT NULL, UNIQUE, `ck_clientes_correo` LIKE '%_@_%._%' |
| id_nodo_domicilio | INTEGER | NOT NULL, `fk_clientes_nodos_red` |
| fecha_alta | TEXT | NOT NULL |

**T7 `pedidos`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_pedido | INTEGER | PK |
| folio | TEXT | NOT NULL, UNIQUE (`PED-000001`) |
| id_cliente | INTEGER | NOT NULL, `fk_pedidos_clientes` |
| id_almacen_origen | INTEGER | NOT NULL, `fk_pedidos_almacenes` |
| fecha_creacion | TEXT | NOT NULL |
| estado | TEXT | NOT NULL, `ck_pedidos_estado` IN ('PENDIENTE','ASIGNADO','EN_RUTA','ENTREGADO','CANCELADO','DEVUELTO') |
| urgencia | INTEGER | NOT NULL, `ck_pedidos_urgencia` BETWEEN 1 AND 5 (5 = máxima) |
| tipo_entrega | TEXT | NOT NULL, `ck_pedidos_tipo_entrega` IN ('ESTANDAR','EXPRESS','PROGRAMADA') |

**T8 `detalle_pedido`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_pedido | INTEGER | PK(1), `fk_detalle_pedido_pedidos` |
| id_electrodomestico | INTEGER | PK(2), `fk_detalle_pedido_electrodomesticos` |
| cantidad | INTEGER | NOT NULL, `ck_detalle_pedido_cantidad` > 0 |
| precio_unitario_centavos | INTEGER | NOT NULL, `ck_detalle_pedido_precio` > 0 (precio al momento de la venta) |

**T9 `historial_estados_pedido`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_historial_estado | INTEGER | PK |
| id_pedido | INTEGER | NOT NULL, `fk_historial_estados_pedidos` |
| estado_anterior | TEXT | NULL, `ck_historial_estados_anterior` (NULL o estado válido) |
| estado_nuevo | TEXT | NOT NULL, `ck_historial_estados_nuevo` (estado válido) |
| fecha | TEXT | NOT NULL |
| motivo | TEXT | NOT NULL |
| | | `ck_historial_estados_cambio` (estado_anterior IS NULL OR estado_anterior <> estado_nuevo) |

**T10 `envios`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_envio | INTEGER | PK |
| id_pedido | INTEGER | NOT NULL, `fk_envios_pedidos` (1:1 por índice único) |
| numero_guia | TEXT | NOT NULL (`EL` + `yyMMdd` + 6 dígitos del id) |
| id_nodo_destino | INTEGER | NOT NULL, `fk_envios_nodos_red` (puede diferir del domicilio del cliente) |
| estado | TEXT | NOT NULL, `ck_envios_estado` IN ('PREPARACION','EN_TRANSITO','ENTREGADO','DEVUELTO','CANCELADO') |
| fecha_creacion | TEXT | NOT NULL |

**T11 `vehiculos`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_vehiculo | INTEGER | PK |
| placa | TEXT | NOT NULL, UNIQUE |
| descripcion | TEXT | NOT NULL |
| capacidad_volumen_m3 | REAL | NOT NULL, `ck_vehiculos_volumen` > 0 |
| capacidad_peso_kg | REAL | NOT NULL, `ck_vehiculos_peso` > 0 |
| id_almacen_base | INTEGER | NOT NULL, `fk_vehiculos_almacenes` |
| activo | INTEGER | NOT NULL DEFAULT 1, `ck_vehiculos_activo` IN (0,1) |

**T12 `asignaciones_envio`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_asignacion | INTEGER | PK |
| id_envio | INTEGER | NOT NULL, `fk_asignaciones_envios` |
| id_vehiculo | INTEGER | NOT NULL, `fk_asignaciones_vehiculos` |
| fecha_asignacion | TEXT | NOT NULL |
| vigente | INTEGER | NOT NULL DEFAULT 1, `ck_asignaciones_vigente` IN (0,1) |

**T13 `tramos`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_tramo | INTEGER | PK |
| id_nodo_origen | INTEGER | NOT NULL, `fk_tramos_origen` |
| id_nodo_destino | INTEGER | NOT NULL, `fk_tramos_destino` |
| distancia_km | REAL | NOT NULL, `ck_tramos_distancia` > 0 |
| costo | REAL | NOT NULL, `ck_tramos_costo` > 0 (minutos estimados) |
| habilitado | INTEGER | NOT NULL DEFAULT 1, `ck_tramos_habilitado` IN (0,1) |
| fecha_actualizacion | TEXT | NOT NULL |
| | | `ck_tramos_sin_lazo` (id_nodo_origen <> id_nodo_destino) |

**T14 `rutas`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_ruta | INTEGER | PK |
| id_vehiculo | INTEGER | NOT NULL, `fk_rutas_vehiculos` |
| id_nodo_origen | INTEGER | NOT NULL, `fk_rutas_nodos_red` |
| fecha_planificacion | TEXT | NOT NULL |
| estado | TEXT | NOT NULL, `ck_rutas_estado` IN ('PLANIFICADA','EN_CURSO','COMPLETADA','RECALCULADA') |
| costo_total | REAL | NOT NULL, `ck_rutas_costo` >= 0 |
| distancia_total_km | REAL | NOT NULL, `ck_rutas_distancia` >= 0 |
| id_ruta_anterior | INTEGER | NULL, `fk_rutas_ruta_anterior` → rutas (trazabilidad del recálculo) |

**T15 `ruta_paradas`**

| Columna | Tipo | Restricción |
|---|---|---|
| id_parada | INTEGER | PK |
| id_ruta | INTEGER | NOT NULL, `fk_ruta_paradas_rutas` |
| orden | INTEGER | NOT NULL, `ck_ruta_paradas_orden` >= 1 |
| id_nodo | INTEGER | NOT NULL, `fk_ruta_paradas_nodos_red` |
| id_envio | INTEGER | NULL, `fk_ruta_paradas_envios` |
| distancia_km_estimada | REAL | NOT NULL, `ck_ruta_paradas_distancia` >= 0 |
| costo_estimado | REAL | NOT NULL, `ck_ruta_paradas_costo` >= 0 (desde la parada anterior) |
| costo_acumulado | REAL | NOT NULL, `ck_ruta_paradas_acumulado` >= 0 |
| estado_entrega | TEXT | NOT NULL, `ck_ruta_paradas_estado` IN ('PENDIENTE','ENTREGADA','FALLIDA','INALCANZABLE') |

**T16 `bitacora_operaciones`**: respaldo persistente de la pila de deshacer.

| Columna | Tipo | Restricción |
|---|---|---|
| id_operacion | INTEGER | PK (orden LIFO) |
| tipo | TEXT | NOT NULL, `ck_bitacora_tipo` IN ('VENTA','CANCELACION','DEVOLUCION','REASIGNACION') |
| carga_util | TEXT | NOT NULL, `ck_bitacora_json` json_valid(carga_util) |
| fecha | TEXT | NOT NULL |
| estado | TEXT | NOT NULL DEFAULT 'APLICADA', `ck_bitacora_estado` IN ('APLICADA','REVERTIDA') |
| fecha_reversion | TEXT | NULL |
| | | `ck_bitacora_reversion` ((estado='APLICADA' AND fecha_reversion IS NULL) OR (estado='REVERTIDA' AND fecha_reversion IS NOT NULL)) |

### 3.2 Índices

| Índice | Tabla(columnas) | Propósito |
|---|---|---|
| `ux_electrodomesticos_sku` | UNIQUE electrodomesticos(sku) | SKU (carga ordenada al ABB) |
| `ix_electrodomesticos_categoria` | electrodomesticos(categoria) | Filtro por categoría |
| `ix_electrodomesticos_marca` | electrodomesticos(marca) | Filtro por marca |
| `ux_historial_precios_vigente` | UNIQUE historial_precios(id_electrodomestico) WHERE vigente_hasta IS NULL | Un único precio vigente |
| `ix_historial_precios_producto_fecha` | historial_precios(id_electrodomestico, vigente_desde) | Historial |
| `ix_inventario_electrodomestico` | inventario(id_electrodomestico) | Existencias por producto |
| `ix_clientes_nodo` | clientes(id_nodo_domicilio) | FK |
| `ix_pedidos_cliente` | pedidos(id_cliente) | Pedidos por cliente |
| `ix_pedidos_estado` | pedidos(estado) | Carga de la cola de despacho |
| `ix_detalle_pedido_electrodomestico` | detalle_pedido(id_electrodomestico) | FK |
| `ix_historial_estados_pedido` | historial_estados_pedido(id_pedido, fecha) | Historial |
| `ux_envios_numero_guia` | UNIQUE envios(numero_guia) | Folio de envío (carga al ABB de guías) |
| `ux_envios_pedido` | UNIQUE envios(id_pedido) | Relación 1:1 |
| `ix_envios_estado` | envios(estado) | Paquetes por asignar |
| `ux_asignaciones_envio_vigente` | UNIQUE asignaciones_envio(id_envio) WHERE vigente = 1 | Un vehículo vigente por envío |
| `ix_asignaciones_vehiculo` | asignaciones_envio(id_vehiculo, vigente) | Carga de un vehículo |
| `ux_tramos_origen_destino` | UNIQUE tramos(id_nodo_origen, id_nodo_destino) | Sin aristas paralelas |
| `ix_tramos_destino` | tramos(id_nodo_destino) | FK |
| `ix_rutas_vehiculo_estado` | rutas(id_vehiculo, estado) | Rutas activas |
| `ux_ruta_paradas_orden` | UNIQUE ruta_paradas(id_ruta, orden) | Orden de paradas |
| `ix_bitacora_estado_id` | bitacora_operaciones(estado, id_operacion) | Reconstrucción de la pila |

### 3.3 Vistas

| Vista | Columnas | Uso |
|---|---|---|
| `v_catalogo` | id_electrodomestico, sku, nombre, marca, categoria, precio_centavos (vigente), peso_kg, volumen_m3, activo, stock_total | Consultas parametrizadas de catálogo |
| `v_pedidos_pendientes` | id_pedido, folio, total_centavos, peso_kg, volumen_m3, codigo_postal, urgencia, tipo_entrega, fecha_creacion, id_nodo_destino | Reconstrucción de `ColaDespacho` |
| `v_paquetes` | id_envio, numero_guia, id_pedido, peso_kg, volumen_m3, id_nodo_destino, estado, urgencia | ABB de guías y FFD |

### 3.4 Mapeo de enumeraciones (C# ↔ TEXT)

`EstadoPedido { Pendiente, Asignado, EnRuta, Entregado, Cancelado, Devuelto }` ↔ `PENDIENTE…DEVUELTO`; `EstadoEnvio { Preparacion, EnTransito, Entregado, Devuelto, Cancelado }`; `TipoEntrega { Estandar, Express, Programada }`; `TipoNodo { AlmacenCentral, CentroAcopio, Domicilio }`; `TipoOperacion { Venta, Cancelacion, Devolucion, Reasignacion }`. Conversión centralizada en `ConversorValores`.

**Correspondencia pedido ↔ envío:** PENDIENTE/ASIGNADO ↔ PREPARACION; EN_RUTA ↔ EN_TRANSITO; ENTREGADO ↔ ENTREGADO; CANCELADO ↔ CANCELADO; DEVUELTO ↔ DEVUELTO.

**Datos del DML (F2):** 14 nodos con códigos postales de Monclova (25700–25799): 2 almacenes centrales, 3 centros de acopio y 9 domicilios, uno de ellos **solo alcanzable por un tramo cerrado** para demostrar el destino inalcanzable. Además: unos 30 tramos, 3 almacenes, 24 electrodomésticos de 6 marcas y 6 categorías (algunos con dos precios en el historial), 10 clientes, 5 vehículos, 12 pedidos en todos los estados con su detalle, envíos e historial, y bitácora coherente (ventas `APLICADA` de los pedidos pendientes y una `REVERTIDA`).

---

## 4. Reglas de diseño que fijan el comportamiento

1. **Escritura** (`EjecutorTransaccional`): `BEGIN IMMEDIATE → SQL → COMMIT → actualizar memoria`. Ante una excepción: `ROLLBACK`, se relanza (`ElectroException` tal cual; otras envueltas en `PersistenciaException`) y la memoria no se toca. Los DAO de escritura reciben `(SqliteConnection, SqliteTransaction)`; ninguno abre su propia transacción, salvo `IRegistroVentasDao`/`IFlotaDao.RegistrarAsignaciones`, que orquestan una transacción completa.
2. **Prioridad de despacho:** `Puntaje = urgencia × 10 + peso(tipo)`, con Express = 3, Estándar = 2 y Programada = 1 (rango 12–53). Desempate: fecha más antigua primero, luego id menor. El montículo es de **mínimo según el comparador**; `ComparadorPrioridadDespacho` devuelve negativo cuando `x` es más prioritario.
3. **Eliminar un pedido concreto de la cola en O(log n):** `ElementoDespacho` guarda `PosicionMonticulo`; `ColaDespacho` localiza el elemento por `idPedido` en O(1) con un **arreglo de dirección directa** (`ArregloDinamico<ElementoDespacho?>` indexado por id). No es una tabla hash: no hay función de dispersión ni colisiones.
4. **IDs del grafo → índices densos** mediante `ArbolBinarioBusqueda<long,int>` construido balanceado desde los ids ordenados. Los tramos se localizan con `ArbolBinarioBusqueda<long,AristaVial>`.
5. **ABB:** inserción, búsqueda, eliminación y recorridos son **iterativos** (recorridos con `PilaArreglo` propia), así que un árbol degenerado no puede desbordar la pila. Solo la construcción balanceada es recursiva, con profundidad máxima ⌈log₂(n+1)⌉ (17 para n = 100,000), registrada en `ProfundidadMaximaUltimaConstruccion`. Altura: árbol vacío = 0, un nodo = 1.
6. **El ABB de guías** indexa solo los envíos no cancelados: deshacer una venta y cancelar llaman a `Eliminar`; deshacer una cancelación vuelve a insertar.
7. **Operaciones inversas** (`IRegistroVentasDao.Revertir`), cada una en su propia transacción, que además marca la fila como `REVERTIDA`; el pop se hace solo después del COMMIT:

   | Operación | Directa | Inversa | Precondición de la inversa |
   |---|---|---|---|
   | VENTA | crea pedido + detalle + envío, descuenta stock, historial, encola | stock += cantidades, pedido → CANCELADO, envío → CANCELADO, anula asignación vigente, saca de la cola y del ABB de guías | pedido PENDIENTE o ASIGNADO |
   | CANCELACION | stock += cantidades, pedido/envío → CANCELADO, anula asignación | stock −= cantidades (falla con `StockInsuficienteException` si ya no hay), restaura estado anterior y asignación, reencola si PENDIENTE, reinserta guía | pedido CANCELADO |
   | DEVOLUCION | (pedido ENTREGADO) stock += cantidades, pedido/envío → DEVUELTO | stock −= cantidades, pedido/envío → ENTREGADO | pedido DEVUELTO |
   | REASIGNACION | anula la asignación vigente e inserta otra con el vehículo nuevo | anula la nueva y reactiva la anterior | asignación nueva vigente |

   `IBitacoraDao.MarcarRevertida` verifica que la operación sea la **APLICADA de mayor id** (LIFO también en la base).
8. **Ruteo dinámico, «solo rutas afectadas»:** cada `RutaPlanificada` conserva los árboles de Dijkstra que usó. Tras **cerrar** un tramo o **subir** su costo, la ruta queda afectada si el tramo pertenece a alguno de esos árboles de caminos mínimos (`ResultadoDijkstra.TramoPredecesor`). Tras **bajar** su costo o **abrirlo**, queda afectada si `d_s(u) + c' < d_s(v)` en algún árbol usado. Las demás rutas no se recalculan. Persistencia: transacción 1 = actualizar el tramo → COMMIT → grafo en memoria; transacción 2 = rutas nuevas (`id_ruta_anterior`) y las viejas como `RECALCULADA` → COMMIT → `RuteadorDinamico.Reemplazar`.
9. **FFD sin tocar la memoria antes del COMMIT:** `FirstFitDecreasing` simula con arreglos `double[]` de capacidad libre copiados de la flota. `ServicioAsignacion` persiste y, después del COMMIT, llama a `Vehiculo.Cargar` y a `ColaDespacho.EliminarPedido`. El lote se toma con `ColaDespacho.EnOrdenDePrioridad()`, que trabaja sobre una copia.
10. **Buffer circular lleno:** `Encolar` devuelve `false`, incrementa `Rechazados` y no sobrescribe nada (perder una solicitud de venta no es aceptable). `ServicioRecepcion` informa el rechazo al llamador. Las solicitudes en el buffer todavía no son ventas; solo existen cuando quedan en SQLite.
11. **Dinero:** `decimal` en el dominio y centavos `long` en SQLite (`ConversorValores.ACentavos/DesdeCentavos`).

---

## 5. Interfaces y clases públicas (firmas)

> Las firmas son definitivas. Los cuerpos llegan en la fase indicada. `internal set` es visible para `Electro.Tests` mediante `InternalsVisibleTo`.

### 5.1 `Electro.Core.Comun` [A]

```csharp
public class ElectroException : Exception { public ElectroException(string mensaje); public ElectroException(string mensaje, Exception interna); }
public sealed class DatosInvalidosException : ElectroException { public DatosInvalidosException(string campo, string mensaje); public string Campo { get; } }
public sealed class StockInsuficienteException : ElectroException { public StockInsuficienteException(string sku, long idAlmacen, int solicitado, int disponible); public string Sku { get; } public long IdAlmacen { get; } public int Solicitado { get; } public int Disponible { get; } }
public sealed class EntidadNoEncontradaException : ElectroException { public EntidadNoEncontradaException(string entidad, string clave); public string Entidad { get; } public string Clave { get; } }
public sealed class OperacionNoReversibleException : ElectroException { public OperacionNoReversibleException(string mensaje); }
public sealed class EstructuraVaciaException : ElectroException { public EstructuraVaciaException(string estructura); public string Estructura { get; } }
public sealed class PersistenciaException : ElectroException { public PersistenciaException(string mensaje, Exception interna); }

public static class Validar {
    public static T NoNulo<T>(T? valor, string campo) where T : class;
    public static string TextoNoVacio(string? valor, string campo);
    public static int Positivo(int valor, string campo);
    public static double Positivo(double valor, string campo);
    public static decimal Positivo(decimal valor, string campo);
    public static int EnRango(int valor, int minimo, int maximo, string campo);
    public static int NoNegativo(int valor, string campo);
}

public interface IReloj { DateTime Ahora { get; } }
public sealed class RelojSistema : IReloj { public DateTime Ahora { get; } }
public sealed class RelojFijo : IReloj { public RelojFijo(DateTime fecha); public DateTime Ahora { get; } public void Avanzar(TimeSpan lapso); }
```

### 5.2 `Electro.Core.Contratos` [A]

```csharp
public interface IComparador<in T> { int Comparar(T x, T y); }

public interface IPosicionMonticulo { int PosicionMonticulo { get; set; } }   // -1 = fuera del montículo

public interface IColaPrioridad<T> : IEnumerable<T> {
    int Cantidad { get; }
    bool EstaVacia { get; }
    void Insertar(T elemento);              // O(log n)
    T Extraer();                            // O(log n); EstructuraVaciaException si vacía
    T VerTope();                            // O(1)
    bool Eliminar(T elemento);              // O(log n)
    void Reubicar(T elemento);              // O(log n), tras cambiar la prioridad
    bool Contiene(T elemento);              // O(1)
    void Limpiar();
}

public interface IPila<T> : IEnumerable<T> {          // enumera de tope a fondo
    int Cantidad { get; }
    bool EstaVacia { get; }
    void Apilar(T elemento);
    T Desapilar();                          // pone en default la casilla liberada
    T VerTope();
    void Limpiar();
}

public interface IBufferPedidos<T> : IEnumerable<T> { // enumera de frente a final
    string Nombre { get; }
    int Cantidad { get; }
    int Capacidad { get; }                  // -1 = ilimitada
    bool EstaVacio { get; }
    bool EstaLleno { get; }
    bool Encolar(T elemento);               // false si está lleno
    T Desencolar();
    T VerFrente();
    long BytesEstimados();                  // estimación analítica de memoria
    void Limpiar();
}

public readonly struct ParClaveValor<TClave, TValor> {
    public ParClaveValor(TClave clave, TValor valor);
    public TClave Clave { get; }
    public TValor Valor { get; }
}

public interface IArbolBusqueda<TClave, TValor> {
    int Cantidad { get; }
    bool EstaVacio { get; }
    int Altura();
    bool Insertar(TClave clave, TValor valor);               // false si la clave ya existe
    bool Buscar(TClave clave, [MaybeNullWhen(false)] out TValor valor);
    bool Contiene(TClave clave);
    bool Reemplazar(TClave clave, TValor valor);
    bool Eliminar(TClave clave);                             // incluye el caso de nodo con dos hijos
    void ConstruirBalanceado(ParClaveValor<TClave, TValor>[] ordenados, int cantidad);
    IEnumerable<ParClaveValor<TClave, TValor>> Inorden();
    IEnumerable<ParClaveValor<TClave, TValor>> Preorden();
    IEnumerable<ParClaveValor<TClave, TValor>> Postorden();
    void Limpiar();
}

public interface IOrdenador<T> {
    string Nombre { get; }
    void Ordenar(T[] datos, int cantidad, IComparador<T> comparador);   // ordena datos[0..cantidad)
}
```

### 5.3 `Electro.Core.Dominio.Logistica` [A]: TDA enlazados por referencias

```csharp
public enum TipoNodo { AlmacenCentral, CentroAcopio, Domicilio }

public sealed class NodoDistribucion {
    public NodoDistribucion(long id, string codigo, string nombre, TipoNodo tipo, string codigoPostal);
    public long Id { get; } public string Codigo { get; } public string Nombre { get; }
    public TipoNodo Tipo { get; } public string CodigoPostal { get; }
    public int Indice { get; internal set; }                  // índice denso en el grafo, -1 fuera
    public AristaVial? PrimeraSalida { get; internal set; }   // cabeza de la lista de adyacencia
    public int GradoSalida { get; internal set; }
    public IEnumerable<AristaVial> Salidas(bool soloHabilitadas = true);
    public override string ToString();
}

public sealed class AristaVial {
    public AristaVial(long idTramo, NodoDistribucion origen, NodoDistribucion destino, double distanciaKm, double costo, bool habilitada);
    public long IdTramo { get; } public NodoDistribucion Origen { get; } public NodoDistribucion Destino { get; }
    public double DistanciaKm { get; }
    public double Costo { get; internal set; }
    public bool Habilitada { get; internal set; }
    public AristaVial? Siguiente { get; internal set; }       // siguiente salida del mismo origen
}

public sealed class Almacen {
    public Almacen(long id, string codigo, string nombre, bool esCentral, long idNodo);
    public long Id { get; } public string Codigo { get; } public string Nombre { get; } public bool EsCentral { get; } public long IdNodo { get; }
    public override string ToString();
}

public sealed class Vehiculo {
    public Vehiculo(long id, string placa, string descripcion, double capacidadVolumenM3, double capacidadPesoKg, long idAlmacenBase);
    public long Id { get; } public string Placa { get; } public string Descripcion { get; }
    public double CapacidadVolumenM3 { get; } public double CapacidadPesoKg { get; } public long IdAlmacenBase { get; }
    public double VolumenUsadoM3 { get; } public double PesoUsadoKg { get; }
    public double VolumenLibreM3 { get; } public double PesoLibreKg { get; }
    public bool Cabe(double volumenM3, double pesoKg);
    public void Cargar(double volumenM3, double pesoKg);      // DatosInvalidosException si no cabe
    public void Descargar(double volumenM3, double pesoKg);
    public void VaciarCarga();
    public Vehiculo? Siguiente { get; internal set; }
}

public sealed class FlotaVehicular : IEnumerable<Vehiculo> {   // lista simplemente enlazada con cabeza y cola
    public int Cantidad { get; }
    public Vehiculo? Primero { get; }
    public void Agregar(Vehiculo vehiculo);                   // O(1), al final
    public Vehiculo? Buscar(long idVehiculo);                 // O(n)
    public bool Quitar(long idVehiculo);                      // rompe el enlace Siguiente del quitado
    public void VaciarCargas();
}
```

### 5.4 `Electro.Core.Dominio.Ventas` [B]

```csharp
public enum EstadoPedido { Pendiente, Asignado, EnRuta, Entregado, Cancelado, Devuelto }
public enum EstadoEnvio { Preparacion, EnTransito, Entregado, Devuelto, Cancelado }
public enum TipoEntrega { Estandar, Express, Programada }

public sealed class Electrodomestico {
    public Electrodomestico(long id, string sku, string nombre, string marca, string categoria, decimal precio, double pesoKg, double volumenM3, bool activo = true);
    public long Id { get; internal set; } public string Sku { get; } public string Nombre { get; }
    public string Marca { get; } public string Categoria { get; }
    public decimal Precio { get; internal set; } public double PesoKg { get; } public double VolumenM3 { get; }
    public bool Activo { get; internal set; }
    public override string ToString();
}

public sealed class Cliente {
    public Cliente(long id, string nombre, string telefono, string correo, long idNodoDomicilio, string codigoPostal);
    public long Id { get; } public string Nombre { get; } public string Telefono { get; } public string Correo { get; }
    public long IdNodoDomicilio { get; } public string CodigoPostal { get; }
    public override string ToString();
}

public sealed class LineaPedido {
    public LineaPedido(long idElectrodomestico, string sku, int cantidad, decimal precioUnitario, double pesoUnitarioKg, double volumenUnitarioM3);
    public long IdElectrodomestico { get; } public string Sku { get; } public int Cantidad { get; }
    public decimal PrecioUnitario { get; } public double PesoUnitarioKg { get; } public double VolumenUnitarioM3 { get; }
    public decimal Subtotal { get; } public double PesoKg { get; } public double VolumenM3 { get; }
}

public sealed class Pedido {
    public Pedido(long id, string folio, long idCliente, long idAlmacenOrigen, DateTime fechaCreacion, EstadoPedido estado, int urgencia, TipoEntrega tipoEntrega, long idNodoDestino, string codigoPostal);
    public long Id { get; } public string Folio { get; } public long IdCliente { get; } public long IdAlmacenOrigen { get; }
    public DateTime FechaCreacion { get; } public EstadoPedido Estado { get; internal set; }
    public int Urgencia { get; } public TipoEntrega TipoEntrega { get; } public long IdNodoDestino { get; } public string CodigoPostal { get; }
    public int CantidadLineas { get; }
    public void AgregarLinea(LineaPedido linea);               // arreglo que crece a mano
    public LineaPedido LineaEn(int indice);
    public IEnumerable<LineaPedido> Lineas();
    public decimal Total(); public double PesoTotalKg(); public double VolumenTotalM3();
    public ResumenPedido ARResumen();
}

public sealed class ResumenPedido {                            // vista ligera para cola, ordenamiento y UI
    public ResumenPedido(long idPedido, string folio, decimal total, double pesoKg, double volumenM3, string codigoPostal, int urgencia, TipoEntrega tipoEntrega, DateTime fechaCreacion, long idNodoDestino);
    public long IdPedido { get; } public string Folio { get; } public decimal Total { get; }
    public double PesoKg { get; } public double VolumenM3 { get; } public string CodigoPostal { get; }
    public int Urgencia { get; } public TipoEntrega TipoEntrega { get; } public DateTime FechaCreacion { get; } public long IdNodoDestino { get; }
    public override string ToString();
}

public sealed class Paquete {                                  // envío físico de un pedido
    public Paquete(long idEnvio, string numeroGuia, long idPedido, double pesoKg, double volumenM3, long idNodoDestino, EstadoEnvio estado, int urgencia);
    public long IdEnvio { get; } public string NumeroGuia { get; } public long IdPedido { get; }
    public double PesoKg { get; } public double VolumenM3 { get; } public long IdNodoDestino { get; }
    public EstadoEnvio Estado { get; internal set; } public int Urgencia { get; }
    public override string ToString();
}

public sealed class RenglonSolicitud { public RenglonSolicitud(string sku, int cantidad); public string Sku { get; } public int Cantidad { get; } }

public sealed class SolicitudVenta {
    public SolicitudVenta(long idCliente, long idAlmacen, int urgencia, TipoEntrega tipoEntrega);
    public long IdCliente { get; } public long IdAlmacen { get; } public int Urgencia { get; } public TipoEntrega TipoEntrega { get; }
    public long? IdNodoEntrega { get; set; }                   // null = domicilio del cliente
    public int CantidadRenglones { get; }
    public void AgregarRenglon(string sku, int cantidad);      // suma si el SKU se repite
    public RenglonSolicitud RenglonEn(int indice);
    public IEnumerable<RenglonSolicitud> Renglones();
    public void Validar();                                     // DatosInvalidosException
}
```

### 5.5 `Electro.Core.Estructuras.*` [B]

```csharp
// Lineales
public sealed class NodoSimple<T> { public NodoSimple(T valor); public T Valor { get; set; } public NodoSimple<T>? Siguiente { get; set; } }

public sealed class ArregloDinamico<T> : IEnumerable<T> {     // duplica capacidad; libera casillas con default
    public ArregloDinamico(int capacidadInicial = 8);
    public int Cantidad { get; } public int Capacidad { get; }
    public T this[int indice] { get; set; }
    public void Agregar(T elemento);
    public void InsertarEn(int indice, T elemento);
    public T EliminarEn(int indice);
    public void Limpiar();
    public T[] ACopia();                                       // arreglo exacto de Cantidad elementos
    public static ArregloDinamico<T> Desde(T[] origen, int cantidad);
}

public sealed class ColaCircular<T> : IBufferPedidos<T> {     // arreglo estático; frente/final módulo capacidad
    public ColaCircular(int capacidad);
    public long Rechazados { get; }                            // intentos de Encolar estando lleno
    /* + miembros de IBufferPedidos<T> */
}

public sealed class ListaEnlazadaCola<T> : IBufferPedidos<T> { // nodos NodoSimple<T>, cabeza y cola
    public ListaEnlazadaCola();
    /* + miembros de IBufferPedidos<T>; Capacidad = -1, EstaLleno = false */
}

public sealed class ComparacionBuffers {
    public int Capacidad { get; } public int Elementos { get; }
    public long BytesMedidosCircular { get; } public long BytesMedidosEnlazada { get; }
    public long BytesEstimadosCircular { get; } public long BytesEstimadosEnlazada { get; }
}
public static class MedidorMemoria {
    public static long MedirBytesAsignados(Action accion);     // GC.GetAllocatedBytesForCurrentThread
    public static long EstimarBytesColaCircular(int capacidad);
    public static long EstimarBytesListaEnlazada(int elementos);
    public static ComparacionBuffers CompararBuffers(int capacidad, int elementos);
}

public sealed class PilaArreglo<T> : IPila<T> { public PilaArreglo(int capacidadInicial = 16); /* + IPila<T> */ }

// Monticulo
public sealed class MonticuloBinario<T> : IColaPrioridad<T> where T : class, IPosicionMonticulo {
    public MonticuloBinario(IComparador<T> comparador, int capacidadInicial = 16);   // tope = mínimo según comparador
    /* + IColaPrioridad<T>; Extraer/Eliminar ponen null la casilla y PosicionMonticulo = -1 */
}

// Arboles
public sealed class NodoArbol<TClave, TValor> {
    public NodoArbol(TClave clave, TValor valor);
    public TClave Clave { get; internal set; } public TValor Valor { get; internal set; }
    public NodoArbol<TClave, TValor>? Izquierdo { get; internal set; }
    public NodoArbol<TClave, TValor>? Derecho { get; internal set; }
}

public sealed class PasoConstruccionAbb<TClave> {
    public int Paso { get; } public int Profundidad { get; } public int Inicio { get; } public int Fin { get; } public int Medio { get; }
    public TClave Clave { get; } public string Lado { get; }   // "raíz" | "izquierdo" | "derecho"
    public string ClavePadre { get; }
}

public sealed class ComparadorEntero64 : IComparador<long> { public int Comparar(long x, long y); }
public sealed class ComparadorTextoOrdinal : IComparador<string> { public int Comparar(string x, string y); }

public sealed class ArbolBinarioBusqueda<TClave, TValor> : IArbolBusqueda<TClave, TValor> {
    public ArbolBinarioBusqueda(IComparador<TClave> comparador);
    public NodoArbol<TClave, TValor>? Raiz { get; }
    public int ProfundidadMaximaUltimaConstruccion { get; }
    public long ComparacionesUltimaBusqueda { get; }
    public ArregloDinamico<PasoConstruccionAbb<TClave>> ConstruirBalanceadoConTraza(ParClaveValor<TClave, TValor>[] ordenados, int cantidad);
    public static int AlturaTeoricaMinima(int n);              // ⌈log₂(n+1)⌉
    /* + IArbolBusqueda<TClave,TValor> */
}
```

### 5.6 `Electro.Core.Despacho` [B]

```csharp
public static class CalculadoraPrioridad {
    public const int UrgenciaMinima = 1; public const int UrgenciaMaxima = 5;
    public static int PesoTipoEntrega(TipoEntrega tipo);       // Express 3, Estandar 2, Programada 1
    public static int Puntaje(int urgencia, TipoEntrega tipo); // urgencia*10 + peso
}

public sealed class ElementoDespacho : IPosicionMonticulo {
    public ElementoDespacho(ResumenPedido pedido);
    public ResumenPedido Pedido { get; } public int Puntaje { get; } public int PosicionMonticulo { get; set; }
}
public sealed class ComparadorPrioridadDespacho : IComparador<ElementoDespacho> { public int Comparar(ElementoDespacho x, ElementoDespacho y); }

public sealed class ColaDespacho : IEnumerable<ResumenPedido> {        // enumera en orden interno del montículo
    public ColaDespacho(int capacidadInicial = 64);
    public int Cantidad { get; } public bool EstaVacia { get; }
    public void Encolar(ResumenPedido pedido);                 // DatosInvalidosException si ya está
    public ResumenPedido ExtraerSiguiente();
    public ResumenPedido VerSiguiente();
    public bool EliminarPedido(long idPedido);                 // O(log n)
    public bool Contiene(long idPedido);                       // O(1)
    public ResumenPedido[] EnOrdenDePrioridad();               // copia; no altera la cola
    public void Limpiar();
}

public enum TipoOperacion { Venta, Cancelacion, Devolucion, Reasignacion }

public sealed class OperacionRegistrada {
    public OperacionRegistrada(long idOperacion, TipoOperacion tipo, string cargaUtilJson, DateTime fecha);
    public long IdOperacion { get; } public TipoOperacion Tipo { get; } public string CargaUtilJson { get; } public DateTime Fecha { get; }
    public string Descripcion();
}

public sealed class LineaCarga { public long IdElectrodomestico { get; init; } public string Sku { get; init; } = ""; public int Cantidad { get; init; } }
public sealed class CargaVenta { public long IdPedido { get; init; } public long IdEnvio { get; init; } public string NumeroGuia { get; init; } = ""; public long IdAlmacen { get; init; } public LineaCarga[] Lineas { get; init; } = []; }
public sealed class CargaCancelacion { public long IdPedido { get; init; } public long IdEnvio { get; init; } public string NumeroGuia { get; init; } = ""; public long IdAlmacen { get; init; } public EstadoPedido EstadoAnterior { get; init; } public long? IdAsignacionAnulada { get; init; } public LineaCarga[] Lineas { get; init; } = []; }
public sealed class CargaDevolucion { public long IdPedido { get; init; } public long IdEnvio { get; init; } public long IdAlmacen { get; init; } public LineaCarga[] Lineas { get; init; } = []; }
public sealed class CargaReasignacion { public long IdEnvio { get; init; } public long IdAsignacionAnterior { get; init; } public long IdAsignacionNueva { get; init; } public long IdVehiculoAnterior { get; init; } public long IdVehiculoNuevo { get; init; } }

public static class SerializadorCargas {                      // System.Text.Json, enums como texto
    public static string Serializar<TCarga>(TCarga carga) where TCarga : class;
    public static TCarga Deserializar<TCarga>(string json) where TCarga : class;
}

public sealed class PilaDeshacer : IEnumerable<OperacionRegistrada> {  // envuelve PilaArreglo<OperacionRegistrada>
    public int Cantidad { get; } public bool EstaVacia { get; }
    public void Apilar(OperacionRegistrada operacion);
    public OperacionRegistrada VerTope();
    public OperacionRegistrada Desapilar();
    public void CargarDesdeBitacora(ArregloDinamico<OperacionRegistrada> aplicadasAscendente);
    public void Limpiar();
}
```

### 5.7 `Electro.Core.Asignacion` [A]

```csharp
public sealed class ComparadorPaqueteVolumenDesc : IComparador<Paquete> { public int Comparar(Paquete x, Paquete y); } // volumen ↓, peso ↓, id ↑
public sealed class AsignacionPaquete { public AsignacionPaquete(Paquete paquete, Vehiculo vehiculo); public Paquete Paquete { get; } public Vehiculo Vehiculo { get; } }
public sealed class ResultadoAsignacion {
    public AsignacionPaquete[] Asignaciones { get; } public int CantidadAsignados { get; }
    public Paquete[] NoAsignados { get; } public int CantidadNoAsignados { get; }
    public int VehiculosUsados { get; }
    public double VolumenAsignadoA(long idVehiculo); public double PesoAsignadoA(long idVehiculo);
}
public static class FirstFitDecreasing {
    public static ResultadoAsignacion Asignar(Paquete[] paquetes, int cantidad, FlotaVehicular flota); // no modifica entrada ni flota
}
```

### 5.8 `Electro.Core.Caminos` [B]

```csharp
public sealed class ResultadoCamino {
    public bool Alcanzable { get; } public int Origen { get; } public int Destino { get; }
    public double CostoTotal { get; } public int[] Nodos { get; } public int Longitud { get; }
    public int ProfundidadRecursion { get; } public string Motivo { get; }
    public static ResultadoCamino Inalcanzable(int origen, int destino, string motivo);
}
public static class ReconstructorCamino {
    public static ResultadoCamino Reconstruir(int[] predecesores, double[] distancias, int origen, int destino); // recursivo; profundidad ≤ V
}
public sealed class TramoHojaDeRuta {
    public int Numero { get; } public string Desde { get; } public string Hasta { get; } public string Recorrido { get; }
    public double Costo { get; } public double DistanciaKm { get; } public double CostoAcumulado { get; } public string NumeroGuia { get; }
}
public sealed class HojaDeRuta {
    public string Vehiculo { get; } public string Origen { get; } public DateTime Fecha { get; }
    public TramoHojaDeRuta[] Tramos { get; } public int CantidadTramos { get; }
    public string[] Inalcanzables { get; } public int CantidadInalcanzables { get; }
    public double CostoTotal { get; } public double DistanciaTotalKm { get; }
    public string ATexto();
}
public static class GeneradorHojaDeRuta {
    public static HojaDeRuta Generar(GrafoLogistico grafo, RutaPlanificada ruta, string placaVehiculo, DateTime fecha, Func<long, string>? guiaDeEnvio = null);
}
```

### 5.9 `Electro.Core.Grafos` [A]

```csharp
public sealed class GrafoLogistico {
    public GrafoLogistico(int capacidadInicial = 16);
    public int CantidadNodos { get; } public int CantidadAristas { get; }
    public int AgregarNodo(NodoDistribucion nodo);             // índice denso
    public AristaVial AgregarArista(long idTramo, long idNodoOrigen, long idNodoDestino, double distanciaKm, double costo, bool habilitada);
    public void IndexarIds();                                  // reconstruye ABB id→índice balanceado tras la carga masiva
    public int IndiceDe(long idNodo);                          // -1 si no existe
    public NodoDistribucion NodoEn(int indice);
    public AristaVial? BuscarTramo(long idTramo);
    public void CerrarTramo(long idTramo); public void AbrirTramo(long idTramo); public void CambiarCosto(long idTramo, double nuevoCosto);
    public IEnumerable<NodoDistribucion> Nodos(); public IEnumerable<AristaVial> Aristas();
}

public sealed class EntradaDijkstra : IPosicionMonticulo { public EntradaDijkstra(int indice, double distancia); public int Indice { get; } public double Distancia { get; } public int PosicionMonticulo { get; set; } }
public sealed class ComparadorEntradaDijkstra : IComparador<EntradaDijkstra> { public int Comparar(EntradaDijkstra x, EntradaDijkstra y); }

public sealed class PasoTrazaDijkstra {
    public int Iteracion { get; } public int NodoExtraido { get; } public string CodigoNodo { get; }
    public double DistanciaExtraida { get; } public bool Obsoleta { get; } public string Relajaciones { get; }
    public double[] Distancias { get; } public int[] Predecesores { get; }      // instantánea tras la iteración
}

public sealed class ResultadoDijkstra {
    public int Origen { get; } public double[] Distancias { get; } public int[] Predecesores { get; }
    public long[] TramoPredecesor { get; }                    // id del tramo con el que se llegó; -1
    public ArregloDinamico<PasoTrazaDijkstra>? Traza { get; }
    public int Iteraciones { get; } public int EntradasObsoletas { get; }
    public bool EsAlcanzable(int destino);
    public ResultadoCamino CaminoA(int destino);              // usa ReconstructorCamino (B)
    public bool TramoEnArbol(long idTramo);
}
public static class Dijkstra { public static ResultadoDijkstra Ejecutar(GrafoLogistico grafo, int origen, bool registrarTraza = false); } // MonticuloBinario de B, inserción perezosa

public static class ExportadorTrazaDijkstra {
    public static string ATablaMarkdown(GrafoLogistico grafo, ResultadoDijkstra resultado);
    public static void ACsv(GrafoLogistico grafo, ResultadoDijkstra resultado, string rutaArchivo);
}

public sealed class DestinoEntrega { public DestinoEntrega(int indiceNodo, long? idEnvio); public int IndiceNodo { get; } public long? IdEnvio { get; } }
public sealed class ParadaPlanificada {
    public int Orden { get; } public int IndiceNodo { get; } public long? IdEnvio { get; }
    public double CostoDesdeAnterior { get; } public double DistanciaKmDesdeAnterior { get; } public double CostoAcumulado { get; }
    public ResultadoCamino Camino { get; }
}
public sealed class RutaPlanificada {
    public long IdRuta { get; internal set; } public long IdVehiculo { get; } public int Origen { get; }
    public ParadaPlanificada[] Paradas { get; } public int CantidadParadas { get; }
    public DestinoEntrega[] Inalcanzables { get; } public int CantidadInalcanzables { get; }
    public double CostoTotal { get; } public double DistanciaTotalKm { get; }
    public ResultadoDijkstra[] ArbolesUsados { get; } public int CantidadArboles { get; }
    public DestinoEntrega[] Destinos();                       // los originales, para recalcular
}
public static class VecinoMasCercano {
    public static RutaPlanificada Planificar(GrafoLogistico grafo, long idVehiculo, int origen, DestinoEntrega[] destinos, int cantidad);
    public static RutaPlanificada ReconstruirConOrden(GrafoLogistico grafo, long idVehiculo, int origen, DestinoEntrega[] enOrden, int cantidad);
}

public enum TipoCambioTramo { Cierre, Apertura, CambioCosto }
public sealed class ResultadoRecalculo {
    public long IdTramo { get; } public TipoCambioTramo Tipo { get; } public double CostoAnterior { get; } public double CostoNuevo { get; }
    public int RutasEvaluadas { get; } public int CantidadAfectadas { get; }
    public RutaPlanificada[] RutasAnteriores { get; } public RutaPlanificada[] RutasNuevas { get; }
}
public sealed class RuteadorDinamico {
    public RuteadorDinamico(GrafoLogistico grafo);
    public int CantidadRutas { get; }
    public void Registrar(RutaPlanificada ruta);
    public void Reemplazar(long idRutaAnterior, RutaPlanificada nueva);
    public bool Quitar(long idRuta);
    public IEnumerable<RutaPlanificada> Rutas();
    public bool EsAfectada(RutaPlanificada ruta, AristaVial tramo, double costoAnterior, TipoCambioTramo tipo);
    public ResultadoRecalculo CalcularRecalculo(long idTramo, double costoAnterior, TipoCambioTramo tipo); // puro: no reemplaza
}
```

### 5.10 `Electro.Core.Ordenamiento` [B; `MergeSort` C]

```csharp
public enum CriterioOrden { Precio, Peso, CodigoPostal, Urgencia }
public sealed class ComparadorPorPrecio : IComparador<ResumenPedido> { public int Comparar(ResumenPedido x, ResumenPedido y); }      // ascendente
public sealed class ComparadorPorPeso : IComparador<ResumenPedido> { public int Comparar(ResumenPedido x, ResumenPedido y); }
public sealed class ComparadorPorCodigoPostal : IComparador<ResumenPedido> { public int Comparar(ResumenPedido x, ResumenPedido y); } // ordinal
public sealed class ComparadorPorUrgencia : IComparador<ResumenPedido> { public int Comparar(ResumenPedido x, ResumenPedido y); }    // 1..5
public sealed class ComparadorInverso<T> : IComparador<T> { public ComparadorInverso(IComparador<T> baseComparador); public int Comparar(T x, T y); }
public static class FabricaComparadores { public static IComparador<ResumenPedido> Crear(CriterioOrden criterio, bool descendente = false); }

public sealed class QuickSort3Vias<T> : IOrdenador<T> {        // mediana de tres + Dijkstra 3-way; recursión en la partición menor
    public string Nombre { get; } public int ProfundidadMaximaUltimaEjecucion { get; }
    public void Ordenar(T[] datos, int cantidad, IComparador<T> comparador);
}
public sealed class MergeSort<T> : IOrdenador<T> {             // [C] top-down, estable, un búfer auxiliar de n
    public string Nombre { get; } public void Ordenar(T[] datos, int cantidad, IComparador<T> comparador);
}
```

### 5.11 `Electro.Core.Rendimiento` [A]

```csharp
public sealed class InsertionSort<T> : IOrdenador<T> { public const int LimiteElementos = 10_000; public string Nombre { get; } public void Ordenar(T[] datos, int cantidad, IComparador<T> comparador); } // ArgumentOutOfRange si cantidad > 10,000

public static class Estadistica { public static double Promedio(double[] valores, int n); public static double DesviacionEstandar(double[] valores, int n); } // muestral (n-1)

public enum EscenarioDatos { Aleatorio, Ordenado }
public sealed class ConfiguracionBenchmark {
    public int[] Tamanos { get; init; } = [10_000, 50_000, 100_000];
    public int CorridasCalentamiento { get; init; } = 1;
    public int CorridasMedidas { get; init; } = 5;
    public int Semilla { get; init; } = 2026;
    public static ConfiguracionBenchmark PorDefecto();
    public static ConfiguracionBenchmark Rapida();           // tamaños pequeños, para pruebas unitarias
}
public class MedicionBenchmark {
    public MedicionBenchmark(string algoritmo, string escenario, int n, int corridas, double promedioMs, double desviacionMs);
    public string Algoritmo { get; } public string Escenario { get; } public int N { get; } public int Corridas { get; }
    public double PromedioMs { get; } public double DesviacionMs { get; }
}
public sealed class MedicionBusquedaAbb : MedicionBenchmark {
    public MedicionBusquedaAbb(string algoritmo, string escenario, int n, int corridas, double promedioMs, double desviacionMs, int alturaArbol, int alturaTeorica, double comparacionesPromedio);
    public int AlturaArbol { get; } public int AlturaTeorica { get; } public double ComparacionesPromedio { get; }
}
public static class GeneradorDatos {
    public static ResumenPedido[] GenerarResumenes(int n, int semilla);          // urgencia 1..5 (muchos repetidos)
    public static long[] GenerarClavesUnicas(int n, int semilla);
    public static Paquete[] GenerarPaquetes(int n, int semilla);
    public static SolicitudVenta[] GenerarSolicitudes(int n, int semilla, long[] idsClientes, string[] skus, long idAlmacen);
}
public static class CurvaTeorica {
    public static double NLogN(int n); public static double NCuadrada(int n); public static double LogN(int n);
    public static double Factor(double empiricoPrimerPunto, double teoricoPrimerPunto);
}
public static class ExportadorCsv {                            // CultureInfo.InvariantCulture
    public static void EscribirMediciones(string rutaArchivo, ArregloDinamico<MedicionBenchmark> mediciones); // + columnas teórica_nlogn, teórica_n2, teórica_logn escaladas
    public static void EscribirBusquedaAbb(string rutaArchivo, ArregloDinamico<MedicionBusquedaAbb> mediciones);
}
public sealed class BenchmarkBusquedaAbb {
    public BenchmarkBusquedaAbb(ConfiguracionBenchmark configuracion);
    public ArregloDinamico<MedicionBusquedaAbb> Ejecutar(int busquedasPorCorrida = 10_000);  // ABB vs secuencial en arreglo
}
public sealed class BancoBenchmarks {
    public BancoBenchmarks(ConfiguracionBenchmark configuracion);
    public ArregloDinamico<MedicionBenchmark> EjecutarOrdenamientos(CriterioOrden criterio); // QuickSort, MergeSort, InsertionSort (≤10,000) × aleatorio/ordenado
}
```

### 5.12 `Electro.Core.Datos` [C]

```csharp
public interface IFabricaConexiones { string RutaBaseDatos { get; } SqliteConnection Abrir(); }  // abierta + PRAGMA foreign_keys = ON
public sealed class FabricaConexiones : IFabricaConexiones { public FabricaConexiones(string rutaBaseDatos); public string RutaBaseDatos { get; } public SqliteConnection Abrir(); }

public sealed class EjecutorTransaccional {
    public EjecutorTransaccional(IFabricaConexiones fabrica);
    public T Ejecutar<T>(Func<SqliteConnection, SqliteTransaction, T> trabajo);
    public void Ejecutar(Action<SqliteConnection, SqliteTransaction> trabajo);
}

public static class InicializadorBaseDatos {
    public static void Crear(string rutaBaseDatos, bool incluirDatosPrueba = true, bool sobrescribir = false);
    public static bool EsquemaExiste(IFabricaConexiones fabrica);
    public static string LeerScript(string nombreArchivo);     // "DDL.sql" | "DML.sql" (recurso embebido)
    public static void EjecutarScript(SqliteConnection conexion, SqliteTransaction transaccion, string sql);
}

public static class ConversorValores {
    public static string ATexto(EstadoPedido v); public static string ATexto(EstadoEnvio v); public static string ATexto(TipoEntrega v);
    public static string ATexto(TipoNodo v); public static string ATexto(TipoOperacion v);
    public static EstadoPedido EstadoPedidoDesde(string t); public static EstadoEnvio EstadoEnvioDesde(string t);
    public static TipoEntrega TipoEntregaDesde(string t); public static TipoNodo TipoNodoDesde(string t); public static TipoOperacion TipoOperacionDesde(string t);
    public static long ACentavos(decimal monto); public static decimal DesdeCentavos(long centavos);
    public static string FechaATexto(DateTime fecha); public static DateTime FechaDesde(string texto);
}

public static class GeneradorFolios { public static string FolioPedido(long idPedido); public static string NumeroGuia(long idEnvio, DateTime fecha); }

public sealed class FiltroCatalogo {
    public decimal? PrecioMinimo { get; init; } public decimal? PrecioMaximo { get; init; }
    public string? Marca { get; init; } public string? Categoria { get; init; } public string? TextoNombre { get; init; }
    public bool SoloActivos { get; init; } = true; public int Limite { get; init; } = 500;
}
```

**`Electro.Core.Datos.Dao`: DTO de lectura (`RegistrosDatos.cs`)**

```csharp
public sealed class ExistenciaAlmacen { public long IdAlmacen { get; init; } public string Almacen { get; init; } = ""; public int Stock { get; init; } public int StockMinimo { get; init; } }
public sealed class HistorialEstadoPedido { public long IdHistorial { get; init; } public EstadoPedido? Anterior { get; init; } public EstadoPedido Nuevo { get; init; } public DateTime Fecha { get; init; } public string Motivo { get; init; } = ""; }
public sealed class AsignacionVigente { public long IdAsignacion { get; init; } public long IdEnvio { get; init; } public long IdVehiculo { get; init; } public long IdNodoDestino { get; init; } public double VolumenM3 { get; init; } public double PesoKg { get; init; } public DateTime Fecha { get; init; } }
public sealed class RegistroTramo { public long IdTramo { get; init; } public long IdNodoOrigen { get; init; } public long IdNodoDestino { get; init; } public double DistanciaKm { get; init; } public double Costo { get; init; } public bool Habilitado { get; init; } }
public sealed class RegistroRuta { public long IdRuta { get; init; } public long IdVehiculo { get; init; } public long IdNodoOrigen { get; init; } public string Estado { get; init; } = ""; public double CostoTotal { get; init; } public DateTime Fecha { get; init; } }
public sealed class RegistroParada { public long IdParada { get; init; } public int Orden { get; init; } public long IdNodo { get; init; } public long? IdEnvio { get; init; } public double DistanciaKm { get; init; } public double Costo { get; init; } public double CostoAcumulado { get; init; } public string EstadoEntrega { get; init; } = ""; }
```

**Resultados de escritura (`ResultadosPersistencia.cs`)**

```csharp
[Flags] public enum AccionMemoria { Ninguna = 0, QuitarDeCola = 1, ReinsertarEnCola = 2, QuitarGuia = 4, ReinsertarGuia = 8, LiberarCargaVehiculo = 16, RestaurarCargaVehiculo = 32, MoverCargaVehiculo = 64, ActualizarEstadoGuia = 128 }
public sealed class ResultadoVenta { public long IdOperacion { get; init; } public required Pedido Pedido { get; init; } public required ResumenPedido Resumen { get; init; } public required Paquete Paquete { get; init; } }
public sealed class ResultadoOperacion { public long IdOperacion { get; init; } public required ResumenPedido Resumen { get; init; } public required Paquete Paquete { get; init; } public EstadoPedido EstadoAnterior { get; init; } public long? IdVehiculoLiberado { get; init; } public AccionMemoria Acciones { get; init; } }
public sealed class ResultadoReasignacion { public long IdOperacion { get; init; } public long IdEnvio { get; init; } public long IdVehiculoAnterior { get; init; } public long IdVehiculoNuevo { get; init; } public double VolumenM3 { get; init; } public double PesoKg { get; init; } }
public sealed class ResultadoReversion { public required OperacionRegistrada Operacion { get; init; } public ResumenPedido? Resumen { get; init; } public Paquete? Paquete { get; init; } public long? IdVehiculoOrigen { get; init; } public long? IdVehiculoDestino { get; init; } public double VolumenM3 { get; init; } public double PesoKg { get; init; } public AccionMemoria Acciones { get; init; } }
```

**Contratos DAO (`ContratosDao.cs`)**: cada interfaz tiene su implementación `XxxDao(IFabricaConexiones)` en un archivo propio.

```csharp
public interface IClienteDao { ArregloDinamico<Cliente> ObtenerTodos(); Cliente? ObtenerPorId(long idCliente); }

public interface IInventarioDao {
    int ObtenerStock(long idAlmacen, long idElectrodomestico);
    ArregloDinamico<ExistenciaAlmacen> ObtenerExistencias(long idElectrodomestico);
    void Descontar(SqliteConnection c, SqliteTransaction t, long idAlmacen, long idElectrodomestico, string sku, int cantidad); // StockInsuficienteException
    void Reponer(SqliteConnection c, SqliteTransaction t, long idAlmacen, long idElectrodomestico, int cantidad);
    void Establecer(SqliteConnection c, SqliteTransaction t, long idAlmacen, long idElectrodomestico, int stock);
}

public interface IPedidoDao {
    Pedido? ObtenerPorId(long idPedido);                       // con líneas
    ArregloDinamico<ResumenPedido> ObtenerPendientesDespacho();
    ArregloDinamico<ResumenPedido> ObtenerResumenes(EstadoPedido? estado);
    ArregloDinamico<HistorialEstadoPedido> ObtenerHistorial(long idPedido);
    EstadoPedido ObtenerEstado(SqliteConnection c, SqliteTransaction t, long idPedido);
    long Insertar(SqliteConnection c, SqliteTransaction t, long idCliente, long idAlmacen, DateTime fecha, int urgencia, TipoEntrega tipo, out string folio);
    void InsertarLinea(SqliteConnection c, SqliteTransaction t, long idPedido, LineaPedido linea);
    void CambiarEstado(SqliteConnection c, SqliteTransaction t, long idPedido, EstadoPedido? esperado, EstadoPedido nuevo, string motivo, DateTime fecha); // + historial
}

public interface IEnvioDao {
    Paquete? ObtenerPorGuia(string numeroGuia);
    Paquete? ObtenerPorPedido(long idPedido);
    ArregloDinamico<Paquete> ObtenerActivosOrdenadosPorGuia();
    long Insertar(SqliteConnection c, SqliteTransaction t, long idPedido, long idNodoDestino, DateTime fecha, out string numeroGuia);
    void CambiarEstado(SqliteConnection c, SqliteTransaction t, long idEnvio, EstadoEnvio nuevo);
    Paquete ObtenerPorId(SqliteConnection c, SqliteTransaction t, long idEnvio);
}

public interface IFlotaDao {
    ArregloDinamico<Vehiculo> ObtenerVehiculosActivos();
    ArregloDinamico<AsignacionVigente> ObtenerAsignacionesVigentes();          // de envíos no entregados ni cancelados
    AsignacionVigente? ObtenerAsignacionVigente(SqliteConnection c, SqliteTransaction t, long idEnvio);
    long InsertarAsignacion(SqliteConnection c, SqliteTransaction t, long idEnvio, long idVehiculo, DateTime fecha);
    void AnularAsignacion(SqliteConnection c, SqliteTransaction t, long idAsignacion);
    void ReactivarAsignacion(SqliteConnection c, SqliteTransaction t, long idAsignacion);
    long[] RegistrarAsignaciones(ResultadoAsignacion resultado, DateTime fecha);  // transacción completa; pedidos → ASIGNADO
}

public interface IBitacoraDao {
    long Registrar(SqliteConnection c, SqliteTransaction t, TipoOperacion tipo, string cargaJson, DateTime fecha);
    void MarcarRevertida(SqliteConnection c, SqliteTransaction t, long idOperacion, DateTime fecha); // exige que sea la APLICADA de mayor id
    ArregloDinamico<OperacionRegistrada> ObtenerAplicadasAscendente();
}

public interface IRegistroVentasDao {                          // cada método = una transacción completa
    ResultadoVenta RegistrarVenta(SolicitudVenta solicitud, DateTime fecha);
    ResultadoOperacion CancelarPedido(long idPedido, string motivo, DateTime fecha);
    ResultadoOperacion DevolverPedido(long idPedido, string motivo, DateTime fecha);
    ResultadoReasignacion ReasignarEnvio(long idEnvio, long idVehiculoNuevo, DateTime fecha);
    ResultadoOperacion AvanzarEstado(long idPedido, EstadoPedido nuevo, DateTime fecha);       // EN_RUTA / ENTREGADO, no revertible
    ResultadoReversion Revertir(OperacionRegistrada operacion, DateTime fecha);
}

public interface ICatalogoDao {                                // (F4)
    ArregloDinamico<Electrodomestico> ObtenerActivosOrdenadosPorSku();
    Electrodomestico? BuscarPorSku(string sku);
    ArregloDinamico<Electrodomestico> Filtrar(FiltroCatalogo filtro);          // parametrizada
    ArregloDinamico<string> ObtenerMarcas();
    ArregloDinamico<string> ObtenerCategorias();
    long Insertar(SqliteConnection c, SqliteTransaction t, Electrodomestico e, DateTime fecha);
    void CambiarPrecio(SqliteConnection c, SqliteTransaction t, long idElectrodomestico, decimal nuevoPrecio, DateTime fecha);
    void DarDeBaja(SqliteConnection c, SqliteTransaction t, long idElectrodomestico);
}

public interface IRedDao {                                     // (F5)
    ArregloDinamico<NodoDistribucion> ObtenerNodosOrdenadosPorId();
    ArregloDinamico<Almacen> ObtenerAlmacenes();
    ArregloDinamico<RegistroTramo> ObtenerTramos();
    void ActualizarCosto(SqliteConnection c, SqliteTransaction t, long idTramo, double nuevoCosto, DateTime fecha);
    void CambiarHabilitado(SqliteConnection c, SqliteTransaction t, long idTramo, bool habilitado, DateTime fecha);
}

public interface IRutaDao {                                    // (F5)
    long InsertarRuta(SqliteConnection c, SqliteTransaction t, RutaPlanificada ruta, GrafoLogistico grafo, DateTime fecha, long? idRutaAnterior);
    void MarcarRecalculada(SqliteConnection c, SqliteTransaction t, long idRuta);
    ArregloDinamico<RegistroRuta> ObtenerRutasActivas();
    ArregloDinamico<RegistroParada> ObtenerParadas(long idRuta);
    void ActualizarEstadoParada(SqliteConnection c, SqliteTransaction t, long idParada, string estadoEntrega);
}
```

**`Electro.Core.Datos.Sincronizacion` / `.Respaldo` / `.Rendimiento`**

```csharp
public sealed class EstadoMemoria {
    public ArbolBinarioBusqueda<string, Electrodomestico> IndiceSku { get; internal set; }   // (F4)
    public ArbolBinarioBusqueda<string, Paquete> IndiceGuias { get; internal set; }          // (F4)
    public ColaDespacho Cola { get; internal set; }
    public PilaDeshacer Pila { get; internal set; }
    public FlotaVehicular Flota { get; internal set; }
    public GrafoLogistico Grafo { get; internal set; }                                       // (F5)
    public RuteadorDinamico Ruteador { get; internal set; }                                  // (F5)
}
public sealed class CargadorMemoria {
    public CargadorMemoria(IFabricaConexiones fabrica);
    public EstadoMemoria CargarTodo();
    public void RecargarEn(EstadoMemoria destino);
    public ColaDespacho CargarCola(); public PilaDeshacer CargarPila(); public FlotaVehicular CargarFlota();
    public ArbolBinarioBusqueda<string, Electrodomestico> CargarCatalogo();                   // central recursivo
    public ArbolBinarioBusqueda<string, Paquete> CargarGuias();
    public GrafoLogistico CargarGrafo(); public RuteadorDinamico CargarRutas(GrafoLogistico grafo);
}

public sealed class ResultadoIntegridad { public bool Correcta { get; init; } public string IntegrityCheck { get; init; } = ""; public int ViolacionesLlaveForanea { get; init; } public string Detalle { get; init; } = ""; }
public sealed class ServicioRespaldo {
    public ServicioRespaldo(IFabricaConexiones fabrica);
    public string Respaldar(string rutaDestino);               // SqliteConnection.BackupDatabase
    public void Restaurar(string rutaRespaldo);
    public ResultadoIntegridad VerificarIntegridad();          // PRAGMA integrity_check + foreign_key_check
}

public sealed class BenchmarkCargaMasiva {
    public BenchmarkCargaMasiva(ConfiguracionBenchmark configuracion, string carpetaTemporal);
    public ArregloDinamico<MedicionBenchmark> Ejecutar();      // "FilaPorFila" (autocommit) vs "LoteUnaTransaccion"
}
public sealed class BenchmarkIndices {
    public BenchmarkIndices(ConfiguracionBenchmark configuracion, string carpetaTemporal);
    public ArregloDinamico<MedicionBenchmark> Ejecutar(int consultasPorCorrida = 1_000); // SKU/categoría sin vs con índice, e inserción con índices
}
```

### 5.13 `Electro.Core.Servicios`: punto de entrada único para App y Consola

```csharp
public sealed class SistemaElectro {                          // [A] raíz de composición
    public static SistemaElectro Abrir(string rutaBaseDatos, IReloj? reloj = null);  // crea la BD si falta y carga la memoria
    public static string RutaPredeterminada();                // busca datos/electro.db subiendo desde AppContext.BaseDirectory
    public string RutaBaseDatos { get; }
    public EstadoMemoria Memoria { get; }
    public ServicioVentas Ventas { get; } public ServicioDeshacer Deshacer { get; } public ServicioDespacho Despacho { get; }
    public ServicioRecepcion Recepcion { get; } public ServicioAsignacion Asignacion { get; }
    public ServicioCatalogo Catalogo { get; }                  // (F4)
    public ServicioRutas Rutas { get; }                        // (F5)
    public ServicioRespaldo Respaldo { get; }                  // (F7)
    public IReloj Reloj { get; }                               // (F2, agregado)
    public ParClaveValor<string, long>[] ConteoTablas();       // (F2, agregado) tabla → filas
    public void Recargar();                                    // reconstruye la memoria desde SQLite
}

public sealed class ServicioVentas {                          // [C]
    public ServicioVentas(IRegistroVentasDao ventas, IPedidoDao pedidos, IClienteDao clientes, IRedDao red, EstadoMemoria memoria, IReloj reloj);
    public ResultadoVenta RegistrarVenta(SolicitudVenta solicitud);
    public ResultadoOperacion CancelarPedido(long idPedido, string motivo);
    public ResultadoOperacion DevolverPedido(long idPedido, string motivo);
    public Pedido ObtenerPedido(long idPedido);
    public ArregloDinamico<ResumenPedido> Pedidos(EstadoPedido? estado);
    public ArregloDinamico<HistorialEstadoPedido> Historial(long idPedido);
    public ArregloDinamico<Cliente> Clientes();
    public ArregloDinamico<Almacen> Almacenes();
}

public sealed class ServicioDeshacer {                        // [B]
    public ServicioDeshacer(IRegistroVentasDao ventas, EstadoMemoria memoria, IReloj reloj);
    public bool PuedeDeshacer { get; }
    public OperacionRegistrada? UltimaOperacion();
    public ResultadoReversion Deshacer();                      // Revertir (transacción) → COMMIT → pop → memoria
    public IEnumerable<OperacionRegistrada> Historial();       // del tope al fondo
}

public sealed class ServicioDespacho {                        // [B]
    public ServicioDespacho(IRegistroVentasDao ventas, EstadoMemoria memoria, IReloj reloj);
    public int Pendientes { get; }
    public ResumenPedido? VerSiguiente();
    public ResumenPedido[] PendientesPorPrioridad();
    public ResultadoReasignacion ReasignarEnvio(long idEnvio, long idVehiculoNuevo);
    public ResultadoOperacion MarcarEnRuta(long idPedido);
    public ResultadoOperacion MarcarEntregado(long idPedido);
    public IEnumerable<Vehiculo> Vehiculos();
}

public enum TipoBuffer { Circular, Enlazado }
public sealed class ResultadoProcesamiento { public int Procesadas { get; init; } public int Fallidas { get; init; } public required ArregloDinamico<string> Errores { get; init; } }
public sealed class ServicioRecepcion {                       // [B]
    public ServicioRecepcion(IBufferPedidos<SolicitudVenta> buffer, ServicioVentas ventas);
    public static IBufferPedidos<SolicitudVenta> CrearBuffer(TipoBuffer tipo, int capacidad);
    public IBufferPedidos<SolicitudVenta> Buffer { get; }
    public bool Recibir(SolicitudVenta solicitud);             // false = buffer lleno (rechazada)
    public int EnEspera { get; }
    public ResultadoProcesamiento ProcesarTodo();
}

public sealed class ServicioAsignacion {                      // [A]
    public ServicioAsignacion(IFlotaDao flota, IEnvioDao envios, EstadoMemoria memoria, IReloj reloj);
    public ResultadoAsignacion AsignarSiguienteLote(int maximoPedidos = 50);  // prioridad → FFD → transacción → memoria
}

public sealed class ServicioCatalogo {                        // [C] (F4)
    public ServicioCatalogo(ICatalogoDao catalogo, IInventarioDao inventario, EstadoMemoria memoria, IReloj reloj);
    public Electrodomestico? BuscarPorSku(string sku);         // ABB en memoria
    public Paquete? BuscarGuia(string numeroGuia);             // ABB en memoria
    public ArregloDinamico<Electrodomestico> Filtrar(FiltroCatalogo filtro);   // SQLite
    public IEnumerable<Electrodomestico> RecorrerInorden(); public IEnumerable<Electrodomestico> RecorrerPreorden(); public IEnumerable<Electrodomestico> RecorrerPostorden();
    public ArregloDinamico<ExistenciaAlmacen> Existencias(string sku);
    public Electrodomestico AgregarProducto(Electrodomestico nuevo, long idAlmacenInicial, int stockInicial);
    public void CambiarPrecio(string sku, decimal nuevoPrecio);
    public void DarDeBaja(string sku);
    public int AlturaIndiceSku { get; } public int CantidadSku { get; }
    public ArregloDinamico<string> Marcas(); public ArregloDinamico<string> Categorias();
}

public sealed class ServicioRutas {                           // [A] (F5)
    public ServicioRutas(IRedDao red, IRutaDao rutas, IFlotaDao flota, IFabricaConexiones fabrica, EstadoMemoria memoria, IReloj reloj);
    public ResultadoDijkstra CalcularDesde(long idNodoOrigen, bool registrarTraza);
    public ResultadoCamino Camino(long idNodoOrigen, long idNodoDestino);
    public RutaPlanificada PlanificarRutaVehiculo(long idVehiculo);
    public HojaDeRuta HojaDeRuta(long idRuta);
    public ResultadoRecalculo CerrarTramo(long idTramo);
    public ResultadoRecalculo AbrirTramo(long idTramo);
    public ResultadoRecalculo CambiarCostoTramo(long idTramo, double nuevoCosto);
    public IEnumerable<NodoDistribucion> Nodos(); public IEnumerable<AristaVial> Tramos(); public IEnumerable<RutaPlanificada> RutasActivas();
}
```

### 5.14 Electro.App y Electro.Consola

```csharp
// Electro.App [A]
public interface IPestanaModulo { string Titulo { get; } int Orden { get; } void Inicializar(SistemaElectro sistema); void Refrescar(); }
// FormPrincipal crea un TabControl e instancia por reflexión todo UserControl que implemente IPestanaModulo.

// Electro.Consola [A]
public sealed class ContextoConsola { public required string RutaBaseDatos { get; init; } public required string CarpetaResultados { get; init; } public required string[] Argumentos { get; init; } public TextWriter Salida { get; init; } = Console.Out; public Func<string?> LeerLinea { get; init; } = Console.ReadLine; }
public interface IEscenarioConsola { string Clave { get; } string Titulo { get; } int Orden { get; } string? ArgumentoCli { get; } int Ejecutar(ContextoConsola contexto); } // código de salida
// MenuPrincipal descubre escenarios por reflexión; Program: sin argumentos → menú; con ArgumentoCli → ejecuta y sale.
```

### 5.15 Infraestructura de pruebas

```csharp
public sealed class BaseDatosTemporal : IDisposable {          // [C] tests/Infraestructura
    public static BaseDatosTemporal Crear(bool conDatosPrueba = true);
    public string Ruta { get; } public IFabricaConexiones Fabrica { get; }
    public SistemaElectro AbrirSistema(IReloj? reloj = null);
    public void Dispose();                                     // SqliteConnection.ClearAllPools() y borra el archivo
}
public static class RutasRepositorio {                         // [B]
    public static string Raiz();                               // carpeta que contiene ElectroLogistica.sln
    public static string Proyecto(string nombre);              // src/<nombre>
    public static string DllConsola();                         // salida de Electro.Consola junto a las pruebas
}
```

---

## 6. Plan de Git (resumen; el detalle va en `docs/git/Flujo_Git.md`, E8)

- Ramas: `main` (protegida, solo merges de fase con etiqueta `fase-N`), `develop` (integración) y `feature/f<N>-<perfil>-<tema>`, por ejemplo `feature/f3-B-monticulo`, `feature/f3-C-dao-ventas`, `feature/f5-A-dijkstra`.
- Cada PR va a `develop`. **A revisa** los PR de B y C; **B revisa** los de A (nadie aprueba su propio PR). El revisor solo comenta y el **autor corrige** en su rama (regla 7).
- Convención de mensajes: `tipo(ámbito): descripción`, con tipo ∈ {feat, test, docs, refactor, fix, chore}. Ejemplo: `feat(monticulo): eliminar elemento en O(log n) con posición indexada`.
- Equilibrio: cada archivo tiene un solo dueño (tabla §1). Por número de archivos de código y documentos, A ≈ 34 %, B ≈ 35 % y C ≈ 31 %.

---

## 7. Entregas restantes (M = 10)

| Entrega | Contenido | Archivos |
|---|---|---|
| **1** – Fase 1 | Viabilidad, costo/beneficio (MXN, supuestos y fuentes), requerimientos, casos de uso, reglas de integridad, matriz de riesgos, Gantt | `docs/reporte/10_…`, `11_…`, `12_…`, `docs/diagramas/casos_uso.md`, `docs/diagramas/gantt.md` |
| **2a** – Fase 2 | Solución, csproj, `.gitignore`, Comun, Contratos, Dominio (A y B), estructuras lineales y buffers, App/Consola base, pruebas de F2 (sin BD) | raíz, `src/Electro.Core/{Comun,Contratos,Dominio,Estructuras/Lineales*}`, `src/Electro.App/*` base, `src/Electro.Consola/*` base + `EscenarioBuffers.cs`, `tests/…/Electro.Tests.csproj`, `Infraestructura/RutasRepositorio.cs`, `Fase2/*` (excepto Esquema) |
| **2b** – Fase 2 | DDL, DML, `.db`, capa base de datos, escenario `--crear-bd`, prueba de esquema, diagramas de arquitectura, clases y E-R, secciones 13, 14 y 20 | `sql/DDL.sql`, `sql/DML.sql`, `datos/electro.db`, `Datos/{IFabricaConexiones,FabricaConexiones,EjecutorTransaccional,InicializadorBaseDatos,ConversorValores}.cs`, `EscenarioCrearBaseDatos.cs`, `BaseDatosTemporal.cs`, `EsquemaSqliteTests.cs`, `docs/diagramas/{arquitectura,clases,entidad_relacion}.md`, `docs/reporte/{13,14,20}_…` |
| **3a** – Fase 3 | Pila, montículo, cola de despacho, pila de deshacer, cargas, FFD + pruebas | `PilaArreglo.cs`, `MonticuloBinario.cs`, `Despacho/*`, `Asignacion/*`, `Fase3/{PilaArreglo,MonticuloBinario,ColaDespacho,PilaDeshacer,FirstFitDecreasing}Tests.cs` |
| **3b** – Fase 3 | DAO transaccional, sincronización, servicios de ventas, deshacer, despacho, recepción y asignación, pestañas Ventas y Despacho, escenarios + pruebas, sección 21 | `GeneradorFolios.cs`, `Dao/{ContratosDao,RegistrosDatos,ResultadosPersistencia,Cliente,Inventario,Pedido,Envio,Flota,Bitacora,RegistroVentas}…`, `Sincronizacion/*`, `Servicios/{SistemaElectro,ServicioVentas,ServicioDeshacer,ServicioDespacho,ServicioRecepcion,ServicioAsignacion}.cs`, `PestanaVentas.cs`, `PestanaDespacho.cs`, `EscenarioVentasDeshacer.cs`, `EscenarioAsignacionFfd.cs`, `Fase3/{RegistroVentasDao,DeshacerPersistente}Tests.cs`, `docs/reporte/21_…` |
| **4** – Fase 4 | ABB, catálogo SQLite + índices, ServicioCatalogo, benchmark ABB vs secuencial (infraestructura de benchmark), pestaña Inventario, escenarios + pruebas, sección 22 | `Estructuras/Arboles/*`, `FiltroCatalogo.cs`, `CatalogoDao.cs`, ♻`ContratosDao`, ♻`EstadoMemoria`, ♻`CargadorMemoria`, ♻`SistemaElectro`, `ServicioCatalogo.cs`, `Rendimiento/{Estadistica,ConfiguracionBenchmark,MedicionBenchmark,GeneradorDatos,CurvaTeorica,ExportadorCsv,BenchmarkBusquedaAbb}.cs`, `PestanaInventario.cs`, `EscenarioCatalogoAbb.cs`, `EscenarioBenchmarkAbb.cs`, `Fase4/*`, `docs/reporte/22_…` |
| **5a** – Fase 5 | Grafo, Dijkstra con traza, reconstrucción recursiva, hoja de ruta, vecino más cercano + pruebas | `Grafos/{GrafoLogistico,EntradaDijkstra,PasoTrazaDijkstra,ResultadoDijkstra,Dijkstra,ExportadorTrazaDijkstra,RutaPlanificada,VecinoMasCercano}.cs`, `Caminos/*`, `Fase5/{GrafoLogistico,Dijkstra,ReconstructorCamino,VecinoMasCercano}Tests.cs` |
| **5b** – Fase 5 | Ruteo dinámico, RedDao, RutaDao, ServicioRutas, pestaña Rutas, escenarios de traza y ruteo dinámico + pruebas, sección 23 (traza sobre los datos del DML) | `Grafos/{ResultadoRecalculo,RuteadorDinamico}.cs`, `RedDao.cs`, `RutaDao.cs`, ♻`ContratosDao`, ♻`RegistrosDatos`, ♻`EstadoMemoria`, ♻`CargadorMemoria`, ♻`SistemaElectro`, `ServicioRutas.cs`, `PestanaRutas.cs`, `EscenarioTrazaDijkstra.cs`, `EscenarioRuteoDinamico.cs`, `Fase5/{RuteadorDinamico,RutaDao}Tests.cs`, `docs/reporte/23_…` |
| **6a** – Fase 6 | Comparadores, QuickSort 3 vías, MergeSort, InsertionSort + pruebas | `Ordenamiento/*`, `Rendimiento/InsertionSort.cs`, `Fase6/{Comparadores,QuickSort3Vias,MergeSort,InsertionSort}Tests.cs` |
| **6b** – Fase 6 | Banco de benchmarks, carga masiva e índices, CSV, gráficas en Python, escenarios + pruebas, sección 24 | `BancoBenchmarks.cs`, ♻`GeneradorDatos`, ♻`ExportadorCsv`, `Datos/Rendimiento/*`, `EscenarioDatosAleatorios.cs`, `EscenarioBenchmarkOrdenamiento.cs`, `EscenarioBenchmarkSqlite.cs`, `scripts/*`, `resultados/.gitkeep`, `Fase6/{EstadisticaCsv,BenchmarkSqlite}Tests.cs`, `docs/reporte/24_…` |
| **7** – Fase 7 | Respaldo, transacción interrumpida, casos límite, integración, arquitectura, integridad/CHECK, recuperación, memoria, esquema final, procedimiento VS, secciones 15 y 25, clases y arquitectura finales | `ServicioRespaldo.cs`, ♻`SistemaElectro`, `EscenarioTransaccionInterrumpida.cs`, `EscenarioRespaldo.cs`, `Fase7/*`, `sql/ESQUEMA_FINAL.sql`, `docs/esquema/Esquema_SQLite.md`, `docs/memoria/Procedimiento_Diagnostico_VS.md`, ♻`docs/diagramas/clases.md`, ♻`docs/reporte/13_…`, `docs/reporte/{15,25}_…`, `capturas/.gitkeep` |
| **8** | README y flujo Git completo (ramas, orden de commits con mensajes, PR) | `README.md`, `docs/git/Flujo_Git.md` |
| **9** | Reporte APA completo: portada, resumen, conclusiones (equipo + borradores individuales), referencias, tabla de cumplimiento de rúbricas, metadata y comando `pandoc` | `docs/reporte/{metadata.yaml,00,01,30,40,50}_…` |
| **10** | Guion del video con tiempos, reparto, qué mostrar en pantalla y lista de comprobación | `docs/video/Guion_Video.md` |

### Registro de cambios al contrato

Ningún nombre existente se renombró; los cambios son adelantos o adiciones.

| Entrega | Cambio | Archivos afectados |
|---|---|---|
| 2a | Microsoft.Data.Sqlite 10.0.15 (la 10.0.0 arrastra SQLitePCLRaw 2.1.11, con vulnerabilidad alta) | `Electro.Core.csproj`, `Electro.Tests.csproj` |
| 2a/2b | `Electro.App` pasa a la 2b; `SistemaElectro` mínimo y `TipoOperacion` se adelantan a la Fase 2; `SistemaElectro` agrega `Reloj` y `ConteoTablas()` | `SistemaElectro.cs`, `TipoOperacion.cs`, `Electro.App/*` |
| 3b | `ServicioAsignacion` recibe también `IPedidoDao` y agrupa el lote por almacén de origen; `FirstFitDecreasing.Asignar` agrega una sobrecarga con `idAlmacenBase` | `ServicioAsignacion.cs`, `FirstFitDecreasing.cs`, `ResultadoAsignacion.cs`, `SistemaElectro.cs` |
| 3b | Los resultados de escritura agregan `Operacion` (fila de bitácora creada) y `ResultadoReversion.Descripcion`; `ResultadoOperacion` agrega `EstadoNuevo` | `ResultadosPersistencia.cs` |
| 3b | Contratos DAO con sobrecargas `(conexión, transacción)` adicionales y `IFlotaDao.ObtenerVehiculo`/`CargaActual`; `RedDao` aparece en la Fase 3 (nodos y almacenes) y se completa en la 5 | `ContratosDao.cs`, `RedDao.cs` |
| 3b | Archivo nuevo `Datos/Dao/AyudanteSql.cs` (interno, consultas parametrizadas) | — |
| 3b | `EstadoMemoria` concentra la aplicación de resultados en memoria (`Aplicar…`), así que los servicios no cambian cuando la Fase 4 agrega los ABB | `EstadoMemoria.cs` |
| 3b | `ContextoConsola.CrearCopiaDeTrabajo`: los escenarios que escriben trabajan sobre una copia y no alteran `datos/electro.db` | `ContextoConsola.cs` |
| 4 | `ICatalogoDao` agrega `RegistrarProducto`, `RegistrarCambioPrecio` y `RegistrarBaja` (transacción completa); la infraestructura de benchmarks (`Estadistica`, `GeneradorDatos`, `ExportadorCsv`…) se entrega completa en la Fase 4 y no se modifica en la 6; `GeneradorDatos.GenerarClavesOrdenadas` y `CurvaTeorica.Lineal` son nuevos | `ContratosDao.cs`, `CatalogoDao.cs`, `Rendimiento/*` |
| 5 | `ResultadoDijkstra.CaminoUsaTramo` y `RuteadorDinamico.RefrescarArboles`/`Buscar`/`BuscarPorVehiculo`: criterio exacto de rutas afectadas | `ResultadoDijkstra.cs`, `RuteadorDinamico.cs`, `ServicioRutas.cs` |
| 7 | `SistemaElectro.RestaurarDesde`; `Electro.Consola` referencia `Microsoft.Data.Sqlite` (escenario de transacción interrumpida); `tests/…/Fase5/GrafoPrueba.cs` (grafo de referencia de las pruebas) | `SistemaElectro.cs`, `Electro.Consola.csproj` |

Las fases de código (2 a 7) se compilan y se prueban con `dotnet build` y `dotnet test` en este equipo (tiene el SDK de .NET 10) antes de darlas por cerradas. Los benchmarks no se ejecutan con fines de reporte: sus resultados quedan como `[PEGAR RESULTADO: …]`.
