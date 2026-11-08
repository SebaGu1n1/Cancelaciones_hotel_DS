# Predicción de Cancelación de Reservas Hoteleras

## 1. Planteo del Problema
El objetivo de este proyecto es construir un modelo de clasificación binaria supervisada para predecir la probabilidad de que una reserva sea cancelada antes de la fecha de llegada. Esto permite al equipo pasar a una gestión dinámica de sobreventa, retención y exigencia de garantías focalizadas.

## 2. Impacto de la Predicción
* Minimizar el costo de oportunidad de habitaciones vacías sin desbordar la capacidad.
* Calibración precisa de umbrales de sobreventa y ajuste de tarifas dinámicas.
* Conocer la ocupación real para coordinar turnos de recepción, limpieza y compras de Alimentos y Bebidas.
* Detectar qué canales de comercialización y tipos de depósito presentan mayor riesgo de abandono para renegociar comisiones y condiciones de reserva por canal.

## 3. Puntos de Dolor
1. **Pérdida directa de ingresos por inventario congelado:** Reservas que bloquean cuartos durante meses y cancelan a días del check-in.
2. **Costos y fricción por sobreventa no controlada:** Derivación forzada de huéspedes a hoteles competidores ante arribos imprevistos.
3. **Ineficiencia en políticas homogéneas:** Exigir depósitos no reembolsables a todos los clientes reduce drásticamente la tasa de conversión general.

## 4. Hipótesis de Negocio Iniciales
* **H1 (Anticipación / Lead Time):** A mayor tiempo de anticipación entre la reserva y el check-in, mayor es la probabilidad de cancelación debido a la volatilidad en la planificación del huésped.
* **H2 (Compromiso Financiero / Depósito):** Las reservas bajo la modalidad sin depósito presentan una tasa de cancelación significativamente superior frente a reservas con depósito no reembolsable.
* **H3 (Historial del Cliente / Fidelización):** Los huéspedes que ya se hospedaron anteriormente o que no registran cancelaciones pasadas presentan una probabilidad de cancelación sensiblemente menor que clientes esporádicos captados por intermediarios.
