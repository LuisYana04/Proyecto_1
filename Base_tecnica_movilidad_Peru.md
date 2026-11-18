# Planificador de transporte del Perú
## Alcance inicial, arquitectura y auditoría de fuentes

Revisión del 18 de noviembre de 2026.

### Alcance de la primera versión

| Apartado | Entrada | Resultado |
|---|---|---|
| En mi ciudad | Dos puntos de una ciudad | Caminata, línea, sentido, paradero de subida y de bajada, transbordos. |
| Entre ciudades | Dos ciudades y una fecha | Servicios disponibles en el catálogo, terminales, duración publicada y enlace al operador. |
| Puerta a puerta | Dos direcciones, referencias o pines | Tramos urbanos y de larga distancia en un itinerario con cada conexión identificada. Si falta un tramo, se indica cuál y se conserva la parte encontrada. |

La primera validación de punta a punta es Mall Aventura de Paucarpata → Plaza de Armas de Lima, pasando por Terrapuerto Arequipa y Terminal Javier Prado, La Victoria. La plaza es el destino de prueba elegido para que sea reproducible. El primer modo interprovincial es bus. El modelo de datos admite otros modos cuando exista una fuente que permita incorporarlos.

### Proyectos de referencia

| Proyecto | Implementación observada | Aplicación al proyecto |
|---|---|---|
| Arequipa Bus | Flutter/Dart con Trufi Core. Docker Compose despliega OpenTripPlanner, PostgreSQL y un actualizador ASP.NET Core. Generador GTFS en JavaScript/Node.js. | Usar la separación entre interfaz, motor y generación de datos. |
| MiRuta Trujillo | Flutter/Dart con Trufi Core. Servidor con OpenTripPlanner, YARP, Photon, tileserver-gl y proxy GTFS-RT. Generador GTFS desde OSM en TypeScript. | Referencia para incorporar nuevas ciudades mediante fuentes versionadas. |
| Mappe y Rutas AQP | Funciones públicas verificadas. La arquitectura interna queda por confirmar. | Comparar presentación de sentido, letrero, paraderos y referencias. |
| Rome2Rio | Itinerarios multimodales y enlaces entre poblaciones. Arquitectura interna fuera de lo verificado. | Comparar la claridad de las conexiones y el detalle del primer y último tramo. |

