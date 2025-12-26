# apache-flink-basic

Hola, como parte del último tópico a estudiar, producto de una oferta de trabajo que ando persiguiendo, entraremos
a un tópico chido que es el straming de datos. Esto lo hago con la finalidad de que me sirva como cheatsheet para mi
y algún otro desafortunado que ande en las mismas.

Cuando se hace procesamiento por batches, o etl's donde haces un madral de scripts para procesar datos haciendo joins
tal vez por código para balancear la carga entre el lenguaje y tu SGBD mientras haces join de esa info y añades lógica de negocio
para unificar esa información, generalmente siempre es por scripts, y tu sabrás que en modelos de aplicación old school
es por consulta con offsets, una unificación de joins de vectores de tu lenguaje, filters, condicionales, todo eso durante ciertos
intervalos para no sobresaturar nada, luego si lees csv's también entra en lo mismo, pero todo es por batch, no hay un etl consecuente...

hasta que entra Flink.

Como se ha discutido en anteriores prácticas, Kafka sirve no solo como una cola, sino que además es de ALTO RENDIMIENTO, 
para miles de mensajes te los entrega consecuentemente y bien. Ahora imagina que tienes muchisimos datasources como:

metadatos de archivos

csv's de backups de tus bdd's tal vez para deupurar información.

Información de documentos que a lo mejos escaneaste con alguna libreria de python para extraer el texto de documentos o imagenes

Bases de datos distribuidas

XML'S, facturas, etc.

Kafka te ayuda a tener un paralelismo en tiempo real para procesas toda esa información que tu ya categorizaste para poder
hacer un stream de datos, ya no mandas batch, lo mandas enseguida, como si arrojaras fichas a la mesa, no agarras un manojo
y los pasas a tu compañero. Imagina que tienes secretarios con varias manos dispuestos a procesar toda esa info de elementos, en vez
de un secretario que te acepta un paquete de hojas cada hora, toma un descanso y le sigue y asi mero, es PROCESAMIENTO EN TIEMPO REAL.

Ahora, mandas las fichas, u hojas, si tu secretario es lento pues solo aunque reciba una por una va a ir juntando o se va a tardar por hoja.
Con flink, tienes deformes con varios brazos que en lo que tu le pasas uno, ya acabaron una hoja y asi mero. Se denomina streaming ETL, y no
es por cada hora o por noche, es CONSTANTE. Tienes un ETL que corre a cada segundo, las 24/7, sobre datos infinitos, y tu eres el encargado
de decidir como procesas y guardas esos datos. Osea no calculas para guardar, solo te encargas de guardar, y se crea una ventana temporal con datos ya hechos. No harás
un insert a cada rato, y eso si lo analizas bien, reduce tiempo.

Pero esto no desmerita los procesos old school, unos economizan costos y va de acuerdo a la urgencia y magnitud de la data, aqui
es para temas de otra liga...

Ahora, procederé a empaparme de la info de su api de flink. Entendiendo las bases ahora ya sabes porque se usan estas herramientas.
