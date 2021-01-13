# Creando-intervalos-en-RStudio

Muchas veces queremos tenemos un conjuntos de datos en donde agruparlos se nos haria mucho más fácil poder trabajar y dar un mejor análisis e interpretación de lo mismo.

### **¿En qué momento crear intervalo podría ser útil?**

En muchos entornos de análisis de datos, puede resultar útil dividir una variable continua, como la edad, en una variable categórica. O bien, es posible que desee clasificar una variable categórica como año en un contenedor más grande, como 1990-2000. Hay muchas razones para no hacer esto al realizar un análisis de regresión, pero para presentaciones simples de datos demográficos en tablas, podría tener sentido. ¡La función de `cut()` en R simplifica esta tarea!

La función `cut` divide el rango de x en intervalos y codifica los valores en x según el intervalo en el que caen. El intervalo más a la izquierda corresponde al nivel uno, el siguiente más a la izquierda al nivel dos y así sucesivamente.

**Sintaxis**
```{r eval=FALSE}
cut(x, breaks, labels = NULL,
    include.lowest = FALSE, right = TRUE, dig.lab = 3,
    ordered_result = FALSE, ...)
```

[ ]
* **x:** un vector numérico que se va a convertir en un factor mediante el corte.

* **breaks:** ya sea un vector numérico de dos o más puntos de corte únicos o un solo número (mayor o igual a 2) que da el número de intervalos en los que se x va a cortar.

* **labels:** etiquetas para los niveles de la categoría resultante. De forma predeterminada, las etiquetas se construyen utilizando "(a,b]"notación de intervalo. Si `labels = FALSE`, se devuelven códigos enteros simples en lugar de un factor.

* **include.lowest:** lógico, que indica si se `right = FALSE` debe incluir una 
'x[i]' igual al valor más bajo (o más alto, para ) de 'rupturas'.

* **right:** lógico, que indica si los intervalos deben cerrado a la derecha (y abierto a la izquierda) o viceversa.

* **dig.lab:** entero que se usa cuando no se dan etiquetas. Determina la cantidad de dígitos utilizados para formatear los números de pausa.

* **ordered_result:** lógico: ¿el resultado debería ser un factor ordenado?

### **Detalles**

Cuando `breaks` se especifica como un solo número, el rango de los datos se divide en `breaks` partes de igual longitud, y luego los límites externos se alejan en un 0,1% del rango para garantizar que los valores extremos caigan dentro de los intervalos de ruptura. (Si $x$ es un vector constante, se crean intervalos de igual longitud, uno de los cuales incluye el valor único).

Si `labels` se especifica un parámetro, sus valores se utilizan para nombrar los niveles de los factores. Si no se especifica ninguno, las etiquetas de nivel de factor se construyen como `"(b1, b2]", "(b2, b3]"` etc. para `right = TRUE` y como `"[b1, b2)", ...` si `right = FALSE`. En este caso, `dig.lab` indica el número mínimo de dígitos se debe utilizar en el formato de los números `b1, b2, ....` Se utilizará un valor mayor (hasta 12) si es necesario para distinguir entre cualquier par de puntos finales: si esto falla, `"Range3"` se utilizarán etiquetas como . El formateo se realiza mediante formatC.

El método predeterminado clasificará un vector numérico de `breaks`, pero no se requieren otros métodos y `labels` corresponderán a los intervalos después de la clasificación.

### **¿Comó usamos la función cut()?**

Primero, simularemos algunos datos de un ensayo clínico hipotético que incluye variables para la identificación del paciente, la edad y el año de inscripción.

```{r}
library(knitr)
clinica <-
    data.frame(pacientes = 1:100,              
               edad = rnorm(100, mean = 60, sd = 8),
               anyo = sample(paste("19", 85:99, sep = ""),100, replace = TRUE))
kable(head(clinica,10), align = "c")
```
```{r}
summary(clinica)
```

Ahora, usaremos la función `cut` para convertir la edad en un factor, que es lo que llama una variable categórica. Nuestras primeras llamadas de ejemplo se cortan con el argumento break establecido en un solo número. Este método hará que el corte divida la edad en 4 intervalos. Las etiquetas predeterminadas utilizan notación matemática estándar para intervalos abiertos y cerrados.

