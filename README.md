[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/KzqfxGd5)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=22850519&assignment_repo_type=AssignmentRepo)
# Lab02 - Caracterización de osciladores (externo vs. interno)


## 1. Integrantes
* [Alexis Castillo Ariza](https://github.com/alexiscastillo-04) 
* [Nicolas Torres Pinzon](https://github.com/Nicolas0807)
## 2. Documentación

### 2.1 Descripción del laboratorio
Como lo indica el objetivo principal, este laboratorio consiste en aprender a utilizar los tipos de osicladores que maneja el microcontrolador PIC18F45k22 como lo son: Osilador Interno, Oscilador Externo (Cristal de Cuarzo) y Oscilador RC.

La idea principal es simular cada uno de los montajes establecidos en la guia y determinar una fecuencia unica para cada tipo de oscilador la cual es de 500Hz. 

Como se describe en este laboratorio al realizar el montaje en simulacion por Proteus y en Fisico, se debe utilizar un codigo principal el cual sera explicado mas adelante, el mismo maneja las funciones de acuerdo al tipo de oscilador que se seleccione y no obstante debe ser mostrada su señal de salida en el osciloscopio, tomar fotos de los montajes y guardar la imagen de cada una de las señales dadas con las respectivas frecuencias y medidas dentro de una usb por medio del osciloscopio.

Montaje 1 : Oscilador Interno

Montaje 2: Osicilador Externo

Montaje 3: Oscilador RC

### 2.2 Explicación del código implementado
a continuacion vamos a explicar el codigo en el cual estamos basandonos para realizar el laboratorio, explicamos por partes para que asi sea mas facil visualizar el codigo 
```
// ========================== MODO DE OSCILADOR ==========================
// 1 = INTOSC interno
// 2 = Cristal externo HS
// 3 = RC externo
#define MODE 3  

#if MODE == 1
    #pragma config FOSC = INTIO67   // Oscilador interno
    #define USE_PLL 0
#elif MODE == 2
    #pragma config FOSC = HSHP     // Cristal HS
    #define USE_PLL 0
#elif MODE == 3
    #pragma config FOSC = RC       // RC externo
    #define USE_PLL 0
#else
    #error "Modo de oscilador inválido"
#endif
```
En este primer bloque de código, definimos una llamada variable MODEy le asignamos un número que determina el tipo de oscilador a utilizar. A continuación, empleamos condicionales ( if, elify else) para configurar el oscilador según el valor de MODE:

Si MODE es igual a 1, se programa el oscilador interno ( INTIO67), ideal para aplicaciones que no requieren un cristal externo.

Si MODE es 2, se configura el cristal HS externo ( HSHP), que ofrece mayor precisión y estabilidad.

Si MODE es 3, se selecciona el oscilador RC externo ( RC), útil en diseños con componentes pasivos simples.

Finalmente, el bloque else genera un error con el mensaje "Modo de oscilador inválido" si no se selecciona ninguno de los modos anteriores.

Este enfoque permite seleccionar fácilmente el oscilador mediante el valor de MODE al inicio del código 

```
// ========================== FRECUENCIA DEL OSCILADOR =====================
#if MODE == 1 || MODE == 2
    #if USE_PLL
        #define _XTAL_FREQ 64000000UL // 16 MHz * 4
    #else
        #define _XTAL_FREQ 16000000UL
    #endif
#else
    #define _XTAL_FREQ 200000UL // Ajustar según resistencia + condensador
#endif
```
En el segundo bloque, configuramos la frecuencia del oscilador ( _XTAL_FREQ) según el modo seleccionado y el uso del PLL:

Para los modos 1 (INTOSC interno) y 2 (cristal HS externo), si USE_PLLestá activado, se define una frecuencia de
64
 
megahercio
64megahercio(16 MHz multiplicados por 4). De lo contrario, se usa
16
 
megahercio
16megahercio.

Para el modo 3 (RC externo), se establece una frecuencia base de
200
 
kHz
200kHz, que debe ajustarse manualmente según el valor de la resistencia y condensador utilizados.

Este bloque asegura que el compilador tenga la frecuencia correcta para retrasos y temporizadores

```
// ========================== FUNCIONES ==========================
void delay_ms(uint16_t ms) {
    while(ms--) {
        __delay_ms(1);
    }
}

void init_pins(void) {
    // RC0 salida blinker
    TRISCbits.TRISC0 = 0;
    LATCbits.LATC0 = 0;

    // RA6 salida CLKO solo si modo lo permite
    if(MODE == 1 || (MODE == 2 && USE_PLL)) {
        TRISAbits.TRISA6 = 0;
        LATAbits.LATA6 = 0;
    }
}

void init_oscillator(void) {
#if USE_PLL
    OSCCONbits.SPLLEN = 1;  // habilita PLL
#endif
}
```
En este bloque definimos tres funciones esenciales para el control del microcontrolador:

delay_ms(uint16_t ms)
Crea retardos precisos en milisegundos usando un bucle que invoca la función __delay_ms(1)del compilador el número de veces especificado.

init_pins(void)
Inicializa los pines de E/S:

Configure RC0 como salida para un LED intermitente (inicia en bajo).

Configura RA6 (CLKO) como salida solo si:

MODE == 1(INTOSC), o

MODE == 2(HS) y USE_PLL está activado.

init_oscillator(void)
Habilita el PLL si USE_PLLestá definido, configurando el bit SPLLEN = 1en el registro OSCCON.

Estas funciones preparan el hardware para operación estable

```
// ========================== PROGRAMA PRINCIPAL ==========================
void main(void) {
    init_pins();
    init_oscillator();

    while(1) {
        // RC0 toggle ? 500 Hz
        LATCbits.LATC0 = 1;
        delay_ms(1);
        LATCbits.LATC0 = 0;
        delay_ms(1);
    }
}
```
En la función main(void), el programa principal realiza la inicialización secuencial del hardware y ejecuta un bucle infinito para generar una señal de prueba:

Primero llama a init_pins()para configurar los pines de E/S ya init_oscillator()para estabilizar el reloj. Luego, en el bucle while(1), alterna el pin RC0 (LED intermitente) para generar una frecuencia de 500 Hz :

Enciende RC0 ( LATC0 = 1), espera 1 ms.

Apaga RC0 ( LATC0 = 0), espera 1 ms.

Período total : 2 ms → Frecuencia :
500
 
Hz
500Hz.

Este parpadeo verifica el correcto funcionamiento del oscilador configurado y la precisión de los retrasos


### 2.3 Análisis y comparación

#### Tabla 1: Medición en frío

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |   469.35                  |                500                 |  46             |    6.2 %          | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |       496.1        |       0.8 %        |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |        299.56       |        40 %       | |

#### Tabla 2: Medición con calor

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |    no hay señal                  |                500                 |    32          | 93.6 %               | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |       499.2      |          0.2%     |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |       410.8        |   18 %            | |

#### Tabla 3: Deriva

| Modo de oscilador |RC0 deriva (Hz) |
|------------------|--------------------|
| INTOSC (interno) |     468               |                
| HS (cristal externo 16 MHz) |       3.7         |                |
| RC externo       |       1.6          |                


<!-- Agregar tablas para valores usando PLL -->

<!-- Complemente con análisis de lo registrado en tablas -->

## 2.4 Diagramas
aqui podemos encontrar el diagrama que utilizamos en proteus : 
INTOSC (interno): 

Cristal externo 


RC 


## 2.5 Formas de onda

### INTOSC (interno) 


### HS

## RC

## 3. Evidencias de implementación
INTOSC (interno): 

Cristal externo 


RC 

## 4. Preguntas

* ¿En qué modo se obtuvo la medición más cercana a la frecuencia teórica?

se visualiza que  con el cristal externo se evidencia una frecuencia muy cercana a la esperada  es decir en el modo 2  en la configuracion del codigo 

* ¿Fue posible evidenciar el fenómeno de deriva? ¿Qué factores podrían explicar la variación de frecuencia al calentar el PIC?



* ¿Cuál es más preciso en cuanto a frecuencia teórica vs. medida?

se evidecia que el cristal externo es mas preciso en cuanto a la teoria vs medida  ya que se puede evidenciar en la formas de onda que la frecuencia es muy cercana a lo que se espera 
* Explique cómo usar RC0 para estimar la frecuencia del oscilador cuando RA6 no está disponible.
En el código, el pin RC0 genera una señal de 500 Hz necesaria (período de 2 ms) usando retardos calibrados de 1 ms cada uno. Esta señal sirve como referencia confiable para estimar la frecuencia del oscilador cuando RA6 (CLKO) no está disponible.  la ventaja de esto es que  RC0 funciona en todos los modos (1, 2 y 3), mientras RA6 solo está disponible en modo 1 o (modo 2 + PLL), haciendo de RC0 el método más universal para verificar la frecuencia real del oscilador configurado 

* Si se quisiera duplicar la frecuencia del PIC usando PLL, ¿en qué modos se podría aplicar?
verificando el codigo solo se podria aplicar en el modo 1 y modo 2 ya que en el modo 3 que es el RC usa una frecuencia directa 
* Enliste ventajas y desventajas de cada modo.

Modo 1: INTOSC Interno
Ventajas:

- Sin componentes externos.

- Bajo consumo.

- Rápida inicialización.

Desventajas:

- Menor precisión (±2%).

- Deriva térmica.

Modo 2: Cristal HS Externo
Ventajas:

- Alta precisión (<0.1%).

- Estabilidad excelente.

- Compatible con PLL (64 MHz).

Desventajas:

- Requiere cristal + capacitores.

- Más costoso.

- Inicialización lenta.

Modo 3: RC Externo
Ventajas:

- Muy económico.

- Solo resistencia + capacitor.

- Fácil prototipado.

Desventajas:

- Baja precisión (±10-20%).

- Sin PLL (solo 200 kHz).

- Deriva con temperatura.

## 5. Referencias
1. Universidad Nacional del Nordeste. (s.f.). Descripción General del PIC16F877. Recuperado de: 
https://exa.unne.edu.ar/ingenieria/sysistemas/public_html/Archi_pdf/HojaDatos/Microcontroladores/PIC16F877.pdf
2. Código Electrónica. (2019). Datasheet PIC16F877A. Recuperado de: 
https://codigoelectronica.com/blog/datasheet-pic16f877a
3. Scribd. (s.f.). PIC16F877A: Características y Configuración - Osciladores. Recuperado de: 
https://es.scribd.com/document/465764784/PIC16F877-pdf
