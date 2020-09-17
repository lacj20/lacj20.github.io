# Estructura de los índices

<div class=text-justify>

La función principal de un índice en una base de datos relacional es acelerar la ejecución de las sentencias SQL, el objetivo del presente material no es tener la conceptualización superflua de cómo trabaja un índice, en realidad, lo que se quiere es profundizar hasta donde sea necesario para lograr entender los aspectos a tomar en consideración cuando se trata de optimizar las consultas SQL.

<br/><br/>
Un índice se crea a través del comando `create index`, requiere de un espacio en disco y contiene una copia parcial de los datos en la tabla. Análogamente un índice de BD se parece mucho al índice de un libro, ocupa su propio espacio, es redundante y hace referencia a la información almacenada en otro lugar.
<br/><br/>

Buscar dentro de un almacén de datos indexado, es como buscar una entrada en un directorio telefónico. La base de su funcionalidad requiere una correcta _estructura ordenada_ de modo que este orden sirva para clasificar y determinar la posición de cada elemento lo más rápido y optimizado que se pueda, sin embargo, un índice en una BD es algo más complejo que un directorio telefónico debido a que constantemente está siendo modificado por operaciones de inserción, borrado o incluso actualización. Estas operaciones deben hacerse rápidamente conservando el orden y sin mover grandes cantidades de datos.
<br/><br/>

Claramente, un índice de BD es un desafío que, para nuestra fortuna, el motor de BD ya resuelve, no obstante, es imperativo que los desarrolladores conozcan tanto su estructura como su comportamiento.

</div>

La BD puede implementar dos tipos de índices, uno llamado [BTREE](btree.md) basado en las estructuras de datos [LISTA DOBLEMENTE ENLAZADA](lista-doble.md) y [ÁRBOL B](arbol-b.md), y el segundo tipo de índice a través de un HASH. Los índices de tipo btree son adecuados para consultas que usan operadores de relación y de rango, tales como <code> <, >, =, <> y between</code>, mientras que los de tipo hash son funcionales cuando las consultas usan únicamente operadores de igualdad exacta `=`, cabe mencionar que el índice de tipo hash puede presentar colisiones, suele ser más lento que btree en consultas que involucran rangos y no puede emplearse la cláusula `orden by` con ellos. En el caso particular de MySQL, se usan los índices de tipo btree, y por este motivo profundizaremos más en estos.


## Nodos hoja en un BTREE

Para entender paulatinamente la estructura interna y funcionamiento de los índices btree, es recomendable iniciar su análisis por los nodos hoja. El propósito de un índice es proporcionar una representación ordenada de los datos, sin embargo, no es posible almacenar datos de forma secuencial debido que una operación de `insert` tendría que mover grandes cantidades de datos provocando consumo excesivo de tiempo y lentitud en la ejecución. Este problema se soluciona usando una estructura que mantenga un orden lógico independiente del orden físico de los datos, esta estructura de datos se llama lista doblemente enlazada donde la ubicación física de los nuevos nodos que se insertan no tiene importancia debido al orden lógico que la estructura mantiene. Como esta estructura mantiene una referencia al nodo anterior y al siguiente, la BD puede leer el índice hacia adelante o hacia atrás según sea necesario. Además, la inserción o eliminación de un nodo no requiere que se muevan grandes cantidades de datos ya que solo se requiere actualizar los apuntadores.

Específicamente, los índices BTree utilizan una lista doblemente enlazada para apuntar los nodos hoja, cada nodo hoja se almacena en un bloque dentro de la BD, a este bloque también se le conoce como página y suele tener un tamaño de algunos kilobytes. El orden del índice se mantiene en dos niveles diferentes, los registros del índice que llevan a los nodos hoja y los nodos hoja interconectados entre sí por medio de una lista doblemente enlazada.

<div style="text-align:center;margin:2em 1em;">
    <img src="imagenes/nodoshoja.png"/><br/>
    <strong>Figura 1.1. La conexión entre los nodos hoja y los registros de la tabla.</strong>
</div>

La *figura 1.1* muestra la conexión que existe entre los nodos hoja (parte izquierda) y los registros de la tabla (parte derecha), nota que la tabla fue indexada por la columna 2, de modo que este campo es el que se toma en cuenta para construir los nodos hoja del índice, cada entrada en uno de la lista enlazada consta de al menos dos campos, la llave y un campo especial llamado rowid cuyo objetivo es referenciar la dirección física donde se encuentra almacenado el registro. A diferencia del índice, los datos de la tabla están almacenados en una estructura que no está ordenada, no existe relación entre los registros almacenados en el mismo bloque de la tabla, ni existen conexiones entre los bloques.

## Árbol de búsqueda BTREE

Para almacenar los índices, la BD hace uso de un [árbol de búsqueda equilibrado](arbol-busqueda-equilibrado.md), específicamente un BTree, esta estructura implementa nodos donde almacena información para encontrar rápidamente un nodo hoja en la lista enlazada, regularmente el tiempo para encontrar un nodo es logarítmico, se usa el mismo principio de la [búsqueda binaria](busqueda.binaria), iniciando en la raíz del árbol y descendiendo por sus ramas hasta los nodos hoja, escogiendo en el recorrido los subnodos de las ramas de acuerdo a la posición relativa del valor buscado respecto a los valores de cada nodo. Considere el siguiente ejemplo de un árbol de índices BTree.

<div style="text-align:center;margin:2em 1em;">
    <img id="f2" src="imagenes/estructuraBTree.png"/><br/>
    <strong>Figura 1.2. Estructura de un BTree.</strong>
