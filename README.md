# 7 Práctica Personas API
1. [Arrancar Zeppelin ](#schema1)
2. [Importación de librerías ](#schema2)
3. [Cargar ficheros ](#schema3)
4. [Crear una clase](#schema4)
5. [Crear función para procesar personas](#schema5)
6. [Conexión con SparkSQL](#schema6)
7. [Procesar la inforamción](#schema7)
8. [Convertir el RDD a dataset cacheado](#schema8)
9. [Uso de API en lugar del SQL Litera](#schema9)
10. [Filter](#schema10)
11. [GroupBy](#schema11)


<hr>

<a name="schema1"></a>

# 1. Arrancar Zeppelin
Navegamos en la consola hasta llegar donde tenemos descargados la carpeta Zeppelin y ejecutamos:
~~~
bin/zeppelin-daemon.sh start
~~~

Seguidamente abrimos un página en el navegador y vamos a `http://localhost:8080`, se nos abre zeppelin y creamos un nuevo notebook, llamado Temperatura Sensor y como intérprete elegimos `spark2`
<hr>

<a name="schema2"></a>

# 2. Importación de librerías

~~~scala
import org.apache.spark._
import org.apache.spark.SparkContext._
import org.apache.spark.sql._
import spark.implicits._
~~~

<hr>

<a name="schema3"></a>

# 3. Cargar ficheros

~~~scala
val lineas = spark.sparkContext.textFile("file:///home/patricia/Documentos/scala/7-personas-api/data/friends.csv")
~~~

<hr>


<a name="schema4"></a>

# 4. Creamos un clase Persona

~~~scala
case class Persona(ID: Int, nombre:String,edad:Int,numAmigos: Int)
~~~
<hr>

<a name="schema5"></a>

# 5. Crear función para procesar personas

~~~scala
def procesarPersona(linea:String):Persona = {
    val campos = linea.split(",")
    val persona = Persona(campos(0).toInt, campos(1),campos(2).toInt, campos(3).toInt )
    persona
}
~~~


<hr>

<a name="schema6"></a>

# 6. Conexion con SparkSQL
~~~scala
val spark = SparkSession.builder.appName("SQL API").getOrCreate()
~~~


<hr>

<a name="schema7"></a>

# 7. Procesar la inforamción
~~~scala
val personas = lineas.map(procesarPersona)
~~~

<hr>

<hr>

<a name="schema8"></a>

# 8. Convertir el RDD a dataset cacheado

Lo ponemos en cache para mantenerlo en memoria y poderlo manipular en un momento dado.
~~~scala
val personaDSCacheado = persona.toDS().cache()
personaDSCacheado.printSchema
~~~
![scala](./images/001.png)


<hr>

<a name="schema9"></a>

# 9. Uso de API en lugar del SQL Litera
Con `.show()` nos muestra en pantalla los 20 en caso de haberlos.
~~~scala
personaDSCacheado.select(personaDSCacheado("nombre")).show()
~~~
~~~scala
personaDSCacheado.select($"nombre").show()
~~~
Estas dos sentencias imprimen los mismo, al poner el símbolo `$`delante del campo a imprimir es lo mismo que si nombramos el dataset. 


![scala](./images/002.png)

Hay que separa con comas dentro del `select` para obtener más de un campo.
~~~scala
personaDSCacheado.select(personaDSCacheado("nombre"), personaDSCacheado("edad") + 10).show()
~~~
![scala](./images/005.png)
<hr>

<a name="schema10"></a>

# 10. Filter

~~~scala
personaDSCacheado.filter(personaDSCacheado("edad")<21).show()
~~~
![scala](./images/003.png)

<hr>

<a name="schema11"></a>

# 11. GroupBy
~~~scala
personaDSCacheado.groupBy("edad").count().show()
~~~
![scala](./images/004.png)