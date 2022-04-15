# Bases de datos para estudios de Abies pinsapo en Andalucía. Parte III

Jessica Bernal
14/04/22

*Nota: Se adjunta html con resultados en la repo*

### 3. Estado sanitario del monte

En partes anteriores se ha empleado una visión clásica de la ciencia de datos en aplicaciones forestales utilizando código abierto, R, que nos permite trabajar de manera rápida y eficiente en el uso de grandes conjuntos de datos. En esta sesión se propone construir lo que se conoce como Data Lake o "Lago de datos", un repositorio centralizado que contiene una gran cantidad de datos en bruto que se mantienen allí hasta que sea necesario. Es decir, el dato se puede guardar sin modificarlo o transformarlo. De hecho, los tipos de datos que permite van desde estructurados (modelo relacional tradicional), semi-estructurados (CSV, XML, JSON, etc.), datos no estructurados (correos electrónicos, PDFs, documentos) hasta datos binarios (como audios o videos).

De entre las múltiples opciones comerciales, los más utilizados son Azure Data Lake
![image](https://user-images.githubusercontent.com/100314590/163583663-4bec3a18-255f-4ea5-a989-8a334a2bf72f.png)

y AWS (Amazon Web Services) Data Lake
![image](https://user-images.githubusercontent.com/100314590/163583676-3de85ddd-0c8e-4eef-99da-422aa24a21ad.png)

No obstante, para el micro Data Lake que se va a generar en esta ocasión, RStudio será suficiente.

Generalmente, se considera que el procesado de los datos se compone de cinco estados:(1) Datos descargados, (2) Datos preparados y limpios, (3) Análisis de datos, (4) Presentación de datos, y (5) Datos contando una historia.



#### 3.1. Recopilación de los datos
Antes de comenzar a recopilar datos de distintas fuentes, es necesario conocer qué se quiere hacer con ellos y plantear los objetivos que se quieren alcanzar. Tener unas preguntas claras ayudará a determinar qué datos las responden mejor y cómo deben procesarse. En el caso práctico con el que se está trabajando, se quiere complementar el estudio de la vegetación del inventario de ordenación con información del estado sanitario de la masa, conocer su salud y su evolución temporal.

El Reglamento CEE 3528/86 sobre protección de bosques contra los efectos de la contaminación atmosférica, puso en marcha una serie de acciones para el seguimiento del estado de los ecosistemas forestales en todos los países comunitarios, entre ellos, el establecimiento de la Red Europea de Seguimiento de Daños en los Bosques con muestreos sistemáticos anuales de la evolución del estado de salud de los bosques en parcelas sobre una malla de 16x16km.

![image](https://user-images.githubusercontent.com/100314590/163583690-53eff5f7-a490-4e50-b912-8034c79f6182.png)

Posteriormente, la Junta de Andalucía lanzó en el año 2000 una Red Autonómica de Equilibrios Biológicos, sobre la base de la malla kilométrica existente, pero densificada de 8 x 8 km (Red SEDA). Para el caso concreto de la especie de pinsapo el muestreo se intensificó en una malla de 1 x 1 km en ecosistemas con presencia de *Abies pinsapo* (Red PINSAPO).

![image](https://user-images.githubusercontent.com/100314590/163583703-4909f7c9-40b2-4d6c-865c-85eed16e57ee.png)

Los datos más actualizados de la Red PINSAPO están a disposición del usuario en la web de descargas de la REDIAM: [RedSedaPinsapo](https://descargasrediam.cica.es/repo/s/RUR?path=%2F05_CALIDAD_AMBIENTAL%2F04_ECOSISTEMAS_FORESTALES%2F01_EQUILIBRIOS_BIOLOGICOS%2FRedSedaPinsapo)

Se trata de una base de datos relacional en formato mdb., aunque para su distribución, las tablas originales han sido exportadas a formato csv. Sin embargo, para su correcto uso es necesario entender la arquitectura con la que fue diseñada, información que podemos consultar en este caso dentro de la carpeta *Documentos* > archivo *Relaciones_RedPinsapo.pdf*

![image](https://user-images.githubusercontent.com/100314590/163583724-cbd8c5aa-4d80-4c8e-a831-62ba626c6ac3.png)

Una vez descargada la información, cargamos los datos:

```{r}
Admon.pto<-read.csv("C:/Geodirectorio/Yunquera/BBDD/RedPinsapo/ADMON_PUNTO.csv")
Arbol.base<-read.csv("C:/Geodirectorio/Yunquera/BBDD/RedPinsapo/ARBOL_BASE.csv")
Evalua.arbol<-read.csv("C:/Geodirectorio/Yunquera/BBDD/RedPinsapo/EVALUA_ARBOL.csv")
```



#### 3.2. Preparación y limpieza de los datos
Cuando se están preparando datos para responder a un objetivo propuesto, se debe de dedicar un tiempo a revisar y construir un conjunto de datos que pueda aportar información coherente y real y que además sea fácilmente comprensible y manejable para los procesos posteriores que se necesiten. En esta etapa se dedicará un tiempo a pulir los datos. Es lo que se conoce como limpieza de datos y consiste en modificar o eliminar datos incorrectos, redundantes, incompletos o inconsistentes. Algunos ejemplos muy típicos suelen ser limpiar espacios, tildes o símbolos en cadenas de texto; corregir faltas ortográficas; incoherencias entre singulares y plurales; proporcionar sinónimos a los elementos que se refieren al mismo objeto; cambiar mayúsculas y minúsculas; o buscar valores duplicados por error. Se trata de un paso de vital importancia porque la precisión del análisis posterior dependerá de la calidad de los datos.

Se va a valorar ahora la coherencia de los nombres de los campos en las tablas introducidas.

```{r}
#Nombre de las columnas de la tabla general de administración del punto
names(Admon.pto)
```


```{r}
#Cambiamos el nombre al primer campo
names(Admon.pto)[1]<-"X"
#Comprobación del cambio de nombre
names(Admon.pto)
```


Repetimos la operación para las otras 2 tablas.

```{r}
#Nombre de las columnas de la tabla general de administración del punto
names(Arbol.base)
```

```{r}
#Cambiar nombre al primer campo
names(Arbol.base)[1]<-"ID_ARBOL"

#Comprobación del cambio de nombre
names(Arbol.base)
```

```{r}
#Nombre de las columnas de la tabla general de administración del punto
names(Evalua.arbol)
```

```{r}
#Cambiar nombre al primer campo
names(Evalua.arbol)[1]<-"ID_ARBOL"

#Comprobación del cambio de nombre
names(Evalua.arbol)
```


En la arquitectura relacional de la base de datos, se indica que el campo *PUNTO* es el campo común entre la tabla de *Admon.pto* y la de *Arbol.base*. El campo *ID_ARBOL* es el que relaciona las tablas de *Arbol.base* y la de *Evalua.arbol*.

Se va a valorar la coherencia del campo de *COD_MN*, que se refiere al código del monte, en el caso del ejemplo es *MA-30037-AY*, según la capa *kml* de montes públicos descargada anteriormente.

```{r}
#Códigos de monte introducidos en la capa
levels(as.factor(Admon.pto$COD_MN))
```

```{r}
#Número de puntos muestreados con código similar al del monte
length(grep("^MA-30037",Admon.pto$COD_MNT))
```

Parece que hay un punto al que no se le ha incluido el código del monte al que pertenece. También el monte de trabajo, recibe códigos distintos entre sí y al de la capa de montes públicos: *MA-30037* y *MA-30037-CCAY*. Por tanto, se va a hacer la selección de puntos pertenecientes al monte, basándose en la posición de los mismos mediante sus coordenadas X e Y, y se comprobará si coincide con el número de puntos muestreados que contienen un código parecido al del monte de Pinar de Yunquera.

```{r}
#Activación de la librería necesaria
library(sf)

#Ojo: Si estamos haciendo este ejercicio en una sesión nueva, recordar sentar el directorio y llamar al archivo Pinar.Yunquera
setwd("C:/Geodirectorio/Yunquera")
Montes.Publicos<-st_read("Montes_Publicos_Malaga.kml")
MA.30037.AY<-Montes.Publicos[which(Montes.Publicos$Name=="MA-30037-AY"),1]
Pinar.Yunquera<-MA.30037.AY[1,]

#Transformación de la tabla en un shapefile 
Admon.pto.sp <- st_as_sf(x=Admon.pto,coords=c("X","Y"), crs=32630)

#Proyección de la capa del monte al mismo crs que los puntos
Pinar.Yunquera<-st_transform(Pinar.Yunquera,
                             crs=st_crs(Admon.pto.sp))

#Eliminación de datos en la coordenada z
Pinar.Yunquera<-st_zm(Pinar.Yunquera,drop=TRUE)

#Representación de ambas capas
library(mapview)

mapview(Admon.pto.sp,zcol="PUNTO")+(Pinar.Yunquera) 
```

![image](https://user-images.githubusercontent.com/100314590/163583867-b00ed575-b343-43dd-97cb-142fd0900568.png)


Se cuentan 12 puntos dentro de los límites del monte. Ha debido de haber algún fallo en la introducción de los datos, por lo que la selección de los mismos debe ser mediante una función geográfica. Se utilizá *st_within* que genera una operación espacial en la que comprueba la inclusión de una geometría en otra y devuelve *TRUE* o *FALSE*.

```{r}
#Selección geográfica de los puntos 
Ptos.sp.monte<-Admon.pto.sp[which(st_within(Admon.pto.sp,
                                            st_geometry(Pinar.Yunquera),
                                            sparse=FALSE)==TRUE),]
```



Y se vuelven a comprobar los códigos de los montes


```{r}
#Códigos de monte en la capa 
levels(as.factor(Ptos.sp.monte$COD_MN))
```

```{r}
#Localización de los puntos según su código de monte
mapview(Ptos.sp.monte,zcol="COD_MNT")+(Pinar.Yunquera)
```
![image](https://user-images.githubusercontent.com/100314590/163583919-09196d00-920a-4b1b-8210-dd30ec1d2b60.png)



Se comprueba que se trata de un error humano y se puede corregir.

```{r}
#Corrección de códigos de monte 
Ptos.sp.monte$COD_MN<-"MA-30037-AY"

#Comprobación
levels(as.factor(Ptos.sp.monte$COD_MN))
```

Para la revisión de los posibles valores duplicados, se va a utilizar la función *unique()*, que devuelve la misma tabla, pero con las repeticiones eliminadas.

```{r}
#Número de filas de la tabla
nrow(Ptos.sp.monte)
```
```{r}
#Número de filas de la tabla con los valores duplicados eliminados
nrow(unique(Ptos.sp.monte))
```
```{r}
#Número de filas de la tabla
nrow(Arbol.base)
```
```{r}
#Número de filas de la tabla con los valores duplicados eliminados
nrow(unique(Arbol.base))
```
```{r}
#Número de filas de la tabla
nrow(Evalua.arbol)
```
```{r}
#Número de filas de la tabla con los valores duplicados eliminados
nrow(unique(Evalua.arbol))
```

Parece no haber problema de duplicidad de registros. Se puede pasar a la siguiente fase.

#### 3.3. Análisis de los datos

Durante este paso se perfilan los datos para buscar e identificar patrones, detectar valores atípicos y así encontrar relaciones interesantes o patrones. Aquí es donde entra en juego el *Data Mining* que consiste en el conjunto de técnicas y tecnologías que pertinen identificar patrones que expliquen el comportamiento de los datos con la intención de comprenderlos mejor.

Se va a realizar, por tanto, el análisis del estado sanitario del monte. Para ello, primero se van a unir las tablas siguiendo la relación entre los campos.

```{r}
#Unir tablas Ptos.sp.monte y Arbol.base por el campo PUNTO
Arbol.base.r.2<-merge(Ptos.sp.monte,Arbol.base,by="PUNTO",all.x=TRUE)

#Unir tablas Arbol.base.r.2 y Evalua.arbol por el campo ID_ARBOL
Evalua.arbol.r.2<-merge(Arbol.base.r.2,Evalua.arbol,by="ID_ARBOL",
                        all.x=TRUE)
```



Para saber cuántos árboles han sido clasificados en cada tipo de defoliación, se va a utilizar la función *table()*, que genera tablas de contingencia con las suma de valores de cada combinación de factores que se utilice.

```{r}
#Cuenta de cada nivel de defoliación por punto
table(Evalua.arbol.r.2$DEFO,Evalua.arbol.r.2$PUNTO)
```


Puede comprobarse que, aunque la mayoría de los árboles de los puntos de muestreo presentan unos porcentajes de defoliación suaves, los puntos *SN0230*, *SN0213* y *SN0221* son los que contienen más árboles muertos o con defoliación del 100%. Se puede plantear la pregunta si sus valores medios se ven sensiblemente afectados.

```{r}
#Valor medio de defoliación por el campo PUNTO
tapply(Evalua.arbol.r.2$DEFO, Evalua.arbol.r.2$PUNTO, mean,na.rm=TRUE)
```

Se puede confirmar la sospecha de que los puntos *SN0213* y *SN0221* son los que presentan valores defoliativos medios de los árboles más altos, mientras que la parcela *SN0230* no se encuentra entre las de mayor media.

Aunque se ha obtenido una idea general de lo que está ocurriendo fitosanitariamente en la zona, sería más útil extraer conclusiones temporales aprovechando la repetitividad del inventario.

```{r}
#Cuenta de cada nivel de defoliación por punto y campaña
table(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$CAMP==2001)],
      Evalua.arbol.r.2$PUNTO[which(Evalua.arbol.r.2$CAMP==2001)])
```



A veces se utiliza una simplificación de la realidad en la que se agrupan los valores de una variable en clases que sean significativas para el analista. Por ejemplo, es una práctica bastante común en el seguimiento de la sanidad forestal simplificar la defoliación a clases usando las tablas de *Ferreti, 1994*.

![image](https://user-images.githubusercontent.com/100314590/163583995-42658cac-7ef7-4ed2-996f-52c51fa000ff.png)

```{r}
#Sin defoliación o defoliación ligera
defo.1<-aggregate(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$DEFO<25)]~
            Evalua.arbol.r.2$PUNTO[which(Evalua.arbol.r.2$DEFO<25)]+
            Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$DEFO<25)],
          FUN = length)
names(defo.1)<-c("PUNTO","CAMP","CUENTA")
defo.1$DEFO<-"Ligera"

#Defoliación moderada
defo.2<-aggregate(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$DEFO>=25&Evalua.arbol.r.2$DEFO<60)]~
                    Evalua.arbol.r.2$PUNTO[which(Evalua.arbol.r.2$DEFO>=25&Evalua.arbol.r.2$DEFO<60)]+
                    Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$DEFO>=25&Evalua.arbol.r.2$DEFO<60)],
                  FUN = length)
names(defo.2)<-c("PUNTO","CAMP","CUENTA")
defo.2$DEFO<-"Moderada"

#Defoliación severa
defo.3<-aggregate(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$DEFO>=60&Evalua.arbol.r.2$DEFO<100)]~
                    Evalua.arbol.r.2$PUNTO[which(Evalua.arbol.r.2$DEFO>=60&Evalua.arbol.r.2$DEFO<100)]+
                    Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$DEFO>=60&Evalua.arbol.r.2$DEFO<100)],
                  FUN = length)
names(defo.3)<-c("PUNTO","CAMP","CUENTA")
defo.3$DEFO<-"Severa"

#Muertos en pie
defo.4<-aggregate(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$DEFO==100)]~
                    Evalua.arbol.r.2$PUNTO[which(Evalua.arbol.r.2$DEFO==100)]+
                    Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$DEFO==100)],
                  FUN = length)
names(defo.4)<-c("PUNTO","CAMP","CUENTA")
defo.4$DEFO<-"Muertos"
```



Y finalmente, las unimos en una única tabla,

```{r}
defo<-rbind(defo.1,defo.2)
defo<-rbind(defo,defo.3)
defo<-rbind(defo,defo.4)
```


Por otro lado, puede resultar interesante que los datos de clases de defoliación contengan la situación geográfica de los puntos para una representación espacial de los mismos.

```{r}
Ptos.sp.monte.defo<-merge(Ptos.sp.monte,defo,by="PUNTO",all.x=TRUE)
```



#### 3.4. Visualización de los datos

El siguiente paso consiste en crear visualizaciones explicativas de los análisis efectuados con los gráficos más adecuados.

Primero se va a realizar una comparativa a través de los cuartiles de los datos de cada parcela y durante la serie temporal, lo que se conocen como diagramas de caja y bigotes o *boxplot*.

```{r}
#Guardar las características originales de las representaciones gráficas
opar<-par()

#Cambiar las características de representación para que en un sólo gráfico se puedan incluir los 12 puntos en 4 filas de 3 columnas
par(mfrow=c(4,3),cex=0.45)

#Boxplot del punto SN0201
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0201")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0201")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0201")

#Boxplot del punto SN0203
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0203")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0203")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0203")

#Boxplot del punto SN0204
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0204")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0204")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0204")

#Boxplot del punto SN0205
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0205")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0205")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0205")

#Boxplot del punto SN0206
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0206")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0206")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0206")

#Boxplot del punto SN0210
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0210")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0210")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0210")

#Boxplot del punto SN0211
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0211")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0211")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0211")

#Boxplot del punto SN0212
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0212")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0212")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0212")

#Boxplot del punto SN0213
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0213")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0213")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0213")

#Boxplot del punto SN0221
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0221")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0221")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0221")

#Boxplot del punto SN0222
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0222")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0222")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0222")

#Boxplot del punto SN0230
boxplot(Evalua.arbol.r.2$DEFO[which(Evalua.arbol.r.2$PUNTO=="SN0230")]~Evalua.arbol.r.2$CAMP[which(Evalua.arbol.r.2$PUNTO=="SN0230")],
        xlab="Año",ylab="% Defoliación",ylim=c(0,100),main="Punto SN0230")



```
![image](https://user-images.githubusercontent.com/100314590/163584035-6eba7899-ff33-437c-b8ab-4a4c3d1bd242.png)


```{r}
#Recuperación de las características originales de las representaciones gráficas
par(opar)
```

Con este gráfico se consigue interpretar que hay parcelas (*SN201, SN206, SN211, SN2022*) que presentan valores de defoliación concentrados y bajos, frente a otro grupo de puntos donde se da mayor dispersión en la defoliación de la parcela, con numerosos valores atípicos fuera de los cuartiles de referencia (*SN203, SN204, SN205, SN210, SN213, SN221 y SN230*). Asimismo se puede deducir que en algunas de las ubicaciones del monte (*SN213, SN221 y SN230*), hubo un aumento significativo de la defoliación en el año 2006.

No obstante, se podría mejorar la visualización de los datos para que fuera más explicativa, mediante la simplificación a clases de defoliación.

```{r}
#Cambiar las características de representación para que en un sólo gráfico se puedan incluir los 12 puntos en 4 filas de 3 columnas
par(mfrow=c(4,3),cex=0.45)

#Gráfico por clases de defoliación del punto SN0201
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0201")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0201")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0201")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0201")],
     defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0201")],
     ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0201")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0201")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0201")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0201")],
       ylim=c(0,25),pch=19,col="red",type="b")


#Gráfico por clases de defoliación del punto SN0203
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0203")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0203")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0203")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0203")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0203")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0203")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0203")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0203")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0203")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0204
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0204")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0204")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0204")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0204")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0204")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0204")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0204")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0204")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0204")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0205
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0205")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0205")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0205")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0205")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0205")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0205")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0205")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0205")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0205")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0206
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0206")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0206")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0206")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0206")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0206")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0206")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0206")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0206")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0206")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0210
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0210")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0210")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0210")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0210")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0210")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0210")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0210")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0210")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0210")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0211
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0211")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0211")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0211")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0211")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0211")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0211")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0211")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0211")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0211")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0212
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0212")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0212")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0212")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0212")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0212")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0212")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0212")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0212")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0212")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0213
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0213")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0213")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0213")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0213")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0213")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0213")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0213")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0213")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0213")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0221
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0221")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0221")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0221")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0221")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0221")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0221")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0221")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0221")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0221")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0222
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0222")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0222")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0222")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0222")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0222")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0222")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0222")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0222")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0222")],
       ylim=c(0,25),pch=19,col="red",type="b")

#Gráfico por clases de defoliación del punto SN0230
plot(defo$CAMP[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0230")],
     defo$CUENTA[which(defo$DEFO=="Ligera"&defo$PUNTO=="SN0230")],
     ylim=c(0,25),pch=19,col="darkgreen",type="b",
     xlab="Año",ylab="Nº pies",main="Punto SN0230")
points(defo$CAMP[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0230")],
       defo$CUENTA[which(defo$DEFO=="Moderada"&defo$PUNTO=="SN0230")],
       ylim=c(0,25),pch=19,col="chartreuse",type="b")
points(defo$CAMP[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0230")],
       defo$CUENTA[which(defo$DEFO=="Severa"&defo$PUNTO=="SN0230")],
       ylim=c(0,25),pch=19,col="orange",type="b")
points(defo$CAMP[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0230")],
       defo$CUENTA[which(defo$DEFO=="Muertos"&defo$PUNTO=="SN0230")],
       ylim=c(0,25),pch=19,col="red",type="b")
```

![image](https://user-images.githubusercontent.com/100314590/163584073-ae6874d2-8f73-4385-a227-173c4902a67a.png)


```{r}
#Recuperación de las características originales de las representaciones gráficas
par(opar)
```



Ahora resulta más evidente que hay ubicaciones en el monte donde apenas existe defoliaciones altas en el arbolado y otras donde existe mortandad de los pies y defoliaciones severas. También se observa que en la campaña 2006 en algunas parcelas hubo un aumento significativo de los árboles muertos y con defoliación severa a costa de un descenso en los que presentan defoliación ligera. Sin embargo, ahora resulta más reconocible un patrón de compensación entre las clases de defoliación ligera y defoliación moderada dependiendo de la campaña. Podría continuar estudiándose la influencia climatológica de dichos eventos.

#### 3.5. Los datos contando historias…
El conocido como *Storytelling* de los datos, que implica presentar los resultados, no solamente con gráficos bonitos, sino como una narración inteligible por los humanos.

Con los datos multitemporales de los gráficos anteriores se puede componer una imagen geográfica espacial que refleje lo que ocurre en cada año.

```{r}
#Activación de librería 
library(plotrix)
```

```{r}
#Rampas de color para cada grupo
pal.ligera <- colorRampPalette(c("darkolivegreen1", "darkgreen"))
pal.moderada <- colorRampPalette(c("chartreuse", "chartreuse4"))
pal.severa <- colorRampPalette(c("yellow", "darkorange4"))
pal.muertos <- colorRampPalette(c("orange", "darkred"))
```


Se crea una función que va a ayudar a incluir una leyenda de color en los mapas según el grupo de defoliación al que pertenezcan los datos.

```{r}
addLegendToSFPlot <- function(values = c(0, 1), labels = c("Low", "High"), 
                              palette = c("blue", "red"), ...){
  
  # Get the axis limits and calculate size
  axisLimits <- par()$usr
  xLength <- axisLimits[2] - axisLimits[1]
  yLength <- axisLimits[4] - axisLimits[3]
  
  # Define the colour palette
  colourPalette <- leaflet::colorNumeric(palette, range(values))
  
  # Add the legend
  plotrix::color.legend(xl=axisLimits[2]-0.1*xLength, xr=axisLimits[2],
                        yb=axisLimits[3], yt=axisLimits[3]+0.1* yLength,
                        legend= labels, rect.col=colourPalette(values), 
                        gradient="y", ...)
}
```



Para cada año se genera un mapa que represente cada grupo defoliativo (defoliación ligera, moderada, severa o muerto) en un mapa según sus coordenadas.

```{r}
#Cambiar las características de representación para que en un sólo gráfico se puedan incluir los 4 tipos de defoliación, cada uno en un mapa
par(mfrow=c(2,2),mar=c(0,0,0,0),cex.lab=0.5,bg=NA,
    oma=c(0,0,2,0),res=1000)
 
#Gráfico clases de defoliación ligera
plot(st_geometry(Pinar.Yunquera),axes=TRUE)
plot(Ptos.sp.monte.defo[which(Ptos.sp.monte.defo$DEFO=="Ligera"&
                                Ptos.sp.monte.defo$CAMP==2001),
                        13],col=pal.ligera(24),pch=19,cex=2,add=TRUE)
addLegendToSFPlot(values = seq(1,24,1), 
                  labels = c(1,12,24),
                  palette = c("darkolivegreen1", "darkgreen"),cex=0.5)

#Corrección de márgenes para el nuevo gráfico
par(mar = c(0,0,0,0))

#Gráfico clases de defoliación moderada
plot(st_geometry(Pinar.Yunquera),axes=TRUE)
plot(Ptos.sp.monte.defo[which(Ptos.sp.monte.defo$DEFO=="Moderada"&
                                Ptos.sp.monte.defo$CAMP==2001),
                        13],col=pal.moderada(24),pch=19,cex=2,add=TRUE)
addLegendToSFPlot(values = seq(1,24,1), 
                  labels = c(1,12,24),
                  palette = c("chartreuse", "chartreuse4"),cex=0.5)

#Corrección de márgenes para el nuevo gráfico
par(mar = c(0,0,0,0))

#Gráfico clases de defoliación severa
plot(st_geometry(Pinar.Yunquera),axes=TRUE)
plot(Ptos.sp.monte.defo[which(Ptos.sp.monte.defo$DEFO=="Severa"&
                                Ptos.sp.monte.defo$CAMP==2001),
                        13],col=pal.severa(24),pch=19,cex=2,add=TRUE)
addLegendToSFPlot(values = seq(1,24,1), 
                  labels = c(1,12,24),
                  palette = c("darkorange", "darkorange4"),cex=0.5)

#Corrección de márgenes para el nuevo gráfico
par(mar = c(0,0,0,0))

#Gráfico clases de defoliación muertos
plot(st_geometry(Pinar.Yunquera),axes=TRUE)
plot(Ptos.sp.monte.defo[which(Ptos.sp.monte.defo$DEFO=="Muertos"&
                                Ptos.sp.monte.defo$CAMP==2001),
                        13],col=pal.muertos(24),pch=19,cex=2,add=TRUE)
addLegendToSFPlot(values = seq(1,24,1), 
                  labels = c(1,12,24),
                  palette = c("brown1", "darkred"),cex=0.5)

mtext(2001,outer=TRUE)
```
![image](https://user-images.githubusercontent.com/100314590/163584107-cdd992ad-95eb-412f-ab36-941eca792070.png)


```{r}
#Recuperación de las características originales de las representaciones gráficas
par(opar)
```


Al unir la sucesión de anualidades se puede analizar lo que ha ocurrido en el monte.
![total](https://user-images.githubusercontent.com/100314590/163584221-8633c857-1dde-4248-9ad5-f00b32a3abae.gif)


Se puede concluir que el número de pies con defoliación ligera son los más estables en el tiempo, siendo más numerosos en la zona sur-este del monte. En contraste, las muertes de los pies parecen ocurrir con mayor frecuencia y número en la misma zona. Los árboles con defoliación severa también parecen ajustarse a dicho patrón espacial en todas las campañas.
