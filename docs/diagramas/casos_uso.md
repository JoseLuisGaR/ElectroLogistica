<!-- Ruta: docs/diagramas/casos_uso.md · Responsable: B (Bárbara Sofía Pérez Romo) · Fase 1 -->

# Diagrama de casos de uso – ElectroLogística

Mermaid no tiene un diagrama UML de casos de uso nativo. Se representa con un diagrama de flujo que sigue la notación UML: los actores van fuera del límite del sistema, los casos de uso son óvalos dentro del recuadro `ElectroLogística`, y las relaciones `«include»` y `«extend»` van punteadas y etiquetadas. Los identificadores coinciden con la Tabla 16 del reporte.

```mermaid
flowchart LR
    Vendedor(["👤 Vendedor"])
    Almacenista(["👤 Almacenista"])
    Despachador(["👤 Despachador"])
    Transportista(["👤 Transportista"])
    Gerente(["👤 Gerente"])
    Probador(["👤 Probador QA"])
    SQLite[("SQLite<br/>(sistema externo)")]

    subgraph SIS["ElectroLogística"]
        direction TB
        subgraph VENTA["Venta"]
            CU01(["CU-01 Registrar venta"])
            CU02(["CU-02 Cancelar pedido"])
            CU03(["CU-03 Deshacer última operación"])
            CU04(["CU-04 Recibir y procesar solicitudes"])
            VALSTOCK(["Validar stock"])
        end
        subgraph ALMACEN["Almacén"]
            CU05(["CU-05 Buscar por SKU y filtrar catálogo"])
            CU06(["CU-06 Administrar producto"])
            CU07(["CU-07 Registrar devolución"])
        end
        subgraph DESPACHO["Despacho"]
            CU08(["CU-08 Despachar lote priorizado (FFD)"])
            CU09(["CU-09 Reasignar envío"])
        end
        subgraph PAQUETERIA["Paquetería"]
            CU10(["CU-10 Rastrear paquete por guía"])
            CU11(["CU-11 Planificar ruta y hoja de ruta"])
            CU12(["CU-12 Cerrar tramo / cambiar costo"])
            CU13(["CU-13 Marcar en ruta y entregado"])
            CALC(["Calcular camino mínimo (Dijkstra)"])
        end
        subgraph ADMIN["Administración y pruebas"]
            CU14(["CU-14 Respaldar y verificar base"])
            CU15(["CU-15 Ejecutar escenarios y benchmarks"])
        end
    end

    Vendedor --- CU01
    Vendedor --- CU02
    Vendedor --- CU03
    Vendedor --- CU04
    Vendedor --- CU10
    Almacenista --- CU05
    Almacenista --- CU06
    Almacenista --- CU07
    Despachador --- CU03
    Despachador --- CU08
    Despachador --- CU09
    Despachador --- CU10
    Despachador --- CU11
    Despachador --- CU12
    Transportista --- CU11
    Transportista --- CU13
    Gerente --- CU14
    Probador --- CU15

    CU01 -. "«include»" .-> VALSTOCK
    CU04 -. "«include»" .-> CU01
    CU03 -. "«extend»<br/>(revierte)" .-> CU01
    CU03 -. "«extend»<br/>(revierte)" .-> CU02
    CU03 -. "«extend»<br/>(revierte)" .-> CU07
    CU03 -. "«extend»<br/>(revierte)" .-> CU09
    CU11 -. "«include»" .-> CALC
    CU12 -. "«include»" .-> CU11
    CU07 -. "«include»" .-> CU10

    SIS --- SQLite
```

## Notas de lectura

- **CU-01 «include» Validar stock:** toda venta valida existencias dentro de la misma transacción (RI-01).
- **CU-04 «include» CU-01:** procesar el buffer de recepción registra cada solicitud como venta.
- **CU-03 «extend»:** deshacer amplía venta, cancelación, devolución y reasignación con su operación inversa, persistida en `bitacora_operaciones`.
- **CU-11 «include» Calcular camino mínimo:** la planificación encadena Dijkstra con la heurística del vecino más cercano.
- **CU-12 «include» CU-11:** el ruteo dinámico vuelve a planificar únicamente las rutas afectadas.
- **CU-07 «include» CU-10:** la devolución localiza el pedido por número de guía en el ABB.
