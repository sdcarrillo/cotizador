# Dataset anonimizado de precios de proveedores gastronómicos

Exportado el 2026-10-25 desde la base `prod` del cotizador (commit
`20bbd88`) con `agent/scripts/exportar_dataset.py`
(`cotizador/dataset_anonimo.py`), que abre la base en solo lectura. Correr el
script otra vez sobre la misma base produce exactamente los mismos archivos.

## Procedencia y licencia

- Corte: 2026-10-20 (declarado con `--corte`).
- Período: del 2026-08-19 al 2026-10-20 (la primera y la última observación).
- Base de origen: sha256 `499695f63828e661380ad55d06b0fd7a2c882676d1fa6dfdbb4f181e7915a1e9` (71368704 bytes).
- Licencia: CC BY-NC 4.0 (Creative Commons Atribución-NoComercial 4.0 Internacional, https://creativecommons.org/licenses/by-nc/4.0/deed.es).

## Conteos

| archivo | filas |
|---|---|
| proveedores.csv | 179 |
| observaciones.csv | 72702 |
| productos.csv | 32204 |

Listas de precios (extracciones vigentes con al menos una observación): 322.

Celdas de texto en las que el redactor tapó algo, en total y por regla (una
celda cuenta en cada regla que tapó algo en ella):

| archivo.columna | celdas | correo | url | usuario | teléfono | nombre | núcleo |
|---|---|---|---|---|---|---|---|
| observaciones.product_name | 774 | 0 | 1 | 0 | 0 | 349 | 424 |
| observaciones.product_name_raw | 612 | 0 | 1 | 0 | 0 | 305 | 306 |
| productos.name | 412 | 0 | 1 | 0 | 0 | 162 | 249 |

## Reglas de anonimización

1. Ningún proveedor sale con su nombre. Cada proveedor con al menos una
   observación recibe un seudónimo (`P001`, `P002`...) asignado por orden de su
   id interno, estable entre corridas sobre la misma base. El mapa seudónimo ->
   id interno se escribe FUERA de esta carpeta, como archivo hermano
   (`mapa-seudonimos-<carpeta>.json`), y no se comparte con el dataset.
2. Los productos provisionales (acuñados por el sistema a partir del texto de un
   proveedor) salen con un id opaco, `P-000001`, `P-000002`..., numerado por la
   primera observación que lo nombra. Su sku interno deja recalcular el nombre
   del que salió y, con él, la palabra que el redactor tapó. La tabla id opaco
   -> sku interno se escribe FUERA de esta carpeta, junto al mapa
   (`skus-provisionales-<carpeta>.json`). Los productos de la semilla SEPA, que
   es un catálogo público, salen con su código de barras como sku.
3. No salen teléfonos, correos, direcciones, URLs, nombres comerciales, nombres
   de perfil, condiciones de pago ni notas.
   La evidencia literal NO se incluye (se pide aparte con
   `--con-evidencia`).
4. Todo texto libre que sí sale (rubro, zona, product_name_raw, presentation,
   unit, condicion, product_name, name y, si está, evidence) pasa por un
   redactor que reemplaza por `[redactado]`:
   - correos, URLs (con `http`, con `www` o dominios `.com`, `.org`, `.gob`,
     `.edu`, `.ar`, `.me`, `.io`) y @usuarios;
   - teléfonos: los números que la base conoce (las identidades de teléfono y
     de WhatsApp de los proveedores, los del padrón y los propios), escritos
     con o sin código de país y de área y con cualquier separador, y las formas
     de un teléfono argentino aunque la base no lo conozca: con `+54`, con
     `54 9` separado, con un 0 y su código de área (`(011) 4444-3333`,
     `0221 456-7890`), 11 seguido de 8 dígitos (`11 5555-4444`, `1155554444`),
     `15-5555-4444`, y 8 dígitos después de "tel.", "cel." o "WhatsApp". Un
     código de barras, un precio con separador de miles, un decimal o dos
     precios seguidos no se tapan;
   - los nombres completos con los que la base conoce a los proveedores (el
     nombre comercial, el de perfil de WhatsApp, cómo se presentó en una lista
     y el del padrón de outreach), sin distinguir mayúsculas ni tildes: en
     `evidence`, los de todos los proveedores que la base conoce, estén o no
     en el dataset; en las demás columnas, solo los de los proveedores que
     están en el dataset. Los nombres de 5 caracteres o más se tapan donde
     aparezcan, aun dentro de otra palabra; los de 3 o 4, solo como palabra
     entera. No se buscan los de menos de 3 letras o dígitos ni los que son
     una sola palabra común (un perfil de WhatsApp que dice "Ventas");
   - el núcleo del nombre de cada proveedor en sus propias filas: el nombre
     sin las palabras genéricas (distribuidora, mayorista, frigorífico, los
     rubros, S.A., SRL...) y una lista corta de formas conocidas, como palabra
     entera, en las columnas de texto de sus observaciones. "Distribuidora El
     Ejemplo" tapa "El Ejemplo" en las filas de ese proveedor. Una palabra
     común sola (queso, pollo) no se tapa. El nombre de catálogo de un
     producto, provisional o SEPA, es uno solo, igual en todas las filas y en
     productos.csv, y se tapa con el núcleo de todos los proveedores que lo
     cotizan.
     En las filas de los otros proveedores el núcleo no se tapa: ahí es la
     marca de un producto que venden, no la identidad de quien vende.
5. Las fechas salen sin hora.

## Archivos y columnas

### proveedores.csv

- `seudonimo`: `P001`, `P002`...
- `rubro`: rubro del proveedor según la base.
- `zona`: zona de reparto declarada.
- `n_observaciones`: observaciones de precio del proveedor en este dataset.
- `n_listas`: extracciones (listas) con al menos una observación.
- `primera_observacion`, `ultima_observacion`: fechas (AAAA-MM-DD).
- `canal_entrada`: `whatsapp` o `correo`, según por dónde llegó su PRIMER
  mensaje entrante (WhatsApp Cloud API o el bridge histórico: `whatsapp`;
  correo: `correo`; en mensajes de prueba inyectados decide el tipo de
  identidad por la que llegaron). Vacío si nunca escribió.

### observaciones.csv

- `obs_id`, `extraction_id`, `turn_id`: ids internos de la base. No identifican
  a nadie fuera de ella; sirven para volver a la fila original.
- `seudonimo`: el proveedor (ver proveedores.csv).
- `fecha`: día al que corresponde el precio (el cierre del turno).
- `product_name_raw`: el producto tal como lo escribió el proveedor (redactado).
- `presentation`, `unit`, `quantity_per_presentation`: presentación, unidad a
  la que refiere el precio y cantidad por presentación.
- `price`, `currency`: precio (vacío = sin precio) y moneda.
- `available`: 1 disponible, 0 sin stock (el sin stock es una observación, no
  una omisión).
- `confidence`: confianza que el agente se asignó para ese renglón.
- `product_sku`, `product_name`: producto de catálogo según el método de apareo
  vigente (`bm25+llm v2-k96`): el código de barras de un producto SEPA o el
  id opaco de un provisional (`P-000001`...), y su nombre de catálogo redactado;
  vacíos si ese método no lo apareó.
- `origen`: tipo del mensaje de donde salió el precio (`text`, `document`,
  `image`, `audio`...).
- `tipo_precio`: `lista` (el precio de siempre), `oferta` (un precio de
  oferta o promo, en un renglón aparte del de lista) o `escala` (el precio de
  una escala de cantidad mayor a la mínima, un renglón por escala). Para
  comparar proveedores entre sí se usa `'lista'`, salvo que se quiera comparar
  ofertas o escalas.
- `condicion`: la condición literal del precio ("válida hasta el 9/10",
  "comprando 10 cajas"), redactada; vacía si no tiene.
- `iva`: `incluido` o `sin_iva` cuando la fuente lo dice; vacío si no lo dice.

### productos.csv

- `sku`, `name`, `origin`: los productos de catálogo referenciados por alguna
  observación. `origin` es `seed_sepa` (catálogo SEPA, `sku` es su código de
  barras) o `provisional` (acuñado por el sistema a partir del texto de un
  proveedor, `sku` es su id opaco). El nombre pasa por el redactor y por el
  núcleo de los proveedores que lo cotizan.

## Criterios

- Observaciones incluidas: las de la extracción VIGENTE de cada turno (la última;
  una re-extracción del mismo turno reemplaza a la anterior), la misma regla de
  la vista `v_observaciones`. Proveedores incluidos: los que tienen al menos una
  de esas observaciones.
- Apareo vigente: el veredicto del método activo de `match_methods`
  (`bm25+llm v2-k96`). Los veredictos de métodos anteriores no salen.

## Límites conocidos

- El núcleo se tapa en las filas del proveedor y no en las de los demás.
  Cuando el nombre de un proveedor es también una marca que venden otros,
  la marca queda a la vista en las filas de esos otros, y comparar el mismo
  producto en las filas de los dos puede sugerir qué tapó el redactor en
  las del proveedor. Taparlo en todas las filas cierra esa vía y borra la
  marca de los productos de los demás. Por ahora se eligió conservar las marcas.
- Los productos de la semilla SEPA conservan su código de barras: el catálogo
  público dice su nombre, con la marca, aunque aquí esté tapada porque coincide
  con el nombre de quien lo cotiza.
- Una palabra común sola del nombre de un proveedor ("pollo" en el de una
  "Distribuidora Pollo") no se tapa, y un nombre que es una sola palabra común
  tampoco.
- En las columnas de producto (`product_name_raw`, `product_name`, `name` del
  catálogo) solo se tapan los nombres de los proveedores que ESTÁN en el
  dataset: un proveedor que la base conoce pero no mandó precios y aparece en la
  lista de otro es una marca ("Paladini"), no un sujeto. En la evidencia se
  tapa todo nombre conocido, porque ahí puede ser un tercero recomendado.
- Solo se tapan los nombres tal como están escritos en la base, su núcleo y
  una lista corta de formas conocidas: una abreviatura o un error de tipeo
  distintos pueden sobrevivir en el texto libre, sobre todo en la evidencia.
  Por eso la evidencia no se exporta por defecto, y con `--con-evidencia`
  conviene revisarla antes de publicar.
- Un teléfono que la base no conoce y que está escrito de una forma que no es
  la argentina habitual puede sobrevivir; al revés, un código de 10 dígitos que
  empieza con 11 se tapa como si fuera un teléfono.

## Versión pública

Esta carpeta es la exportación del corte con dos cambios, hechos el 2026-11-12 después de una auditoría
independiente y antes de publicarla:

- La columna `zona` de `proveedores.csv` lleva solo una zona general (`capital`, `norte`, `sur`, `oeste`) o queda
  vacía. La exportación traía, para algunos proveedores, la zona de reparto en texto libre. Cambian
  28 filas.
- En las filas de 2 proveedores se tapó además una palabra de su nombre que usaban como
  marca propia: 5 celdas de sus renglones y 5 nombres de catálogo, que se repiten en
  24 filas de `observaciones.csv`.

Los conteos de las tablas de arriba son los de la exportación y no incluyen estos cambios. Las filas, las
columnas y los seudónimos son los mismos.