</div>

La *figura 1.2* muestra la estructura de índices creada con un BTree, la lista doblemente enlazada establece el orden lógico entre los nodos hoja mientras la raíz y sus ramas permiten hacer búsquedas rápidas entre los nodos hoja. Para ponerlo de una forma más clara consideremos los siguientes puntos sobre el ejemplo en la figura:

1. Es un BTree que se ha construido con 30 registros.

2. Es de grado cuatro, es decir, el máximo número de hijos que puede tener un nodo es cuatro, en la *figura 1.3* podemos observar el nodo `[46, 53, 57, 83]` que tomaremos como nodo ejemplo, se dice que es de grado cuatro porque tiene cuatro hijos: **H1**, **H2**, **H3** y **H4**.

<div style="text-align:center;margin:2em 1em;">
    <img src="imagenes/estructuraNodo.png"/><br/>
    <strong>Figura 1.3. Estructura de un nodo en el BTree</strong>
</div>

3.	Las llaves que se agrupan en un nodo siempre están ordenadas de menor a mayor.

4.	Cada llave almacenada en el nodo, representa a un nodo hijo, esta llave representante es en realidad el valor más grande del nodo hijo al que apunta. En el nodo ejemplo la llave `83`, <code>[46, 53, 57, **83**]</code>, apunta al 4to. hijo, un nodo hoja que contiene las llaves <code>[67, 83, **83**]</code>, observa que el número mayor de este subconjunto representa al nodo.

5.	Nota que las tres llaves en el nodo hijo `[67, 83, 83]` están en el rango `[57, 83]` es decir, están contenidas en el intervalo que forman su llave representante `[83]` y la llave representante anterior `[57]`. Este orden en las llaves y los nodos se mantiene en toda la estructura.

6.	La profundidad (la distancia desde el nodo raíz a su nodo hoja más lejano) en cualquier hoja del árbol es la misma, esto asegura que buscar una llave específica siempre tardará un tiempo logarítmico. [Veáse análisis de complejidad](analisis-complejidad.md).
<div style="width:100%;margin:0px auto;background:#ebf3fc;padding:1em;border-radius:10px;text-align:justify;box-shadow:0px 0px 5px gray;position:relative;padding-left:25px;">
<img src="imagenes/idea.png" style="width:40px;position:absolute;left:-20px;top:15px;">
En resumen un BTree es una estructura de <em>árbol de búsqueda equilibrado</em> que mantiene su profundidad al mínimo posible, esta propiedad permite que todas las operaciones sean logarítmicas, esto quiere decir que son aceptablemente rápidas. Los manejadores de bases de datos que utilizan BTree para el ínidice, mantienen automáticamente la estructura cuando se aplican operaciones de <strong><em>insert</em></strong>, <strong><em>delete</em></strong> y <strong><em>update</em></strong>.
</div>

## Recorrido del índice BTree ##

Los recorridos en un árbol siempre inician desde el nodo raíz, y dependiendo del criterio de búsqueda, se desciende a través de sus nodos ramas hasta llegar a un nodo hoja. En el caso del BTree que estmos tomando como ejemplo, las operaciones de selección, inserción y borrado, siguen este patrón de búsqueda para eficientar el tiempo de ejecución en las consultas (Siempre que se indexe correctamente). 

Analicemos el recorrido en el índice para buscar una llave en el BTree que estamos tomando como ejemplo [Vea figura 1.2](#f2), Si nos enfocamos en buscar la llave **`57`** con la consulta SQL:

``` SQL
Select * from Tabla where columna2=57
```
Suponiendo que la *Tabla* está indexada sobre la *columna 2*, la *figura 1.4* muestra el recorrido del BTree que administra el índice.

<div style="text-align:center;margin:2em 1em;">
    <img src="imagenes/recorridoBtree.png" /><br/>
    <strong>Figura 1.4. Recorrido del índice BTree.</strong>
</div>

La ilustración muestra un fragmento del BTree en análisis, cuando la BD recibe la consulta, hace uso del índice para encontrar la llave que coincida con el criterio de búsqueda, en este caso `57`. El recorrido del árbol inicia desde el nodo raíz `[39, 83, 98]`, Se procesan las entradas en orden ascendente hasta encontrar un valor `>=` al valor buscado `57`, en este ejemplo se encuentra el valor `83`, esta llave hace referencia a un nodo en el siguiente nivel del árbol <code>[46, 53, 57, ***83***]</code>, sobre este nodo rama se repite el proceso encontrando el valor `57` y tomando la referencia para saltar al siguiente nivel del árbol, específicamente en el nodo hoja que contiene la llave que estamos buscando `[55, 57, 57]`, recordemos que los nodos hoja guardan la llave y la dirección donde está almacenado el regitro asociado, entonces, para cada entrada de este nodo que coincida con la llave buscada se devuelve un registro, en este caso obtendriamos dos registros asociados a la llave `57`. 

Es importante resaltar que el uso del índice nos reduce significativamente el costo de encontrar un registro, si analizas detenidamente la *figura 1.4*, solo se consultaron cinco de las treinta llaves para encontrar el nodo hoja que apunta a la ubicación real del registro. A groso modo podemos decir que el costo de encontrar la llave en este ejemplo es igual a 5, sin tomar en cuenta el costo del acceso a disco que la BD tiene que hacer para devolver los datos solicitados del registro en cuestión. 
<div style="background:#ebf3fc;padding:1em;border-radius:10px;text-align:justify">
El índice BTree nos permite encontrar registros rápidamente
</div>