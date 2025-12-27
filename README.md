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

# Primeros avances
Aprender sobre las diferencias entre los diferentes tipos de arranque del flink

<img width="1240" height="1074" alt="image" src="https://github.com/user-attachments/assets/327e27a8-4526-4eef-8586-11fc89d0989f" />

Lo que sé es que la persistencia de los procesados se puede dar en memoria o por medio de rocksDB que persiste los datos
segmentados en disco hasta cierto limite. 

CREATE TABLE Orders (
    order_number BIGINT,
    price        DECIMAL(32,2),
    buyer        ROW<first_name STRING, last_name STRING>,
    order_time   TIMESTAMP(3)
) WITH (
  'connector' = 'datagen',
  'rows-per-second' = '10',
  'number-of-rows' = '100'
);


Ok.... los fundamentos si están con una curva alta, aun con la IA porque me gusta ver si algo que aparece ahi está en la documentación y ver como lo describen ahí.

Ok, para esto primero empezamos con flink SQL, es un gestor de vistas que nos permite manipular y crear consultas con los datos del stream que pasan en memoria. 
Entonces, hay el concepto de la consulta y el SINK, este último es la definición de a donde van tus datos transformados y procesados. 

Creas una tabla demo que es la definición de como llega tu dato. Y el SINK es el como lo procesas.

<img width="1778" height="544" alt="image" src="https://github.com/user-attachments/assets/a9528f37-d8bd-4c32-9616-f696052327b8" />

Aquí creo la tabla orders y como connector pongo 'datagen', uno de los tantos connectors que existen:

<img width="508" height="1027" alt="image" src="https://github.com/user-attachments/assets/b028c960-2df3-4e8f-981a-2a6ec2cc6d2c" />

Datagen genera datos aleatorios para probar tus procesados. 

Entonces al crear un insert into saving_ex select count(*) saving_count,buyer from Orders group by buyer;, lo que haces es que los datos que vienen del connector anexado
a la tabla receiver por asi llamarla, se van agregando al SINK, es decir guardarlos. Se guardan rápidon.
create table saving_ex(saving_count bigint, buyer ROW<first_name STRING, last_name STRING>) with ('connector'= 'print');

En este caso nuestro otro conector es el print, es decir lo imprime en los logs y ya. Esto es para pruebas, y puedes observar en vivo como se populan los datos

<img width="1913" height="1079" alt="image" src="https://github.com/user-attachments/assets/15b5c44c-c4e6-4c6c-97ba-f910d07c3888" />

Los archivos out son donde van tus print

<img width="1426" height="196" alt="image" src="https://github.com/user-attachments/assets/1c319cc5-3723-4f4a-a089-a911adc6976e" />

Los logs pues son los logs que puedes ver desde tu dashboard de flink 

<img width="2162" height="1322" alt="image" src="https://github.com/user-attachments/assets/3aee723c-8d73-4939-8e09-53bb7f4d6ac4" />


Siii, si tiene su curva de aprendizaje :p

Entonces source es lo que va recibiendo de la fuente de datos, el pipeline o job aborda los operators, y las transformaciones 

<img width="1703" height="1406" alt="image" src="https://github.com/user-attachments/assets/ba7964c5-a734-44e7-970e-72b6d8d55951" />

Y sink guarda o redirecciona. Es la capa del transform. Puntos sobre los que tuve duda y aclaro aquí:

*El sink y posterior se encarga de procesar duplicados, esto porque flink no lo detecta, importante
si fluye el source desde kafka y reiniciar el offset.

*En el momento que tu enciendes la canalización de tus datos, tu start, ya debes definir en sink, puesto que tus datos
no viven dentro de flink, solo los procesa. Si quieres guardarlos debes definir tu sink.

*Hay procesing time y event time, en prod se usa event time siempre ya que es más preciso.

*Las funciones window te ayudan a mantener el orden pero sobre todo, en un stram infinito te ayudan a "batchear" en el save
de acuerdo a intervalos de tiempo.
