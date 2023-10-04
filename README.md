# Ejercicio: Servicio de alquiler de bicicletas públicas de Sevilla (Sevici)
**Autor**: Mariano González.      **Revisores**: Carlos G. Vallejo, José A. Troyano, Fermín Cruz, Toñi Reina.     **Última modificación:** 04/10/2023

## Requisitos previos
En este ejercicio vamos a trabajar con mapas, para lo que usaremos la librería ```folium``` . Para instalar la librería ```folium``` abre una ventana de comandos de Anaconda (Anaconda Prompt) y ejecuta el siguiente comando:
```
pip install folium
```
O si tu instalador es conda
```
conda install –c conda-forge folium
```

En este ejercicio vamos a trabajar con la red de estaciones del servicio de alquiler de bicicletas de Sevilla, Sevici. Para ello disponemos de los datos de las estaciones, obtenidos de [https://citybik.es/](https://citybik.es/).

En primer lugar leeremos los datos de las estaciones desde un fichero CSV. Realizaremos algunas operaciones con los datos, como obtener las estaciones con bicicletas libres o las estaciones más próximas a nuestra ubicación. Finalmente, dibujaremos sobre el mapa las estaciones, distinguiendo entre las que tienen bicicletas disponibles y las que no las tienen.

Las funciones que vamos a desarrollar son las siguientes:

1. **lee_estaciones**: lee los datos de las estaciones desde un fichero csv
2. **estaciones_bicis_libres**: crea una lista con las estaciones que tienen bicicletas libres, ordenada por el número de bicis libres
3. **calcula_distancia**: calcular la distancia a una estación desde un punto dado
4. **estaciones_cercanas**: crea una lista con las estaciones con bicis libres más cercanas a un punto dado, ordenadas de la más cercana a la más lejana
5. **media_coordenadas**: devuelve una coordenada cuya latitud es la media de las latitudes y cuya longitud es la media de las longitudes.


## 1. Carga de datos
Se dispone de los datos de las estaciones de la red de Sevici. Los datos se encuentran en un fichero CSV. Cada línea del fichero contiene seis datos:

    Nombre de la estación
    Número total de bornetas de la estación
    Número de bornetas vacías
    Número de bicicletas disponibles
    Latitud
    Longitud

Los datos dependen del instante en que se obtiene el fichero, ya que el número de bicicletas y bornetas libres varía de forma continua. Estan serían, por ejemplo, las primeras líneas del fichero en un momento dado:

        name,slots,empty_slots,free_bikes,latitude,longitude
        149_CALLE ARROYO,20,11,9,37.397829929383,-5.97567172039552
        257_TORRES ALBARRACIN,20,16,4,37.38376948792722,-5.908921914235877
        243_GLORIETA DEL PRIMERO DE MAYO,15,6,9,37.380439481169994,-5.953481197462845
        109_AVENIDA SAN FRANCISCO JAVIER,15,1,14,37.37988413609134,-5.974382770011586
        073_PLAZA SAN AGUSTIN,15,10,4,37.38951386231434,-5.984362789545622

Los principales aspectos que tendremos que resolver a la hora de procesar estos datos de entrada serán saltar la línea de encabezado del fichero, separar adecuadamente los campos mediante las comas e interpretar el formato de cada uno de los campos, que puede ser de tipo cadena, entero o real.

Para resolver estos problemas haremos uso de algunas utilidades disponibles en la librería estándar de Python. En concreto, antes de empezar, deberemos importar los siguientes elementos:


```python
import csv
import math 
import folium
from collections import namedtuple
```

### 1.1 Definición de tipos basados en ```namedtuple```

 Como el trabajar con coordenadas es algo frecuente, vamos a definir una tupla con nombre llamada ```Coordenadas``` que representa una coordenada con latitud y longitud. Además, definiremos también una tupla con nombre ```Estacion``` que representa una de las estaciones de bicicletas repartidas por la ciudad, con los campos ```nombre```, ```bornetas```, ```bornetas_vacias```, ```bicis_diponibles``` y ```coordenadas```. Fíjate en que este último representa a una coordenada compuesta por latitud y longitud, es decir, será de tipo ```Coordenadas```.


```python
# Creación de un tipo de namedtuple para las coordenadas
# type: Coordenadas(float, float)
Coordenadas = namedtuple('Coordenadas', 'latitud, longitud')
# Creación de un tipo de namedtuple para las estaciones
# type: Estacion(str, int, int, int, Coordenadas(float, float))
Estacion = namedtuple('Estacion', 'nombre, bornetas, bornetas_vacias, bicis_disponibles, coordenadas')
```

### 1.2 Lectura de fichero

La siguiente función será la encargada de leer el fichero con las estaciones y construir a partir de él una estructura de datos en memoria.


```python

# Función de lectura que crea una lista de Estaciones
def lee_estaciones(fichero):
    ''' Lee el fichero de datos y devuelve una lista de estaciones
    
    ENTRADA: 
       :param fichero: Nombre y ruta del fichero a leer
       :type fichero: str
   
    SALIDA: 
       :return: Lista de tuplas de tipo Estacion
       :rtype: [Estacion(str, int, int, int, Coordenadas(float, float))]
    
    Cada estación se representa con una tupla con los siguientes valores:
    - Nombre de la estación
    - Número total de bornetas
    - Número de bornetas vacías
    - Número de bicicletas disponibles
    - Coordenadas
    Usaremos el módulo csv de la librería estándar de Python para leer el
    fichero de entrada.
    Hay que saltar la línea de encabezado del fichero y comenzar a leer los datos
    a partir de la segunda línea.
    Hay que realizar un pequeño procesamiento con los datos numéricos. Hay que
    pasar del formato cadena (que es lo que se interpreta al leer el csv) a un
    valor numérico (para poder aplicar operaciones matemáticas si fuese necesario).
    También hay que crear una tupla con nombre de tipo Coordenadas
    '''
   pass
```

Cree un método ```test_lee_estaciones``` en un módulo sevici_test, que tome como entrada una lista de tuplas de tipo Estacion, de las leídas a partir del archivo, y muestre una salida por consola como la siguiente:

```python
Las tres primeras son :
Estacion(nombre='149_CALLE ARROYO', bornetas=20, bornetas_vacias=11, bicis_disponibles=9, coordenadas=Coordenadas(latitud=37.397829929383, longitud=-5.97567172039552))
Estacion(nombre='257_TORRES ALBARRACIN', bornetas=20, bornetas_vacias=16, bicis_disponibles=4, coordenadas=Coordenadas(latitud=37.38376948792722, longitud=-5.908921914235877))
Estacion(nombre='243_GLORIETA DEL PRIMERO DE MAYO', bornetas=15, bornetas_vacias=6, bicis_disponibles=9, coordenadas=Coordenadas(latitud=37.380439481169994, longitud=-5.953481197462845))
Las tres últimas son :
Estacion(nombre='203_CALLE TESALÓNICA', bornetas=20, bornetas_vacias=7, bicis_disponibles=13, coordenadas=Coordenadas(latitud=37.396093518802566, longitud=-5.958764096016824))
Estacion(nombre='145_AVENIDA REINA MERCEDES', bornetas=20, bornetas_vacias=4, bicis_disponibles=16, coordenadas=Coordenadas(latitud=37.360233228861155, longitud=-5.986318234050904))
Estacion(nombre='048_CALLE CHURRUCA', bornetas=15, bornetas_vacias=11, bicis_disponibles=3, coordenadas=Coordenadas(latitud=37.39715651121527, longitud=-5.991066209916829))
```

## 2. Operaciones de consulta

En esta sección veremos una serie de funciones que nos permitirán filtrar y extraer informaciones de la estructura de datos que estamos manejando para representar las estaciones (una lista de tuplas). Utilizaremos estas funciones de _consulta_ en distintos puntos del resto del ejercicio.

### 2.1 Estaciones con bicicletas libres

La primera función de este tipo creará una lista con las estaciones que tienen un número de bicicletas libres superior a un valor dado, que por defecto es 5. La lista de salida estará formada por tuplas con dos elementos, el número de bicicletas libres y el nombre de la estación, y estará ordenada por el número de bicicletas libres


```python
def estaciones_bicis_libres(estaciones, k=5):
    ''' Estaciones que tienen bicicletas libres
    
    ENTRADA: 
      :param estaciones: lista de estaciones disponibles 
      :type estaciones: [Estacion(str, int, int, int, Coordenadas(float, float))]
      :param k: número mínimo requerido de bicicletas
      :type k: int
    SALIDA: 
      :return: lista de estaciones seleccionadas
      :rtype: [(int, str)] 
    
    Toma como entrada una lista de estaciones y un número k.
    Crea una lista formada por tuplas (número de bicicletas libres, nombre)
    de las estaciones que tienen al menos k bicicletas libres. La lista
    estará ordenada por el número de bicicletas libres.
    '''
    pass
```

Cree un método `test_estaciones_bicis_libres` en un módulo sevici_test, que tome como entrada una lista de tuplas de tipo Estacion, de las leídas a partir del archivo, y muestre una salida por consola como la siguiente (note que hay que invocar al test 3 veces, una para obtener las estaciones con 5 o más bicis libres, otra para obtener las que tienen 10 o más bicis libres, y, finalmente las que tienen 1 bici libre o más):

```python
Hay 147 estaciones con 5 o más bicis libres y las 5 primeras son:
(9, '149_CALLE ARROYO')
(9, '243_GLORIETA DEL PRIMERO DE MAYO')
(14, '109_AVENIDA SAN FRANCISCO JAVIER')
(5, '082_CALLE LUIS MONTOTO')
(5, '226_AVENIDA DOCTOR EMILIO LEMOS')
Hay 83 estaciones con 10 o más bicis libres y las 5 primeras son:
(14, '109_AVENIDA SAN FRANCISCO JAVIER')
(11, '209_AVENIDA ALEMANIA')
(12, '128_VIRGEN DE LORETO')
(35, '087_PLAZA NUEVA')
(11, '220_AVENIDA ALCALDE LUIS URUÑUELA')
Hay 225 estaciones con 1 o más bicis libres y las 5 primeras son:
(9, '149_CALLE ARROYO')
(4, '257_TORRES ALBARRACIN')
(9, '243_GLORIETA DEL PRIMERO DE MAYO')
(14, '109_AVENIDA SAN FRANCISCO JAVIER')
```

    Hay 147 estaciones con 5 o más bicis libres: [(5, '024_CALLE LEÓN XIII'), (5, '043_RONDA CAPUCHINOS'), (5, '051_AVENIDA TORNEO'), (5, '064_CALLE CUESTA DE ROSARIO'), (5, '082_CALLE LUIS MONTOTO')]
    Hay 83 estaciones con 10 o más bicis libres: [(10, '025_AVENIDA DE LA CRUZ ROJA'), (10, '057_PLAZA CRISTO DE BURGOS'), (10, '074_PLAZA PILATOS'), (10, '097_PASEO DE CRISTÓBAL COLÓN'), (10, '105_CALLE FRANCISCO MURILLO')]
    Hay 225 estaciones con al menos una bici libre: [(1, '001_GLORIETA OLIMPICA'), (1, '013_CALLE FERIA'), (1, '015_CALLE DR MARAÑON'), (1, '019_PARLAMENTO'), (1, '026_AVENIDA DE MIRAFLORES')]
    

### 2.2 Estaciones cercanas a una ubicación

La segunda función de consulta creará una lista con las estaciones más próximas a un punto dado que tengan bicicletas libres.

En primer lugar vamos a escribir una función que calcule la distancia entre dos puntos dados por su latitud y longitud. Esta función nos permitirá calcular la distancia entre un punto (nuestra ubicación, por ejemplo) y las estaciones cercanas. Utilizaremos la distancia euclídea con una pequeña modificación para tener en cuenta el número de bicicletas libres que hay en la estación. El motivo es el siguiente: si tenemos dos estaciones a una distancia similar, lo más lógico es elegir aquella que tenga más bicicletas libres, ya que será más probable que encontremos una bicicleta cuando lleguemos a la estación.

La fórmula que usaremos para calcular la distancia entre una ubicación dada por las coordenadas $(x, y)$ y una estación dada por las coordenadas $(xe, ye)$ que tiene $fb$ bicicletas libres será la siguiente:

$$
distancia = \sqrt {(xe-x)^2 + (ye-y)^2}
$$


```python
def calcula_distancia(coordenadas1, coordenadas2):
    ''' Distancia entre un punto y una estación
    ENTRADA: 
    :param coordenadas1: coordenadas del primer punto
    :type coordenadas1: Coordenadas(float, float)
    :param coordenadas2: coordenadas del segundo punto
    :type coordenadas2: Coordenadas(float, float)
      
    SALIDA: 
    :return: distancia entre dos coordenadas
    :rtype: float 
    
    Toma como entrada dos coordenadas y calcula la distancia entre ambas aplicando la fórmula
    
        distancia = sqrt((x2-x1)**2 + (y2-y1)**2)
    '''
    pass
```

Ahora vamos a escribir la función que calcula las estaciones más cercanas a una ubicación dada y que tienen bicicletas libres, utilizando la función anterior para calcular la distancia desde la ubicación hasta cada estación.

La función recibe la lista de estaciones, las coordenadas de la ubicación y un número k, y crea una lista con las k estaciones más cercanas a la ubicación, ordenadas de menor a mayor distancia. La lista resultante está formada por tuplas con tres elementos: la distancia a la ubicación, el nombre de la estación y el número de bicicletas libres de la estación.


```python
def estaciones_cercanas(estaciones, coordenadas, k=5):
    ''' Estaciones cercanas a un punto dado
    
    ENTRADA: 
      :param estaciones: lista de estaciones disponibles
      :type estaciones: [Estacion(str, int, int, int, Coordenadas(float, float))]
      :param coordenadas: coordenadas formada por la latitud y la longitud de un punto
      :type coordenadas: Coordenadas(float, float)
      :param k: número de estaciones cercanas a calcular 
      :type k: int
    SALIDA: 
      :return: Una lista de tuplas con la distancia, nombre y bicicletas libres de las estaciones seleccionadas 
      :rtype: [(float, str, int)] 
    
    Toma como entrada una lista de estaciones,  las coordenadas de  un punto y
    un valor k.
    Crea una lista formada por tuplas (distancia, nombre de estación, bicicletas libres)
    con las k estaciones con bicicletas libres más cercanas al punto dado, ordenadas por
    su distancia a las coordenadas dadas como parámetro.
    '''
    pass
```
Cree un método ```test_estaciones_cercanas``` en un módulo sevici_test, que tome como entrada una lista de tuplas de tipo Estacion, de las leídas a partir del archivo, y la coordenada de un punto y muestre algo como lo siguiente (invocar a este test usando la coordenada (37.357659, -5.9863):

```python
# Test de la función
Las 5 estaciones más cercanas al punto 37.357659, -5.9863 son:
(0.0005378435081587429, '126_AVENIDA REINA MERCEDES', 13)
(0.0013179067450871994, '040_AVENIDA REINA MERCEDES', 7)
(0.0024416608855902374, '146_AVENIDA REINA MERCEDES', 10)
(0.0025742934390284335, '145_AVENIDA REINA MERCEDES', 16)
(0.00473574709175822, '207_CALLE IFNI', 15)
```

    

## 3. Visualización de estaciones

Vamos a visualizar gráficamente algunos resultados. Para ello dibujaremos un mapa y sobre él marcaremos las estaciones que cumplan algunos requisitos, como por ejemplo tener un número mínimo de bicicletas libres.

Ejecuta en primer lugar las dos siguientes funciones. La primera sirve para crear un objeto mapa de _folium_, y le pasaremos la latitud y la longitud que tendrá el punto del centro del mapa, y el nivel de zoom inicial con el que se mostrará el mapa. La segunda crea un objeto de tipo Marcador de _folium_ que representa un marcador que vamos a dibujar en el mapa. Para crear el marcardor tendremos que dar su latitud y longitud, la etiqueta que queremos que aparezca cuando el usuario haga clic sobre el mismo y el color en el que se dibujará el marcador.  


```python
def crea_mapa(latitud, longitud, zoom=9):
    '''
    Función que crea un mapa folium que está centrado en la latitud y longitud
    dados como parámetro y mostrado con el nivel de zoom dado.
    ENTRADA:
        :param latitud: latitud del centro del mapa en pantalla
        :type latitud:float
        :param longitud: longitud del centro del mapa  en pantalla
        :type longitud: float
        :param zoom: nivel del zoom con el que se muestra el mapa
        :type zoom: int
    SALIDA:
        :return: objeto mapa creado
        :rtype: folium.Map
    '''
    mapa = folium.Map(location=[latitud, longitud], 
                      zoom_start=zoom)
    return mapa    
```


```python
def crea_marcador (latitud, longitud, etiqueta, color):
    '''
    Función que crea un marcador rojo con un icono de tipo señal de información.
    El marcador se mostrará en el punto del mapa dado por la latitud y longitud
    y cuandos se mueva el ratón sobre él, se mostrará una etiqueta con el texto
    dado por el parámetro etiqueta
    ENTRADA:
        :param latitud: latitud del marcador
        :type latitud: float
        :param longitud: longitud del marcador
        :type longitud: float
        :param etiqueta: texto de la etiqueta que se asociará al marcador 
        :type etiqueta: str
    SALIDA:
        :return: objeto marcador creado
        :rtype:folium.Marker
    '''
    marcador = folium.Marker([latitud,longitud], 
                   popup=etiqueta, 
                   icon=folium.Icon(color=color, icon='info-sign')) 
    return marcador

```

Ahora solo nos queda crear el mapa y añadirle las estaciones que queremos que aparezcan dibujadas como marcadores. Para ello crearemos la función ```crea_mapa_estaciones```, que se encargará de lo siguiente:
1. Calcular cual va a ser el punto central del mapa.
2. Crear el objeto mapa usando como centro la coordenada obtenida en el punto anterior.
3. Recorrerse todas las estaciones y para cada estación a representar en el mapa, crear un objeto de tipo marcador, y añadirlo al mapa.

Para los puntos 2 y 3 vamos a usar las funciones ```crea_mapa``` y ```crea_marcador``` definidas anteriormente. Para el punto 1, es decir, calcular el centro del mapa, definiremos una función auxiliar que calcule la media de las coordenadas de las estaciones que se van a representar en el mapa, con objeto de que el mapa quede centrado.


```python
def media_coordenadas (estaciones):
    '''Devuelve una coordenada cuya latitud es la media de las latitudes y
    cuya longitud es la media de las longitudes.
    ENTRADA
        :param estaciones: lista de estaciones disponibles
        :ptype estaciones: [Estacion(str, int, int, int, Coordenadas(float, float))]
    SALIDA
        :return: Una coordenada cuya longitud es la media de las longitudes, y la 
             latitud la media de las latitudes
        :rtype: Coordenadas(float, float)
    '''
    pass
```

La función ```crea_mapa_estaciones``` es la función que se va a encargar de crear el mapa con todos los marcadores. Como los marcadores en unas ocasiones los dibujaremos de un color, y en otras de otro, la función ```crea_mapa_estaciones``` tomará como parámetro, además de las estaciones a representar en el mapa, una función que se aplicará a cada estación para obtener el color en el que se dibuja. En Python, las funciones también se pueden pasar como parámetro como cualquier otro tipo.


```python
def crea_mapa_estaciones(estaciones, funcion_color):
    '''Genera un objeto de tipo folium.Map con un marcador por cada
    estación dada como parámetro. El marcador tendrá como etiqueta
    el nombre de la estación, y su color se obtendrá a partir de la 
    función ```funcion_color``` que se pasa como parámetro
    ENTRADA
        :param estaciones: lista de estaciones disponibles
        :type estaciones: [Estacion(str, int, int, int, Coordenadas(float, float))]
        :param funcion_color: Función que se aplica a una estación y devuelve una cadena
            que representa el color en el que se dibuja el marcador
        :type funcion_color: function(Estacion)->str
    SALIDA
        :return: objeto mapa creado con los marcadores
        :rtype: folium.Map
    '''
    #Calculamos la media de las coordenadas de las estaciones, para poder centrar el 
    #mapa
    centro_mapa = media_coordenadas(estaciones)
    # creamos el mapa con folium
    mapa = crea_mapa(centro_mapa.latitud, centro_mapa.longitud, 13)

    for estacion in estaciones:
        etiqueta = estacion.nombre
        color = funcion_color(estacion)
        marcador = crea_marcador (estacion.coordenadas.latitud, estacion.coordenadas.longitud, etiqueta, color)
        marcador.add_to(mapa)
    
    return mapa

```
Finalmente, una vez creado un mapa, lo podemos guardar en un archivo html usando la siguiente función:

```python
def guarda_mapa(mapa, ruta_fichero):
    '''Guard un mapa como archivo html

    :param mapa: Mapa a guardar
    :type mapa: folium.Map
    :param ruta_fichero: Nombre y ruta del fichero
    :type ruta_fichero: str
    '''
    mapa.save(ruta_fichero)
    # Abre el fichero creado en un navegador web
    webbrowser.open("file://" + os.path.realpath(ruta_fichero))
```

### 3.1 Mostrar todas las estaciones

En primer lugar vamos a mostrar sobre el mapa todas las estaciones de la red. Esto nos va a servir también para ver cómo podemos pasar una función como parámetro real. Si todas las estaciones las queremos dibujar en azul, simplemente definiremos una función que devuelva siempre "blue", independientemente de la estación.


```python
def color_azul(estacion):
   '''Función que devuelve siempre azul
   ENTRADA
      :param estacion: Estación para la que quiero averiguar el color
      :type estacion: Estacion(str, int, int, int, Coordenadas(float, float))
   SALIDA
      :return: El color azul
      :rtype: str
   '''
   return "blue"
```

Para probar esta nueva funcionalidad solo tiene que invocar a la función ```crea_mapa_estaciones``` y luego guardar el mapa creado, de la siguiente forma:
```python
mapa_estaciones = crea_mapa_estaciones(estaciones_sevici, color_azul)
guarda_mapa(mapa_estaciones, "./out/azul.html"))
```
Se debe generar un archivo html en la carpeta out con un contenido similar al siguiente:

![](/img/estaciones_azul.png)

Si lo que queremos ahora es crear un mapa en el que se pinten en verde las estaciones que tengan bicis disponibles y en rojo las que no tengan ninguna bici disponible, habrá que crear una función

```python
def obten_color_bicis_disponibles(estacion):
    '''Función que devuelve "red" si la estación no tiene bicis disponibles, y verde en caso contrario 
    ENTRADA
      :param estacion: Estacion para la que quiero averiguar el color
      :type estacion: Estacion(str, int, int, int, Coordenadas(float, float))
    SALIDA
      :return: "red" o "green" dependiendo de si la estación tiene bicis disponibles o no
      :rtype: str
    '''
    res="red"
    if estacion.bicis_disponibles>0:
        res="green"
    return res
```
Para probar esta nueva funcionalidad solo tiene que invocar a la función ```crea_mapa_estaciones``` y luego guardar el mapa creado de la siguiente forma:
```python
mapa_estaciones = crea_mapa_estaciones(estaciones_sevici, obten_color_bicis_disponibles)
guarda_mapa(mapa_estaciones, "./out/estaciones_bicis_disponibles.html"))
```

Se debe generar un archivo html en el que se muestre una distribución similar a la siguiente imagen. Ten en cuenta que la imagen está generada tomando solamente las primeras 5 estaciones para que se vea mejor.
![](/img/estaciones_verdes.png)

¿Qué tendrás que hacer si ahora quiere generar un mapa en el que se dibujen las estaciones que tengan bornetas libres para poder dejar la bici? Añade el código neceario para realizar esta operación.