```{r}
#Intervalos abiertos-cerrado
c1 <- cut(clinica$edad, breaks=4)
c1
```
```{r}
#Intervalo abierto -cerrado
table(c1)
```
También podemos visualizarlo de manera gráfica
```{r}
plot(c1, col=rainbow(5))
```


```{r}
#intervalos abierto -cerrado
c2 <- cut(as.numeric(as.character(clinica$anyo)),breaks=3)
c2
```
```{r}
table(c2)
```
Visualicemos de manera grafica 
```{r}
plot(c2, col=topo.colors(5) )
```


Bueno, los intervalos que el `cut` eligió por defecto no son los más bonitos con el ejemplo de la edad, aunque están bien con el ejemplo del año, ya que ya era discreto. Afortunadamente, podemos especificar los intervalos exactos que queremos para la edad. Nuestro siguiente ejemplo muestra cómo.

```{r}
#Intervalo abierto-cerrado
seq(30, 80, by = 10)
c3 <- cut(clinica$edad, breaks = seq(30, 80, by = 10))
c3
```

```{r}
#Intervalo abierto-cerrado
table(c3)
```

```{r}
#Intervalos cerrado -abierto
table(c3 <- cut(clinica$edad, breaks = seq(30, 80, by = 10), right = F))
```
visualicemos graficamente
```{r}
plot(c3, col=heat.colors(6))
```


Eso se ve bastante bien. No hay ninguna razón por la que el argumento de las rupturas deba estar igualmente espaciado como se hizo anteriormente. Puede ser cualquier agrupación que desee.


Finalmente, mostraremos un ejemplo de una función R personalizada para categorizar edades. Utiliza `cut` dentro de él, pero realiza un preprocesamiento y usa el argumento de `labels` para cortar y hacer que la salida se vea bien.

```{r}
age.cat <- function(x, inferior = 0, superior, by = 10,
                   sep = "-", above.char = "+") {
 labs <- c(paste(seq(inferior, superior - by, by = by),
                 seq(inferior + by - 1, superior - 1, by = by),
                 sep = sep),
           paste(superior, above.char, sep = ""))
 cut(floor(x), breaks = c(seq(inferior, superior, by = by), Inf),
     right = FALSE, labels = labs)
}
```

Esta función categoriza la edad de una manera bastante flexible. La primera asignación a los `labs` dentro de la función crea un vector de etiquetas. Luego, se llama a la función de `cut` para hacer el trabajo, con las etiquetas personalizadas como argumento. Aquí hay algunos ejemplos que utilizan nuestros datos simulados de arriba. Ya no voy a guardar los resultados de las llamadas de función en una variable y llamar a la tabla en ellos, sino que simplemente anidaré la llamada a age.cat en una llamada a la tabla . Anteriormente hice una publicación sobre la función de mesa .

```{r}
a1<-table(age.cat(clinica$edad, superior=70))
a1
```
```{r}
#hist(a1,col=rainbow(8))
plot(a1,col=rainbow(8), lwd = 20)
```


```{r}
a2<-table(age.cat(clinica$edad, inferior = 30, superior =70))
a2
```
```{r}
plot(a2, col=heat.colors(10), lwd = 20)
#hist(a2, col=heat.colors(10))
```


```{r}
a3<-table(age.cat(clinica$edad, inferior = 30, superior = 70, by = 5))
a3
```
visulizando graficamente
```{r}
plot(a3, col=rainbow(10), lwd = 20, main="Intervalos Quinquenales")
```


También podemos visualizar que datos estan perdido y cuales son esos datos

```{r}
c3 <- cut(clinica$edad, breaks = seq(30, 80, by = 10))
which (is.na(c3)) # ¿cuáles datos están perdidos?
c3[is.na(c3)] # ¿a qué valores corresponden los datos perdidos?
```

### **Resumen**

La función `cut` es útil para convertir variables continuas en factores. a como se vio especificar el número de puntos de corte, especificar los puntos de `cut` exactos y una función construida alrededor del `cut`  que simplifica la categorización de una variable de edad y darle las etiquetas adecuadas.
