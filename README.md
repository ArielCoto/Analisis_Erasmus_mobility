# Analisis_Erasmus_mobility
Análisis de la base de datos proporcionada por TidyTuesday Erasmus, España
[![diagram.png](https://i.postimg.cc/HkksjSnC/diagram.png)](https://postimg.cc/w7ndGQDW)

# Otras gráficas que encontraran en el análisis
[![mapa-2.png](https://i.postimg.cc/28SjHcDX/mapa-2.png)](https://postimg.cc/py3trByD)

# 3. 
[![mapa-spain-1.png](https://i.postimg.cc/L5XdVW1t/mapa-spain-1.png)](https://postimg.cc/gXfBzN4j)

# 4. 
[![mapa-spain-2.png](https://i.postimg.cc/RZtkGKND/mapa-spain-2.png)](https://postimg.cc/5XxK2H5B)

# 5. 
[![map1.png](https://i.postimg.cc/d3zWTNmt/map1.png)](https://postimg.cc/4HQvD5Sr)

# Código en R
---
title: "TidyTuesday"
output: github_document
author: "Ariel Coto Tapia"
---
Esta semana del reto TidyTuesday 2022-03-08 vamos a analizar la base de datos "Erasmus student mobility".La base de datos proviene de Data.Europa.
\\
Basicamente, la base de datos es sobre los estudiantes que se benefician de un programa de intercambio llamado Erasmus, que permite a los estudiantes estudiar en universidades miembros de la UE (Unión Europea) por periodos de tiempo establecidos. 
Primero, todos las paqueterias que requerrí para el análisis y sus respectivas librerias.

```{r Load}
library(ggiraph)
library(mapview)
library(tigris)
library(mapview)
library(tigris)
library(glue)
library(dplyr)
library(ggplot2)
library(readxl)
library(stringr)
library(colorspace)
library(sf)
library(tidycensus)
library(patchwork)
library(raster)
library(xlsx)
library(rnaturalearthhires)
library(tidytuesdayR)
library(tidyverse)
library(scales)
library(janitor)
library(RColorBrewer)
library(rworldmap)
library(ggplot2)
library(psych)
library(tmap)
library(rnaturalearth)
library(rnaturalearthdata)
library(sf)
library(cartogram)  
library(maptools)     
library(viridis)      
library(patchwork)
library(showtext
```
Entonces, observemos la base de datos. Como describimos en un principio, la base contiene todos los datos de un intercambio academico, desde país de emisión , país de recepción (con sus respectivas ciudades), duración del intercambio, ciclo academico del intercambio, número de participantes, organizaciones involucradas, perfil del estudiante, etcetera.


```{r}
tuesdata <- tidytuesdayR::tt_load('2022-03-08')
erasmus<-tuesdata$erasmus
attach(erasmus)
```


Luego, empezare a analizar el país de origen de los benificiarios de la beca Erasmus:




```{r}
nombres_paises<-read_delim("https://raw.githubusercontent.com/BjnNowak/TidyTuesday/main/data/iso.csv",delim=';')
# TABLA CON EL PAIS DE ORIGEN DE LOS BENEFICIARIOS DE LA BECA #########################
tabla1<-erasmus %>%
  group_by(sending_country_code,academic_year)%>% #Asocio el país de origen, con el ciclo academico porque es algo que va pegado
  summarize(alumnos_enviados=sum(na.omit(participants))) %>% #Ahora opero sobre la misma tabla el numero de participantes enviados ese mismo año
  left_join(nombres_paises,by=c("sending_country_code"="code"))
  
view(tabla1)
```
Notemos que se envian determinado número de participantes por país en cada ciclo y por supuesto que, no es la misma cantidad de estudiantes para cada país. No es de mi importancia analizar ciclo por cilo, sino que juntare un solo periodo que será de 2014-2020 para todos los paises y así, analizar quienes han enviado más alumnos totales por país.



```{r}
# intentare simplificar una tabla para hacer mas visibles los datos 2014-2020
tabla2<- tabla1 %>%
  summarize(total_alumnos=sum(alumnos_enviados))%>%
  left_join(nombres_paises,by=c("sending_country_code"="code"))
```

Ya teniendo los alumnos totales, como son bastantes paises opté por una opción de mapa, por lo que combinare mi marco de datos, con los datos de mapdata para hacer una grafica del mundo.

```{r}
names(tabla2)[names(tabla2)=="country_name"]<-"region"
fix(tabla2) # del dato "Great Britain" en "region" cambiar por "UK" "IMPORTANTE HACERLO EL LECTOR PARA QUE SALGA INGLATERRA DIBUJADO"!!!
mapdata_original<-map_data("world")
mapdata<-left_join(mapdata_original,tabla2,by="region")
#Notamos que hay demasiados puntos blancos en el mapa... crearemos un nuevo marco de datos
#que solo contengan mis datos disponibles...
#filtramos los que no esten vacios
mapa_datos1<-mapdata %>%
  filter(!is.na(mapdata$total_alumnos))
## Procedere a dibujar el mapa
map1<-ggplot(mapa_datos1, aes(x=long,y=lat,group=group))+
  geom_polygon(aes(fill=total_alumnos),color="black")
map2<- map1+scale_fill_gradient(name="Total de alumnos",
                                low="lemonchiffon",high = "indianred",na.value = "grey50")+
  ggtitle("Emisión de alumnos por Erasmus",
          subtitle = "Periodo 2014-2020")
map2
```


Claramente se observa que los países de Europa central son los más beneficiados por el programa Erasmus , sobre todo Alemania, Polonia, Francia, Inglaterra , España. Tal como observaremos en la siguiente tabla:
```{r}
Paises_mas_enviados<-tabla2[order(tabla2$total_alumnos,decreasing = TRUE),]
head(Paises_mas_enviados)
Paises_menos_enviados<-tabla2[order(tabla2$total_alumnos),]
head(Paises_menos_enviados)
```

Concluimos que  la región con más alumnos beneficiados fue Alemania y Polonia y la región con menos alumnos beneficiados fue Togo y Algeria junto con Palestina. \\

Ahora realizaremos el estudio de los países más populares para un intercambio academico de Erasmus.

```{r}
tablaA<- erasmus %>%
  
  group_by(receiving_country_code,academic_year) %>%
  summarize(alumnos_recibidos=sum(na.omit(participants))) %>%
  left_join(nombres_paises,by=c("receiving_country_code"="code"))
tablaB<- tablaA %>%
  summarize(total_alumnos_recibidos=sum(alumnos_recibidos)) %>%
  left_join(nombres_paises,by=c("receiving_country_code"="code"))
names(tablaB)[names(tablaB)=="country_name"]<-"region"
fix(tablaB)#!!!!!!!!!!! En región  cambiar Gran Bretaña por-> "UK"!!! OJO LECTOR para que salga dibujado UK
mapdata_A<-left_join(mapdata_original,tablaB,by="region")
mapa_datos_A<-mapdata_A %>%
  filter(!is.na(mapdata_A$total_alumnos))
map_3<-ggplot(mapa_datos_A, aes(x=long, y=lat, group=group))+
  geom_polygon(aes(fill=total_alumnos_recibidos),color="black")
map_3_grafico<-map_3+scale_fill_gradient(name="Alumnos por movilidad",
                                         low = "lightskyblue1",high="maroon",na.value = "grey50")+
  ggtitle("Países elegidos para intercambio academico",
          subtitle = "Periodo 2014-2020")
map_3_grafico
```

Se conservan los mismos paises emisores , es decir, los países que más resultan beneficiados con el intercambio Erasmus también son los que más estudiantes reciben. La gran diferencia con respecto a la gráfica pasada es que acá ya no aparecen los países africanos. Confirmamos el análisis con la siguiente tabla: 


```{r}
Paises_mas_populares<-tablaB[order(tablaB$total_alumnos_recibidos,decreasing = TRUE),]
head(Paises_mas_populares)
Paises_menos_populares<- tablaB[order(tablaB$total_alumnos_recibidos,decreasing = FALSE),]
head(Paises_menos_populares)
```



Harémos un análisis especifico en el país de España. Filtraremos nuestros datos para ver, que extranjeros son los que más viajan a españa. (Ojo, solo nos enfocaremos en extranjeros, ya que existen españoles que realizan intercambios dentro de su mismo país)


```{r}
visitantes<-erasmus %>%
  group_by(receiving_city)%>%
  filter(receiving_country_code=="ES")%>%
  filter(sending_country_code!="ES")
tabla_spain<- visitantes %>%
  group_by(sending_country_code,academic_year) %>%
  summarize(alumnos_a_spain=sum(na.omit(participants))) %>%
  left_join(nombres_paises, by=c("sending_country_code"="code"))
#juntando todos los años ...
tabla_spain_2<- tabla_españa1 %>%
  summarize(total_alumnos=sum(na.omit(alumnos_a_españa))) %>%
  left_join(nombres_paises, by=c("sending_country_code"="code"))
Paises_mas_spain<-tabla_spain_2[order(tabla_spain_2$total_alumnos,decreasing = TRUE),]
head(Paises_mas_spain)
```


Identificamos los italianos y portugueses son los que más intercambios estudiantiles realizan a España, posiblemente por el idioma y compartir raiz de lengua romance, en el periodo 2014-2020. \\

Ahora, a qué ciudades llegan los estudiantes? Tienen algúna en particular?

En esta sección.. presenté muchos problemas para adaptar el nombre de las ciudades Españolas ya que los caracteres eran incompatibles con la base de datos para hacer mapas, además de que tuve que agrupar las ciudades en provincias españolas. Por lo que lo hice en Excel. A continuación comparto la liga de github con las bases de datos corregidas.


```{r}
#Aquí solo corregí los nombres de las ciudades y las emparejé con sus provincias
library(readr)
#Aqui usar las bases de datos que puse en el repositorio
ciudades_spain_data <- read_csv("C:/Users/cotot/OneDrive/Desktop/LUZ DE NOCHE/TidyTuesdays/ciudades_spain_data.csv", 
    locale = locale(encoding = "WINDOWS-1252"))
#Aqui voy a corregir la base de datos en excel , porque ya pasé mucho tiempo haciendolo en R y nada más no me sale...
datos_modificados <- read_csv("C:/Users/cotot/OneDrive/Desktop/LUZ DE NOCHE/TidyTuesdays/datos_modificados.csv", 
    locale = locale(encoding = "WINDOWS-1252"))
names(datos_modificados)[names(datos_modificados)=="receiving"]<-"NAME_2"
ciudades_Spain_2<-getData("GADM",country="ESP",level=2) %>%
  st_as_sf()
mapa_spain_data<-left_join(ciudades_Spain_2,datos_modificados,by="NAME_2")
```
Ahora procedo a gráficar las provincias... 
```{r}
tm_shape(mapa_spain_data,projection = 26198) +
  tm_fill(col = "visitantes_spain",
          palette = "Purples",
          title = "Visitantes extranjeros de Erasmus a España",
          subtitle="Periodo 2014-2020")
```


Buscando el mapa ideal .... 

```{r}
tm_shape(mapa_spain_data) + 
  tm_polygons() + 
  tm_bubbles(size = "visitantes_spain", 
             alpha = 0.5, 
             col = "navy",
             title.size = "Estudiantes extranjeros de Erasmus a España.
             Periodo 2014-2020")
```





```{r}
vt_map<-ggplot(mapa_spain_data,aes(fill=visitantes_spain))+
  geom_sf_interactive(aes(data_id=NAME_2))+
  scale_fill_distiller(palette = "Greens",
                       direction = 1,
                       guide = "none")+
  theme_void()
vt_map
vt_plot<-ggplot(datos_modificados,aes(x=visitantes_spain,y=reorder(NAME_2,visitantes_spain),
                                    fill=visitantes_spain))+
  geom_point_interactive(color="black",size=4,shape=21,
                         aes(data_id=NAME_2))+
  scale_fill_distiller(palette = "Greens",direction = 1)+
  labs(title = "¿A donde llegan los estudiantes extranjeros?",
       subtitle = "Periodo 2014-2020",
       y = "",
       x = "",
       fill = "Alumnos Totales") + 
  theme_minimal(base_size = 14)
  
girafe(ggobj = vt_map + vt_plot, width_svg = 10, height_svg = 5) %>%
  girafe_options(opts_hover(css = "fill:cyan;"))
```
