## INFORMACIÓN ENTREGA N°1
El presente documento entrega la información pertinente en relación a la primera entrega del proyecto ***Software Defined Radio***. Es necesario destacar que en esta primera etapa se presenta un modelo básico del proyecto ya mencionado, con el propósito de establecer a grandes rasgos el funcionamiento del sistema, en ningún caso se trata de un modelo fijo, sino más bien de un protocolo que simule una parte del funcionamiento real del sistema, que además, sentará las bases para su posterior modificación y mejoramiento del software ya construido.
Esto se realizó en base a lo presentando en el primer informe realizado y de acuerdo a la estructuración de la planificación expresada en la carta Gantt.
***
####Funcionamiento del software presentado primera etapa
El software presentando en esta oportunidad  bajo el lenguaje de programación ***Python*** presenta las siguientes funciones y/o características; 
- Simulación de la comunicación entre emisor y receptor de manera predeterminada
- Simulación en la transmisión del mensaje bajo las características del protocolo AX.25
  - Codificación y decodificación del mensaje transmitido
  - Inclusión de ruido en dicha simulación

#####Palabras Claves:
* ***Protocolo AX.2***
* ***Codificación***
* ***Decodificación***
* ***Ruido***

***Protocolo AX.25***
: La estructuración del mensaje sigue la norma internacional del protocolo para radiaficionados amateurs. La estructuración y funcionamiento de dicho protocolo puede ser visitado a través del siguiente documento oficial. [Manual Protocolo AX.25](https://www.tapr.org/pub_ax25.html)

***Ruido***
: Hace referencia a cualquiera manifestación indeseable que opaca la señal legítima y que no se relaciona directamente con ella, por lo que deben existir mecanismos que permitan dilucidar de manera legítima y real la señal emitida en un principio.

***Codificación y Decodificación***
: La codificación y Decodificación son procesos vitales a la hora de transmisión de señales. La codificación consiste en adaptar la información que se desea transmitir en señales que puedan viajar por un medio de transmisión, mientras que la decodificación es el proceso inverso, vale decir, tomar las señales captadas en el receptor y convertirlas en información útil.
***
### Explicación de los mecanismos presentados 
#####Simulación del transmisión del mensaje 
La función de este mecanismo radica en la codificación de un mensaje de texto para su posterior transmisión, junto con la codificación se aplican diversos métodos que cumplan con los estándares del protocolo AX.25 señalado anteriormente.
Para la conversión de texto a binario y viceversa se utilizaron las funciones nativas de python
```python
def text_to_bits(text, encoding='utf-8', errors='surrogatepass'):
    bits = bin(int.from_bytes(text.encode(encoding, errors), 'big'))[2:]
    return bits.zfill(8 * ((len(bits) + 7) // 8))

def text_from_bits(bits, encoding='utf-8', errors='surrogatepass'):
    n = int(bits, 2)
    return n.to_bytes((n.bit_length() + 7) // 8, 'big').decode(encoding, errors) or '\0'
```
Junto con ello además se crea una funcion para convertir un número decimal a Binario, que será esencial para convertir el FCS al formato correspondiente 
```python
def decToBin(numeroDecimal):
    numeroBinario = ''
    while numeroDecimal // 2 != 0:
        numeroBinario = str(numeroDecimal % 2) + numeroBinario
        numeroDecimal = numeroDecimal // 2
    return str(numeroDecimal) + numeroBinario
```
Se crea además la función codificar que presenta como parámetro el mensaje, destino, origen, protocolo, y el check. Que enviará el mensaje con los estándares válidos de acuerdo al protocolo utilizado, como lo es la bandera de entrada y salida (***10000001***), el FCS, entre otras. Que permitirán tener un mayor control sobre la correcta transmisión de la información.
```python
def codificar(mensaje, destino, origen, protocolo, check):
```
El checksum juega un rol fundamental en la transmisión pues toma 4 bits del mensaje y los suma, ese valor se guarda en el espacio correspondiente a FCS (***Frame Check Sequence ***) con formato binario, luego cuando se recibe el mensaje se realiza nuevamente un checksum y se compara con el FCS ya obtenido anteriormente, si son iguales se ha transmitido la información con éxito, de lo contrario se produjo un error en la transmición.

Por último el sistema será capaz de distinguir entre los posibles situaciones que podrían ocurrir, informando al usuario de lo acontecido; 
* El mensaje no viene con el protocolo necesario
* El Checksum no corresponde
* El mensaje viene con ruido
* El mensaje no puede ser leído porque no es para nosotros
* El mensaje viene con menos bits de los necesarios


#####Inclusión del Ruido 
En esta función entra el mensaje previamente ya codificado, con todo lo que esto conlleva de acuerdo a la utilización del protocolo AX.25. El mensaje tiene 2200 bits y la aplicación de ruido varía en los siguiente porcentajes:
+ 50% el mensaje no presenta ruido 
+ 25% de ruido en un bit
+ 25% de ruido en dos bit

> *Estos porcentajes fueron decididos por el grupo de trabajo y en ningún caso representan necesariamente porcentajes reales sobre la estimación de ruido en la transmisión de señales.*

A través de una función ***Random()*** entre 0 y 3 se calcula la presencia o no de ruido;
```python
numeroAleatorio = random.randrange(4)
```
Siendo;
* 0 y 1 para manifestar que no presenta ruido
* 2 para ruido en un bit
* 3 para ruido en dos bits
 
Cuando se aplica el ruido al mensaje, se habla de invertir el bit, vale decir si el bit es 0 se cambia por 1 y viceversa. Cuando el número de la función Random es igual a 2, se aplica ruido en un bit, vale decir, se aplica nuevamente una función ***Random*** entre o y 2200
```python
numeroRuido = random.randrange(2200)
```
Correspondiente al largo del mensaje codificado, dependiendo del resultado, se va en busca de la posición de acuerdo al número obtenido y se cambia el bit por su complemento.

```python
if(bitRuido == '0'):
  mensaje = mensaje[0:numeroRuido] + '1' + mensaje[numeroRuido+1:2200]
else:
 mensaje = mensaje[0:numeroRuido] + '0' + mensaje[numeroRuido+1:2200]
```
Lo mismo ocurre para el ruido en dos bits, salvo que esta vez, se debe realizar la búsqueda en dos posiciones distintas.
```python
while(contador < 2):
  numeroRuido = random.randrange(2200)
  bitRuido = mensaje[numeroRuido:numeroRuido+1]
  if(bitRuido == '0'):
    mensaje = mensaje[0:numeroRuido] + '1' + mensaje[numeroRuido+1:2200]
  else:
    mensaje = mensaje[0:numeroRuido] + '0' + mensaje[numeroRuido+1:2200]
  contador = contador + 1
```
Cabe destacar que en esta oportunidad se hace la inclusión de ruido sobre el mensaje, lo cual el sistema verifica si existe o no la presencia de esta anormalidad en el mensaje, en caso positivo se informa dicho acontecimiento, no obstante, la elimininación del ruido del mensaje es parte de la segunda actividad a realizar y no es objeto de estudio en esta ocasión.