[Arequipa Bus, revisión inspeccionada](https://github.com/trufi-association/trufi-arequipa/tree/9b60aa5cc46e6b8f2dad63079770279c6cb1feb7) · [Trufi Core](https://github.com/trufi-association/trufi-core) · [MiRuta Trujillo, revisión inspeccionada](https://github.com/leoguti/Trujillo-app/tree/4eefdd1b7d6d763d1b01302dc222f478ade5f53b) · [Mappe](https://mappe.space/) · [Rutas AQP](https://rutasaqp.com/) · [Rome2Rio](https://www.rome2rio.com/es/)

### Arquitectura elegida

- **React, TypeScript y Vite** para una web adaptable. **MapLibre GL JS** para dibujar recorridos y paraderos. La fuente de teselas y el geocodificador quedan por elegir. El prototipo se puede probar con pines y lugares predefinidos.
- **Node.js y TypeScript** para la API de producto y el importador. Un backend modular comparte el buscador y las respuestas de los tres apartados. El contrato de un tramo incluye modo, puntos de subida/bajada, fuente, fecha de verificación y calidad del dato temporal.
- **OpenTripPlanner 2 (Java)** como motor de planificación urbana y caminata, alimentado con GTFS y OpenStreetMap. La API de producto consulta su interfaz GraphQL. La primera prueba técnica debe confirmar que el motor importa el material y devuelve pasos válidos.
- **PostgreSQL con PostGIS** para terminales, catálogo interprovincial, geometrías propias, referencias locales y registro de fuentes. El catálogo identifica terminales por coordenadas e identidad, también cuando una ciudad tiene varios.
- **Docker Compose** para ejecutar API, base y motor. La API construye Puerta a puerta con los tramos urbanos disponibles y el servicio interprovincial. Considera el margen de embarque, el cambio de fecha en viajes nocturnos y distingue un enlace confirmado de uno orientativo.

[OpenTripPlanner 2](https://docs.opentripplanner.org/) · [MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/) · [Estándar GTFS](https://gtfs.org/)

### Fuentes inspeccionadas

| Fuente | Evidencia | Uso inicial y pendientes |
|---|---|---|
| Arequipa, Trufi | Se descargó el ZIP del commit fijado. Versión declarada 26/10/2026, 227 registros de rutas, 10 239 de paradas y 277 de viajes. Incluye trazados, frecuencias, tarifas y atribuciones. | Primera importación urbana. La cantidad cuenta registros del archivo, incluidas variantes. Verificar en campo el sentido, el letrero, el punto de subida y la vigencia de frecuencias. El ZIP atribuye a OSM, Trufi y Municipalidad. Confirmar condiciones de redistribución de cualquier dato añadido. |
| Trujillo, MiRuta | Se descargó GTFS/out/trujillo.gtfs.zip del commit fijado. Versión 19/03/2026, 86 registros de rutas, 1 485 de paradas y 201 de viajes. Incluye trazados, frecuencias y tarifas. feed_info declara fecha final 21/12/2026, mientras calendar usa 2000–2100. | Segunda importación urbana. Registrar 21/12/2026 como fecha que exige revisión y verificar con el publicador cuál es la actualización. Datos derivados de OSM. No se observó un archivo LICENSE declarado en el árbol inspeccionado de MiRuta. Revisar condiciones del conjunto antes de republicarlo. |
| Trujillo, OMUS archivado | ZIP de 12/11/2024 con 102 registros de rutas y 4 067 de paradas. El metadato feed_publisher_name dice Arequipa Bus y 13 frecuencias declaran intervalos de todo el día. | Referencia histórica. Usar MiRuta para la prueba. La licencia GPL-3.0 del repositorio OMUS no confirma por sí sola las condiciones del GTFS de MiRuta. |
| Lima y Callao, ATU | Portal abierto y geovisor. El geovisor indica actualización de servicios concesionados a enero de 2026 y muestra opciones GeoJSON, KML y SHP. Plaza Cívica confirma que estructuró un GTFS que adoptó Google Maps. | Fuente oficial para contrastar. Sigue pendiente un archivo GTFS público descargable con cobertura utilizable, paradas ordenadas y servicio para el recorrido de prueba, o una validación manual de ese tramo con fuentes oficiales. |
| OpenStreetMap / Geofabrik | Extracto de Perú en formato PBF localizado. OSM publica sus datos bajo ODbL. | Base para red peatonal y geográfica. Conservar atribución y evaluar condiciones aplicables a las bases derivadas. Las calles y los caminos por sí solos no establecen que haya servicio de bus. |
| Interprovincial, Cruz del Sur y SUTRAN | El operador publica Arequipa → Lima desde Terrapuerto Arequipa hasta Terminal Javier Prado, con dirección y datos de referencia. SUTRAN publica información sobre terminales autorizados. | Primer enlace comercial del piloto. Mostrar enlace y marcar el horario consultado como referencia hasta confirmar fecha y salida. El catálogo automatizado requiere acceso autorizado a datos del socio u operador. |

[ZIP Arequipa](https://raw.githubusercontent.com/trufi-association/trufi-arequipa/9b60aa5cc46e6b8f2dad63079770279c6cb1feb7/GTFS-Peru-Arequipa/out/city.gtfs.zip) · [ZIP MiRuta](https://raw.githubusercontent.com/leoguti/Trujillo-app/4eefdd1b7d6d763d1b01302dc222f478ade5f53b/GTFS/out/trujillo.gtfs.zip) · [OMUS en Mobility Database](https://mobilitydatabase.org/feeds/gtfs/mdb-2200) · [ATU](https://sistemas.protransporte.gob.pe/DatosAbiertos) · [Geovisor ATU](https://geovisor.atumovilidad.pe/) · [Plaza Cívica](https://plazacivica.pe/casos/digitalizacion-de-datos-de-transporte-urbano-en-lima-y-callao/) · [Geofabrik Perú](https://download.geofabrik.de/south-america/peru.html) · [Licencia OSM](https://www.openstreetmap.org/copyright/es) · [Cruz del Sur Arequipa–Lima](https://www.cruzdelsur.com.pe/pasajes/arequipa-a-lima/) · [SUTRAN](https://www.gob.pe/institucion/sutran/noticias/1067077-usuarios-pueden-usar-la-app-viaje-seguro-en-quechua-aymara-castellano-e-ingles)

### Verificación reproducible y criterio de publicación

Los tres ZIP se abrieron, pasaron el control de integridad CRC y las referencias revisadas no tuvieron registros huérfanos: viaje → ruta, tiempo de parada → parada y viaje, y frecuencia → viaje. Son comprobaciones estructurales parciales.

| Archivo | SHA-256 |
|---|---|
| Arequipa | 4b161e65eb1cfefe74d727424b747efff8980ca676a5c85b6fdc60a4b7a258bd |
| MiRuta Trujillo | 4bb0693ac43fd7c81097a3cc38f29ac53fb30ce09c6f4278f4c76afd4c37db23 |
| Trujillo OMUS archivado | 3f927c8e8335662463018667df7ce71a2f535793f1cd4bb466e55f50e8099eb0 |

Para publicar un corredor como verificado, registrar origen, destino, fecha de consulta, versión y condiciones de la fuente. Ejecutar el validador GTFS, cargar el archivo en OTP, comprobar las coordenadas de terminales y el orden de paradas en ambos sentidos, y contrastar una prueba de ida y otra de regreso con la realidad operativa. Mostrar frecuencia estimada, horario del operador o información en tiempo real únicamente según la evidencia de cada tramo.

La auditoría acredita insumos para prototipar mapas y rutas urbanas de dos ciudades, más una conexión interprovincial publicada. Quedan pendientes importar los feeds en OTP, verificar los datos sobre el terreno, resolver el último tramo del piloto en Lima y validar los derechos de redistribución de los datos complementarios. La cobertura del Perú se amplía por ciudad y corredor a medida que se completen esas pruebas.
