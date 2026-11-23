# Programar Cajeros — notas para Claude

App web (PWA) para **armar** la programación mensual de turnos de los cajeros. Todo está en `index.html`
(HTML + CSS + JS, sin build). Antes estaba en la carpeta `programar/` del repositorio `Cuadraditos-VP/CAJEROS`;
a pedido de Guillermo se pasó a este repositorio propio (no mezclar repositorios). En CAJEROS quedó solo una
página que redirige acá.

**Nunca subir datos de empleados** (el repositorio es público): la lista de gente se trae de Firebase.

## Cómo trabaja el dueño (Guillermo, admin)
- Habla en español rioplatense y no es programador: respuestas cortas y claras, sin jerga.
- **Publicar** = commit en la rama de trabajo → PR → fusionar a `main` (GitHub Pages sirve `main`). Confirmar con
  Guillermo antes de fusionar (suele alcanzar con "dale").
- **Cambios visuales**: mostrarle antes una captura (Playwright, con gente inventada) y esperar su OK.
- **Reglas de Firebase nuevas**: pedirle las actuales del proyecto de cajeros y devolvérselas completas con el bloque nuevo.
- Prefiere que salga bien de una: medir y probar en pantallas grandes y chicas (1920×1080 y 1536×730) antes de mostrar.

## Cómo es la app
App aparte para **armar** la programación del mes (copia adaptada de `Cuadraditos-VP/programar-jefes`; a pedido de Guillermo).
Se sirve en `cuadraditos-vp.github.io/<este repositorio>/` (`URL_APP` sale de la dirección). Usa el mismo Firebase que cajeros pero con app de Firebase con nombre propio
(`initializeApp(cfg,"programar")`, para no pisar la sesión de la app de cajeros) y colecciones `progr_*`
(`REGLAS-FIREBASE.md`). Claves de `localStorage` con prefijo `pcaj_` (mismo dominio que programar-jefes).
Sin nombres en el HTML: la lista (`m.lista`: leg, nom, g, o) se trae de `turnos_cajeros` (solo el admin).
Colores de fábrica = los de cajeros (y los publicados en la colección `cajeros`). Reglas del día en `cupos(d)` (sacadas
de oct/nov 2026): MS 7; MD 2 VI/SA/DO/feriado; T3 5 SA/DO, 3 resto; víspera (VI, SA, antes de feriado): TE 9, E 7, X 2;
JU: E 3, E1 3, X 2; resto E 4; N y T el resto. Francos: rueda por grupo, 2 cada 8 días (`ruedaGrupos`).
Diciembre (todos los años): 24 y 31 se consideran franco: se marcan solos como evento especial (`DIC_FRANCOS`, en `loadMonth`) y `francosMes` da 2 menos (5); si los francos no alcanzan para la rueda, primero los pares SA-DO/DO-LU completos, después un día de cada otro par.
Criterios de Programar acordados con Guillermo: los pares de francos SA-DO y DO-LU de la rueda no se parten (`parFinde`; a otros
pares se les puede sacar un día); rotación entre francos de más tarde a más temprano, con las noches (N, E1, E, X) como un solo
nivel y los de día también (`ROT`: N E N sí, T TE MS sí, T N no); de noche a día mejor de E a T (`P7`); después del franco lo más tarde posible, cualquier noche da lo mismo (`LATEF`); horas extra del período parejas (`EQUILIBRAR_HE`,
`PG_W.HE`, proporcional a los días sin LAO/licencia); evitar noche → una sola tarde → mañana (N T MS: `nocheTardeManana`, `P8`).
Resumen (a pedido de Guillermo): solo F del mes y HE = horas extra del 26 del mes anterior al 25 de este (`horasExtra`,
`OT_TABLE`/`tipoDia` iguales a los de cajeros); del mes anterior se toma su programación inicial (`m.ini`, que trae
`mesDeCajeros` desde `inicial` de turnos_cajeros; si no hay, `m.t`).
Totales fijos abajo de la pantalla (`fijarTotales`, sticky bottom por fila), como tablero aparte: separador del color
del fondo de pantalla (`tothdr`, a pedido de Guillermo en vez de la franja oscura con los días), color del turno solo en la etiqueta, números en negrita sobre blanco/gris. Totales a medida (a pedido de Guillermo): la grilla llega hasta abajo y la tabla de totales se agranda para empezar justo
donde termina el segundo grupo visible (`ajustarTot`: filas `--trh`, separador `--sep` ≥ 24px igual al espacio entre grupos;
se recalcula al cambiar el grupo de arriba). El alto de renglones es uno solo por mes (`altoDosGrupos`: si no entran dos
grupos + totales, `--hh` 32px y `--rh` hasta 16px) para que nada se corra. Sin totales: toda la pantalla, sin franja.
Sin video/imagen de inicio (a pedido de Guillermo).
Ícono propio (almanaque con casilleros de colores, franja bordó, fondo azul: `icon-*.png`) y manifest con `id`
propio, para no confundirla con la app de cajeros (que tiene el chaleco).
En pantalla la fila de días de arriba no queda fija (`#gridCard .grid thead th{top:auto}`: confundía, los demás grupos
parecían el grupo 1) y la ruedita salta de grupo en grupo (listener `wheel` sobre `#gridCard`; shift/ctrl/horizontal normal).
Condicionales de grilla y totales prendidos (como en programar-jefes) con los avisos de los criterios nuevos (N T MS, SA-DO/DO-LU partido);
`CONDICIONALES = true`; el franco en rojo del resumen queda.
No es gemela de progjefes ni de la app de cajeros: los cambios de acá no van a otros repositorios.


## Probar
No hay pruebas automáticas. Se prueba con Playwright sirviendo la carpeta, bloqueando Firebase (`gstatic`/`googleapis`)
y cargando en `localStorage` `pcaj_ok = "admin"` y `pcaj_meses_v1` con meses de gente inventada; después Programar,
mirar que no haya errores de JavaScript y medir que el separador de los totales coincida con el fin del segundo grupo.
