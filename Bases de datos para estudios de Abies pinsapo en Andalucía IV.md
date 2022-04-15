# Bases de datos para estudios de Abies pinsapo en Andalucia. Parte IV.

Jessica Bernal
15/4/2022



### 4. Crecimiento de las especies

Como es esperable, la información obtenida de parcelas medidas en un momento dado no incluye datos de la evolución de las variables dendrométricas y dasométricas de las masas forestales, por lo que con ella no es posible utilizar determinadas técnicas de ajuste estadístico para datos de crecimiento. A partir de los datos de un único inventario sólo es posible la elaboración de modelos estáticos, como son tablas de producción de selvicultura media observada, que reflejan únicamente un número limitado de evoluciones de la densidad, o los diagramas de manejo de la densidad. La realización de un segundo inventario permite disponer de datos reales de crecimiento, lo que posibilita el desarrollo de modelos dinámicos, más realistas que los estáticos.

El Inventario Forestal Nacional (IFN) podría definirse como un proyecto encaminado a obtener el máximo de información posible sobre la situación, régimen de propiedad y protección, naturaleza, estado legal, probable evolución y capacidad productora de todo tipo de bienes de los montes españoles. Este inventario caracteriza los tipos de montes en España, cuantificando los recursos forestales disponibles, y presentando datos de densidades, existencias, crecimientos, etc., así como facilitando otros parámetros que describen los bosques, las superficies desarboladas en España y su biodiversidad, todo ello con una metodología y características comunes para todo el territorio. El inventario proporciona una información estadística homogénea y adecuada sobre el estado y la evolución de los ecosistemas forestales españoles que sirve, entre otros, como instrumento para la coordinación de las políticas forestales y de conservación de la naturaleza. La unidad básica de trabajo es la provincia y, al ser un inventario continuo, se repiten las mismas mediciones cada 10 años, recorriéndose todo el territorio nacional en cada ciclo decenal.[Inventario Forestal Nacional - IFN](https://www.miteco.gob.es/es/biodiversidad/temas/inventarios-nacionales/inventario-forestal-nacional/default.aspx)

#### 4.1. Recopilación de los datos
##### 4.1.1. IFN2
El segundo ciclo del Inventario Forestal Nacional (IFN2) se inició en 1986 y acabó en 1996. Los datos presentados contienen toda la información disponible del IFN2 digitalizada para la correspondiente provincia y se presenta en dos formas, cartográfica y alfanumérica.

La primera en un formato tipo sistema de información geográfica (SIG), concretamente el *.E00 de intercambio de ArcInfo y corresponde a los estratos, los tipos de propiedad y a las parcelas de campo.

La información alfanumérica está separada en dos grupos: tablas de la publicación y ficheros del proceso de datos. El primero contiene los mismos cuadros de letras y cifras que el libro en soporte papel publicado. En cambio, Los ficheros del proceso de datos se componen de la información presente en los estadillos de las parcelas de campo, de los resultados intermedios del proceso no publicados y de los estadísticos de los parámetros de los árboles medidos, especialmente interesantes para los análisis dendrométricos y dasométricos.

Todos los datos se encuentran disponibles a través de la web del Ministerio para la Transición Ecológica y el Reto Demográfico en este [enlace](https://www.miteco.gob.es/es/biodiversidad/servicios/banco-datos-naturaleza/informacion-disponible/ifn2_descargas.aspx).


![IFN2](https://user-images.githubusercontent.com/100314590/163585388-97dec24b-85cf-4133-a742-443551d4936d.PNG)

##### 4.1.2. IFN3

La información del tercer ciclo del Inventario Forestal Nacional (IFN3) fue tomada entre los años 1997 y 2007. Se encuentra disponible en ficheros MDB de Access comprimidos en formato ZIP a través de la web del Ministerio para la Transición Ecológica y el Reto Demográfico en este [enlace](https://www.miteco.gob.es/es/biodiversidad/servicios/banco-datos-naturaleza/informacion-disponible/ifn3_base_datos_26_50.aspx). 

![IFN3](https://user-images.githubusercontent.com/100314590/163585401-af321374-0368-4aaa-88c6-360fc6890570.PNG)

#### 4.2. Preparación y limpieza de los datos

```{r}
#Ojo: Si estamos haciendo este ejercicio en una sesión nueva, recordar sentar el directorio y llamar al archivo Pinar.Yunquera
setwd("C:/Geodirectorio/Yunquera")
library(sf)
Montes.Publicos<-st_read("Montes_Publicos_Malaga.kml")
MA.30037.AY<-Montes.Publicos[which(Montes.Publicos$Name=="MA-30037-AY"),1]
Pinar.Yunquera<-MA.30037.AY[1,]
```


##### 4.2.1. IFN3
Una vez descargadas y descomprimidas los ficheros de bases de datos SIG y Campo de la provincia de Málaga es posible abrirlos empleando la librería odbc, cuyo objetivo es proporcionar una interfaz para los controladores de Open Database Connectivity (ODBC) y también la librería DBI que permite una definición de interfaz de la base de datos para la comunicación entre R y los sistemas de gestión de bases de datos relacionales.

```{r}
#Librería ODBC que permite la conectividad ODBC con Bases de Datos
#install.packages("odbc")
library(odbc)
# Librería DBI que permite comunicación con BD relacional
# install.packages("DBI")
library(DBI)
```

```{r}
#Listar drivers
odbc::odbcListDrivers()
```
![image](https://user-images.githubusercontent.com/100314590/163585513-5b5a6be8-6d00-4418-8f47-35f622e1ff50.png)


Si no aparece “Microsoft Access dBASE Driver” se deberá instalar el [driver de DB Access](https://www.microsoft.com/en-us/download/details.aspx?id=54920)

Para comenzar, se introduce la ruta a la base de datos de campo y se establece la conexión:

```{r}
#Ruta a la Base de datos de Access
#IFN3 = "C:/Geodirectorio/Yunquera/IFN3/Ifn3p29.mdb"
#Conexión a BD Access. Indicamos encoding
#con <- dbConnect(odbc::odbc(), 
#                 .connection_string = paste0("Driver={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=",IFN3,";"),
#                 encoding = "latin1")
```

En caso de obtener el error:

*"Error: nanodbc/nanodbc.cpp:1021: HY000: [Microsoft][ODBC Microsoft Access Driver]General error Unable to open registry key Temporary (volatile) Ace DSN for process 0x50b0 Thread 0x4f70 DBC 0x362ad78 Jet'. [Microsoft][Administrador de controladores ODBC] Error de SQLSetConnectAttr del controlador [Microsoft][ODBC Microsoft Access Driver]General error Unable to open registry key Temporary (volatile) Ace DSN for process 0x50b0 Thread 0x4f70 DBC 0x362ad78 Jet'. [Microsoft][ODBC Microsoft Access Driver] Cannot open a database created with a previous version of your application."*

deberemos reparar el archivo y hacer una conversión del mismo a un formato que pueda ser abierto con versiones más recientes de Access. En este caso, se ha reparado el archivo con *Stellar repair for Access* y se ha convertido a *.accdb* utilizando la versión de Microsoft Access 2010. 

```{r}
#Ruta a la Base de datos de Access
IFN3 = "C:/Geodirectorio/Yunquera/IFN3/Ifn3p29.accdb"
#Conexión a BD Access. Indicamos encoding
con <- dbConnect(odbc::odbc(), 
                 .connection_string = paste0("Driver={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=",IFN3,";"),
                 encoding = "latin1")
```


Se ha abierto, por tanto, un canal hacía la base de datos al que se le ha llamado *con*. Se trabaja con ella como un objeto que contiene toda la información para hacer una conexión usando ODBC, incluyendo el tipo de conexión, la dirección de la máquina donde está la base de datos, y el nombre de la base de datos.

Finalmente, se imprime en pantalla el listado de tablas presentes en la base de datos.

```{r}
dbListTables(con) 
```

Todas las bases de datos de Microsoft Access contienen varias tablas de “sistema” (todas las tablas cuyo nombre comienza por *Msys*) que se utilizan para codificar metadatos sobre la base de datos, como relaciones de clave primaria/clave externa (*MsysRelationships*) y tiempos de creación y actualización de consultas (*MsysObjects*). De forma predeterminada, el usuario no puede acceder a estas tablas, pero un usuario avanzado puede ejecutar consultas de alto nivel en ellas.

Este apartado se centrará, por el contrario, en utilizar las tablas referentes a la información forestal. En la página web del Ministerio para la Transición Ecológica y el Reto Demográfico se puede encontrar [documentación relativa para la descripción de las tablas y códigos utilizados en esta base de datos](https://www.miteco.gob.es/es/biodiversidad/servicios/banco-datos-naturaleza/documentador_bdcampo_ifn3_tcm30-282240.pdf).

Se comienza leyendo los campos de la tabla de parcelas (*PCParcelas*)

```{r}
#Leer tabla
PCParcelas <- dbReadTable(con, "PCParcelas")
```

```{r}
#Ver datos de las 2 primeras filas
head(PCParcelas,2)
```
![image](https://user-images.githubusercontent.com/100314590/163585615-7fd7f544-f926-4955-be4b-ecafa8832b58.png)


```{r}
#Convertimos la matriz en un data frame para poder trabajar mejor con ella en R
PCParcelas<-as.data.frame(PCParcelas)
names(PCParcelas)
```


Se va a intentar la representación cartográfica de las parcelas empleadas en el inventario. Los campos “CoorX” y “Coory” representan las coordenadas X e Y en el sistema de referencia local que se empleaba cuando se generaron los datos, el sistema European Datum 1950 UTM zona 30 Norte. Sin embargo, como se verá ahora, estos campos presentan errores que debemos subsanar para poder realizar el ploteado.

```{r}
summary(PCParcelas$CoorX)
```

```{r}
summary(PCParcelas$Coory)
```
```{r}
PCParcelas<-PCParcelas[which(PCParcelas$CoorX>0),]
PCParcelas<-PCParcelas[which(PCParcelas$CoorX<500000),]
PCParcelas<-PCParcelas[which(PCParcelas$Coory>4000000),]
```


Hay numerosos registros en los que el valor de la coordenada X o Y es 0. También se dan errores humanos de introducción de los datos, como uno de los registros de las coordenadas X por encima del parámetro de falso este de 500.000 de los valores de definición de la proyección o el registro con valor de 4092.2 en las coordenadas de las Y, en lugar de valores por encima de 4.000.000, que es donde se sitúan las latitudes en las que se encuentra el área de estudio. Al eliminarlos se puede visualizar la localización de las parcelas. Para ello, se empleará la librería *sf*, que proporciona una forma estandarizada de codificar datos vectoriales.

```{r}
#Activar la librería sf necesaria para datos espaciales
# install.packages("sf")
library(sf)
```

```{r}
#Convertir data frame a SpatialPointsDataFrame
IFN3.sp <- st_as_sf(x=PCParcelas,coords=c("CoorX","Coory"), crs=23030)
plot(st_geometry(IFN3.sp), axes=TRUE,main="Parcelas IFN3 en la provincia de Málaga")
```
![image](https://user-images.githubusercontent.com/100314590/163585762-88f01d2b-ed87-4c45-a727-36be8a61c495.png)

El siguiente paso corresponde a la selección de las parcelas encontradas en el interior del monte Pinar de Yunquera. Para ello, ambas capas deben compartir el mismo sistema de referencia. Una vez realizada la transformación del sistema de coordenadas, ya es posible la representación conjunta de los datos.

```{r}
#Convertir los datos a coordenadas geograficas en wgs84
IFN3.sp.WGS84.geograf<-st_transform(IFN3.sp,4326)

#Convertir los datos a coordenadas cartograficas en wgs84
IFN3.sp.WGS84.UTM30N<-st_transform(IFN3.sp.WGS84.geograf,crs=st_crs(32630))

plot(st_geometry(IFN3.sp.WGS84.UTM30N),axes=TRUE,main="Parcelas IFN3 en la provincia de Málaga")
plot(st_geometry(Pinar.Yunquera), border="red", add=TRUE)
```
![image](https://user-images.githubusercontent.com/100314590/163585894-d7d04b10-38ea-4d55-b6a9-cd384302fa06.png)

Del total de las parcelas, se seleccionarán las que se localizan en el interior de la zona de estudio. Para ello, se realizará una intersección geográfica entre ambas capas.

```{r}
#Selección de parcelas
IFN3.monte<-IFN3.sp.WGS84.UTM30N[which(st_within(IFN3.sp.WGS84.UTM30N,
                                            st_geometry(Pinar.Yunquera),
                                            sparse=FALSE)==TRUE),]
length(unique(IFN3.monte$Estadillo))
```

27 son las parcelas del IFN en el monte Pinar de Yunquera.

Abrimos ahora la tabla de Pies Mayores.

```{r}
#Leer tabla de pies mayores
PCMayores <- dbReadTable(con, "PCMayores")

#Nombre de los campos de la tabla
names(PCMayores)
```

Al tratarse de una base de datos relacional, para cada *Estadillo* o parcela de la tabla *PCParcelas* existen varios elementos u observaciones que corresponden en la tabla *PCMayores*. Ahora deben seleccionarse los pies de las parcelas que están incluidos en el interior del monte.

```{r}
#Unión de tablas por el concepto común Estadillo
Pies.IFN3.monte<-merge(IFN3.monte,PCMayores,by="Estadillo",all.x=TRUE)
```



##### 4.2.2. IFN2

Durante el procesado de los datos en el [IFN2](https://www.miteco.gob.es/es/biodiversidad/servicios/banco-datos-naturaleza/informacion-disponible/ifn2_descargas.aspx), se generaron 5 tablas con los datos recogidos en campo y 3 más con los datos procesados y agrupados.

![IFN2_campo](https://user-images.githubusercontent.com/100314590/163585995-3285061d-359a-4866-abe3-dcaace32eac0.png)

Por otro lado, las tablas descargadas referentes al Inventario Forestal Nacional 2 se sirven en formato *.dbf*. Históricamente, este tipo de archivos supusieron una solución de base de datos muy popular para MS-DOS que más tarde fue llevado a otras plataformas como *Unix* y dio inicio a una serie de productos similares. Básicamente, este formato permite organizar los datos en varios registros con campos con un encabezado con información sobre la estructura de datos y de los registros mismos. Además, es compatible con Windows, Linus y Mac.

Para el desarrollo del ejercicio, se van a emplear la tabla de pies mayores y la de valores resumidos por estadillo.

```{r}
#Librería que permite la lectura de archivos .dbf
library(foreign)

#Leer tabla de pies mayores
Pies.Mayores.IFN2<-read.dbf("C:/Geodirectorio/Yunquera/IFN2/PIESMA29.dbf")

#Leer tabla de valores agrupados por estadillo
resumen.parcela.IFN2<-read.dbf("C:/Geodirectorio/Yunquera/IFN2/IIFL03BD.dbf")
```



Para poder trabajar con datos temporales pertenecientes a ambos inventarios, es necesario homogeneizar los nombres y tipos de los campo. Así, por ejemplo, los nombres de los campos de las tablas del IFN2 están en mayúscula, mientras que las del IFN3 están en minúscula. Es necesario unificar criterios para poder ejecutar uniones de tablas. Sin embargo, es necesario tener en cuenta que los campos que se hacen referencia a la geometría espacial de los puntos, deben permanecer en su estado actual para que puedan ser identificados como tales.

```{r}
#Nombres de los campos de la tabla del IFN2
names(Pies.Mayores.IFN2)
```

```{r}
#Nombres de los campos de la tabla del IFN3
names(Pies.IFN3.monte)
```

```{r}
#Conversión a mayúsculas
names(Pies.IFN3.monte)<-toupper(names(Pies.IFN3.monte))

#Recuperación del nombre del campo de geometría
names(Pies.IFN3.monte)[86]<-"geometry"
```


Uno de los campos comunes que sirve para identificar cada pie en cada una de las mediciones temporales es el número de orden que se empleó en la evaluación del árbol en cada inventario. Por eso, es necesario usar una denominación común en ambas tablas.

```{r}
#Cambiar nombre del campo que marca el orden del número de pie en el IFN2
names(Pies.Mayores.IFN2)[3]<-"ORDENIF2"
```


Además, los campos comunes entre ambas tablas deben pertenecer a la misma clase para poder relacionarlas, es decir, si en una tabla contiene valores numéricos en la otra también deben ser numéricos.

```{r}
#Comprobación de la clase del objeto en la tabla IFN2
is.numeric(Pies.Mayores.IFN2$ESTADILLO)
```
```{r}
#Comprobación de la clase del objeto en la tabla IFN3
is.numeric(Pies.IFN3.monte$ESTADILLO)
```

```{r}
#Conversión en valor numérico
Pies.IFN3.monte$ESTADILLO<-as.numeric(as.character(Pies.IFN3.monte$ESTADILLO))
```


```{r}
#Comprobación de la clase del objeto en la tabla IFN2
is.numeric(Pies.IFN3.monte$ORDENIF2)
```

```{r}
#Comprobación de la clase del objeto en la tabla IFN3
is.numeric(Pies.Mayores.IFN2$ORDENIF2)
```
```{r}
#Conversión en valor numérico
Pies.IFN3.monte$ORDENIF2<-as.numeric(as.character(Pies.IFN3.monte$ORDENIF2))
```


Los campos que van a servir para ejecutar operaciones, por ejemplo, la media entre los 2 diámetros normales o la altura de los pies, es necesario que sean de la clase numérica.

```{r}
#Comprobación de la clase del objeto DIAMETRO1
is.numeric(Pies.Mayores.IFN2$DIAMETRO1)
```

```{r}
#Comprobación de la clase del objeto DIAMETRO2
is.numeric(Pies.Mayores.IFN2$DIAMETRO2)
```
```{r}
#Conversión en valor numérico
Pies.Mayores.IFN2$DN1.IFN2<-as.numeric(as.character(Pies.Mayores.IFN2$DIAMETRO1))

#Conversión en valor numérico
Pies.Mayores.IFN2$DN2.IFN2<-as.numeric(as.character(Pies.Mayores.IFN2$DIAMETRO2))

#Comprobación de la clase del objeto ALTURA
is.numeric(Pies.Mayores.IFN2$ALTURA)
```
```{r}
#Conversión en valor numérico
Pies.Mayores.IFN2$HT.IFN2<-as.numeric(as.character(Pies.Mayores.IFN2$ALTURA))
```


Por otro lado, de nada sirve para los cálculos que falten los datos de cualquiera de los 2 años. Algunos de los árboles inventariados, al ser visitados de nuevo, habían muerto o habían sido tumbados, por lo que no se incorporaron en el IFN3. Puesto que se trata de trabajar con valores multitemporales, se van a seleccionar los pies en los que se ejecutaron mediciones repetidas en uno y otro inventario.

```{r}
#Selección de pies comunes entre ambos inventarios
Pies.IFN3.monte.com<-Pies.IFN3.monte[which(Pies.IFN3.monte$ORDENIF2!=0),]
```


Finalmente, se unen ambas tablas.

```{r}
#Unión de tablas del IFN2 y IFN3
Pies.monte.IFN<-merge(Pies.IFN3.monte.com,Pies.Mayores.IFN2,
                      by=c("ESTADILLO","ORDENIF2"))
```


Por otro lado, para considerar la densidad en la caracterización de los crecimientos, se hace uso de la tabla de valores agrupados por estadillo.

```{r}
resumen.parcela<-aggregate(resumen.parcela.IFN2$NARBOLES,
                           by=list(resumen.parcela.IFN2$CESTADILLO),FUN=sum,
                           na.rm=TRUE)

#Primeros 6 valores de la tabla
head(resumen.parcela)
```
![image](https://user-images.githubusercontent.com/100314590/163586067-c64b3234-38ec-4c3f-ba83-bff99d8c4493.png)

```{r}
#Nombre de los campos de la tabla
names(resumen.parcela)
```

```{r}
#Cambio de nombre de los campos de la tabla
names(resumen.parcela)<-c("ESTADILLO","Npies_parc")

#Comprobación de la clase del objeto ESTADILLO
is.numeric(resumen.parcela$ESTADILLO)
```


#### 3.3. Análisis de los datos

En el Inventario Forestal Nacional, el diámetro normal se mide cuidadosamente a 1.30 m del suelo, con una forcípula graduada para apreciar el milímetro, en dos direcciones perpendiculares, de tal manera que, en la primera de ellas, el eje del aparato esté alineado con el centro de la parcela. Para el análisis de datos, necesitamos el valor medio de ambas mediciones.

```{r}
#Cálculo de valor medio del diámetro normal en IFN2
Pies.monte.IFN$DN.IF2<-(Pies.monte.IFN$DN1.IFN2+Pies.monte.IFN$DN2.IFN2)/2

#Cálculo de valor medio del diámetro normal en IFN3
Pies.monte.IFN$DN.IFN3<-(Pies.monte.IFN$DN1+Pies.monte.IFN$DN2)/2
```

```{r}
#Diferencias de diámetros entre los 2 tiempos
Pies.monte.IFN$Dif.DN<-Pies.monte.IFN$DN.IFN3-Pies.monte.IFN$DN.IF2

#Diferencias de alturas entre los 2 tiempos
Pies.monte.IFN$Dif.H<-Pies.monte.IFN$HT-Pies.monte.IFN$HT.IFN2
```

```{r}
#Eliminación de errores de medición
Pies.monte.IFN<-Pies.monte.IFN[which(Pies.monte.IFN$Dif.DN>0&
                                       Pies.monte.IFN$Dif.H>0),]
```



Como en todos los ejercicios, se va a evaluar más concretamente la especie de pinsapo, que corresponde a la de código 32.

```{r}
#Pies de pinsapo
pinsapo<-Pies.monte.IFN[which(Pies.monte.IFN$ESPECIE.x=="032"),]

#Parcelas de pinsapo
parcelas.pinsapo<-unique(pinsapo[,c('ESTADILLO', 'geometry')])
```


Será interesante conocer cómo se comportan los crecimientos en altura y en diámetro normal según las densidades de la parcela y ver qué patrón espacial representa el resultado. Por eso, se agregan los valores medios por parcela y se le añaden los valores del resumen del total de densidad de pies de cada parcela y la geometría de las parcelas.

```{r}
#Valores medios de crecimientos por ESTADILLO
resumen.pinsapo<-aggregate(cbind(Dif.DN,Dif.H)~ESTADILLO,
                     data=pinsapo,FUN=mean,na.rm=TRUE)

#Añadir valores de densidad de las parcelas
resumen.pinsapo<-merge(resumen.pinsapo,resumen.parcela,by="ESTADILLO")

#Añadir geometría
resumen.pinsapo<-merge(resumen.pinsapo,parcelas.pinsapo,by="ESTADILLO")

#Conversión de la tabla en datos geográficos
resumen.pinsapo<-st_as_sf(resumen.pinsapo)
```


Se puede explorar la asociación de interdependencia que cabría pensar que pudiera existir entre los crecimientos en altura y en diámetro, con la densidad de la parcela.

Para conocer el método más adecuado para calcular el coeficiente de correlación entre las variables, primero es necesario explorar si tienen una distribución normal.

```{r}
#Histograma de los datos de altura
hist(resumen.pinsapo$Dif.H)

```
![image](https://user-images.githubusercontent.com/100314590/163586134-28207de8-bded-4176-8916-d127ffd4b8a5.png)

```{r}
#Histograma de los datos de dap
hist(resumen.pinsapo$Dif.DN)
```
![image](https://user-images.githubusercontent.com/100314590/163586258-580c0153-acab-4aa3-9cf2-ae8091813fc7.png)

```{r}
#Histograma de los datos de dap
hist(resumen.pinsapo$Npies_parc)
```
![image](https://user-images.githubusercontent.com/100314590/163586322-1d6fc3d8-8f22-4b08-8e08-c0362bcc2eb6.png)


Ninguna de las variables sigue una distribución normal, por lo que el método empleado será el de *Spearman*.

```{r}

#Test de correlación entre las alturas y las densidades
correlacion.alturas<-cor.test(resumen.pinsapo$Dif.H,
                              resumen.pinsapo$Npies_parc,
                              method="spearman")
```

```{r}
#Correlación entre las alturas y las densidades
correlacion.alturas$estimate
```

```{r}
#Significancia de la correlación entre las alturas y las densidades
correlacion.alturas$p.value
```

```{r}
#Test de correlación entre los diámetros y las densidades
correlacion.diametros<-cor.test(resumen.pinsapo$Dif.DN,
                                resumen.pinsapo$Npies_parc,
                                method="spearman")
```
```{r}
#Correlación entre los diámetros y las densidades
correlacion.diametros$estimate
```

```{r}
#Significancia de la correlación entre las alturas y las densidades
correlacion.diametros$p.value
```


Ninguna de las correlaciones con la densidad parece tener significancia estadística. Sin embargo, este bajo valor es probable que se deba a la muestra tan pequeña de parcelas en la que se ha empleado. Sería necesario aumentar la superficie de estudio y, por tanto, el número de parcelas, para contrastar los resultados, ya que la asociación de interdependencia esperada responde a una hipótesis plausible y lógica contrastada en el mundo forestal.

#### 3.4. Visualización de los datos

Se puede entender la dinámica general del crecimiento en las especies forestales del monte, a través de un gráfico donde se representen los segmentos de crecimiento de cada uno de los pies.

```{r}
plot(0,0,col = "white",
     xlim=c(80,900),ylim=c(0,21),main="Crecimientos.Todas las especies",
     xlab="dap (mm)", ylab="altura (m)")
segments(Pies.monte.IFN$DN.IFN3,Pies.monte.IFN$HT,
         Pies.monte.IFN$DN.IF2,Pies.monte.IFN$HT.IFN2,
         col=rgb(0,0,0,0.35))

```
![image](https://user-images.githubusercontent.com/100314590/163586429-1c946be6-52b6-4bc2-a62b-445acba4f9a7.png)


Si se particulariza a nivel especie, se pueden intuir dinámicas distintas.

```{r}
plot(0,0,col = "white",
     xlim=c(80,900),ylim=c(0,21),main="Crecimientos.Todas las especies",
     xlab="dap (mm)", ylab="altura (m)")
segments(Pies.monte.IFN$DN.IFN3[which(Pies.monte.IFN$ESPECIE.x=="021")],
         Pies.monte.IFN$HT[which(Pies.monte.IFN$ESPECIE.x=="021")],
         Pies.monte.IFN$DN.IF2[which(Pies.monte.IFN$ESPECIE.x=="021")],
         Pies.monte.IFN$HT.IFN2[which(Pies.monte.IFN$ESPECIE.x=="021")],
         col="red")
segments(Pies.monte.IFN$DN.IFN3[which(Pies.monte.IFN$ESPECIE.x=="024")],
         Pies.monte.IFN$HT[which(Pies.monte.IFN$ESPECIE.x=="024")],
         Pies.monte.IFN$DN.IF2[which(Pies.monte.IFN$ESPECIE.x=="024")],
         Pies.monte.IFN$HT.IFN2[which(Pies.monte.IFN$ESPECIE.x=="024")],
         col="green")
segments(Pies.monte.IFN$DN.IFN3[which(Pies.monte.IFN$ESPECIE.x=="026")],
         Pies.monte.IFN$HT[which(Pies.monte.IFN$ESPECIE.x=="026")],
         Pies.monte.IFN$DN.IF2[which(Pies.monte.IFN$ESPECIE.x=="026")],
         Pies.monte.IFN$HT.IFN2[which(Pies.monte.IFN$ESPECIE.x=="026")],
         col="black")
segments(Pies.monte.IFN$DN.IFN3[which(Pies.monte.IFN$ESPECIE.x=="032")],
         Pies.monte.IFN$HT[which(Pies.monte.IFN$ESPECIE.x=="032")],
         Pies.monte.IFN$DN.IF2[which(Pies.monte.IFN$ESPECIE.x=="032")],
         Pies.monte.IFN$HT.IFN2[which(Pies.monte.IFN$ESPECIE.x=="032")],
         col="blue")
legend("bottomright",legend=c("Pinus sylvestris","Pinus pinea",
                              "Pinus pinaster","Abies pinsapo"),
       col=c("red","green","black","blue"),lty=c(1,1,1,1),cex=0.75,
       box.lty=0)
```
![image](https://user-images.githubusercontent.com/100314590/163586482-e6552145-a26a-480e-9d78-2c14866f3159.png)

La especie que parece mostrar mayor variabilidad de comportamiento en su crecimiento es el pinsapo. Por eso, se va a particularizar para dicha especie.

```{r}
plot(0,0,col = "white",
     xlim=c(80,900),ylim=c(0,21),main="Crecimientos.Pinsapo",
     xlab="dap (mm)", ylab="altura (m)")
segments(pinsapo$DN.IFN3,pinsapo$HT,
         pinsapo$DN.IF2,pinsapo$HT.IFN2,
         col=rgb(0,0,0,0.35))
```
![image](https://user-images.githubusercontent.com/100314590/163586531-d0653c8e-7e45-4881-bf58-b3627dd349bc.png)




#### 3.5. Los datos contando historias

Espacialmente, se puede visualizar dónde han ocurrido los mayores cambios tanto en diámetros:

```{r}
opar<-par()

#Gráfico de la geometría del monte
plot(st_geometry(Pinar.Yunquera),axes=TRUE)

#Localización de las parcelas de pinsapo según su crecimiento en dap
plot(resumen.pinsapo["Dif.DN"], breaks=c(0,30,60,90),
     pal = sf.colors(3),
     pch=19,add=TRUE)

#Leyenda
legend("bottomright",
       legend=c("0-30","30-60","60-100"),
       col=rev(sf.colors(3)),
       lty=1,cex=0.85, box.lty=0)
```
![image](https://user-images.githubusercontent.com/100314590/163586589-71319d88-1341-42e8-adbc-fe74f9e30703.png)

```{r}
par(opar)
```


Como en alturas:

```{r}
#Gráfico de la geometría del monte
plot(st_geometry(Pinar.Yunquera),axes=TRUE)

#Localización de las parcelas de pinsapo según su crecimiento en altura
plot(resumen.pinsapo["Dif.H"], breaks=c(0,1,1.5,2,2.5,3),
     pal = sf.colors(5),
     pch=19,add=TRUE)

#Leyenda
legend("bottomright",
       legend=c("0-1","1-1.5","1.5-2","2-2.5","2.5-3"),
       col=rev(sf.colors(5)),
       lty=1,cex=0.85, box.lty=0)
```

![image](https://user-images.githubusercontent.com/100314590/163586636-87511969-b19f-458b-abd4-51c0dda5c34c.png)

```{r}
par(opar)
```



Parece que, en altura las mayores diferencias suceden en la zona sur y sur-oeste, mientras que es justo ahí, donde menos crecimiento diametral se produce. Es probable que, al estar bajo una mayor presión de densidad y competencia en dicha zona los coeficientes de esbeltez del arbolado hayan aumentado, produciendo una masa frágil y poco estable a eventos de vientos extremos.

```{r}
#Gráfico de la geometría del monte
plot(st_geometry(Pinar.Yunquera),axes=TRUE)

#Localización de las parcelas de pinsapo según su crecimiento en altura
plot(resumen.pinsapo["Npies_parc"], breaks=c(0,200,400,600,800),
     pal = sf.colors(4),
     pch=19,add=TRUE)

#Leyenda
legend("bottomright",
       legend=c("0-200","200-400","400-600","600-800"),
       col=rev(sf.colors(4)),
       lty=1,cex=0.85, box.lty=0)
```
![image](https://user-images.githubusercontent.com/100314590/163586689-427f5217-c176-4249-8040-f9e27b973764.png)

*Nota: Para esta parte no he podido generar el html de R Markdown por el siguiente error arrojado en el knitting. Queda pues pendiente (si encuentras la solución a dicho error, se agradece la comunicación :)*
![image](https://user-images.githubusercontent.com/100314590/163589398-1c9f4f54-06a0-4926-8237-fada79359bd7.png)
