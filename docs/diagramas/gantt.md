<!-- Ruta: docs/diagramas/gantt.md · Responsable: C (José Luis García Robinson) · Fase 1 -->

# Cronograma de Gantt – ElectroLogística

Periodo: **25/10/2026 – 25/11/2026**. Siete fases con sus dependencias (`after`) e hitos H1–H7. La tabla de fechas y la ruta crítica están en la sección «Cronograma» del reporte (`docs/reporte/12_integridad_riesgos_gantt.md`). Este diagrama es la fuente; el reporte lo replica.

```mermaid
gantt
    title ElectroLogística · Cronograma 25/10/2026 – 25/11/2026
    dateFormat  YYYY-MM-DD
    axisFormat  %d/%m
    todayMarker off

    section Fase 1 · Viabilidad, requisitos y gestión
    A · Viabilidad y costo/beneficio               :f1a, 2026-10-25, 4d
    B · Requerimientos y casos de uso              :f1b, 2026-10-25, 4d
    C · Integridad, riesgos y Gantt                :f1c, 2026-10-25, 4d
    H1 · Plan aprobado                             :milestone, h1, after f1a f1b f1c, 0d

    section Fase 2 · Arquitectura, BD y TDA
    A · UML, interfaces y TDA logísticos           :f2a, after h1, 5d
    B · TDA de ventas y buffers                    :f2b, after h1, 5d
    C · E-R, DDL, DML y electro.db                 :f2c, after h1, 5d
    H2 · Solución y esquema funcionales            :milestone, h2, after f2a f2b f2c, 0d

    section Fase 3 · Despacho, ventas y auditoría
    B · Montículo binario y cola de despacho       :f3b1, after h2, 3d
    B · Pila de deshacer persistente               :f3b2, after f3b1, 3d
    C · DAO transaccional de ventas y reversiones  :f3c, after h2, 6d
    A · First-Fit Decreasing                       :f3a, after f3b1, 3d
    H3 · Ventas y despacho integrados              :milestone, h3, after f3a f3b2 f3c, 0d

    section Fase 4 · Catálogo e indexación con ABB
    B · ABB de SKU y de guías                      :f4b, after h3, 2d
    C · Índices, consultas y carga al ABB          :f4c, after h3, 4d
    A · Benchmark ABB vs. secuencial               :f4a, after f4b, 2d
    H4 · Catálogo indexado                         :milestone, h4, after f4a f4b f4c, 0d

    section Fase 5 · Red logística y ruteo
    A · Grafo y Dijkstra con traza                 :f5a1, after h4, 3d
    B · Reconstrucción recursiva y hoja de ruta    :f5b, after h4, 2d
    C · Tablas y DAO de ruteo                      :f5c, after h4, 3d
    A · Vecino más cercano y ruteo dinámico        :f5a2, after f5a1 f5b f5c, 2d
    H5 · Ruteo dinámico operativo                  :milestone, h5, after f5a2, 0d

    section Fase 6 · Ordenamiento y rendimiento
    B · QuickSort 3 vías y comparadores            :f6b, after f5b, 3d
    C · MergeSort                                  :f6c1, after f5c, 2d
    C · Benchmark de carga masiva e índices        :f6c2, after f6c1, 2d
    A · Banco de benchmarks y gráficas             :f6a, after f5a2 f6b f6c1, 2d
    H6 · Reporte de rendimiento                    :milestone, h6, after f6a f6c2, 0d

    section Fase 7 · Integración y cierre
    B · Casos límite e integración                 :f7b, after h5 h6, 3d
    C · Integridad, recuperación y respaldo        :f7c, after h5 h6, 3d
    A · Revisión SOLID, memoria y arquitectura     :f7a, after h5 h6, 3d
    H7 · Repositorio listo                         :milestone, h7, after f7a f7b f7c, 0d

    section Entrega
    Todos · README, reporte APA y rúbricas         :c1, after h7, 2d
    Todos · Ensayo y grabación del video           :c2, 2026-11-24, 1d
    Todos · Edición, publicación y entrega         :c3, 2026-11-25, 1d
    Entrega final                                  :milestone, entrega, 2026-11-25, 0d
```

## Carga por integrante (horas estimadas, Tabla 4 del reporte)

| Integrante | F1 | F2 | F3 | F4 | F5 | F6 | F7 | Total |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| A · Stephanny Elizabeth Torres Romo | 16 | 18 | 14 | 12 | 26 | 18 | 16 | 120 |
| B · Bárbara Sofía Pérez Romo | 16 | 18 | 22 | 20 | 10 | 12 | 18 | 116 |
| C · José Luis García Robinson | 16 | 20 | 22 | 14 | 12 | 16 | 18 | 118 |
