[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/KzqfxGd5)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=22850519&assignment_repo_type=AssignmentRepo)
# Lab02 - Caracterización de osciladores (externo vs. interno)


## 1. Integrantes
* [Alexis Castillo Ariza](https://github.com/alexiscastillo-04) 
* [Nicolas Torres Pinzon](https://github.com/Nicolas0807)
## 2. Documentación

### 2.1 Descripción del laboratorio

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
| INTOSC (interno) | 16,000,000       | Sí                 |                     |                500                 |               |               | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |               |               |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |               |               | |

#### Tabla 2: Medición con calor

| Modo de oscilador | Freq. teórica Fosc | RA6 medible (CLKO)? | Freq. medida RA6 (Hz) | Freq. teórica RC0 (Hz)| Freq. medida RC0 (Hz) | Error RC0 (%) |  
|------------------|------------------|---------------------|---------------|---------------------|---------------|---------------|
| INTOSC (interno) | 16,000,000       | Sí                 |                     |                500                 |               |               | |
| HS (cristal externo 16 MHz) | 16,000,000 | No |     NA      |               500                 |               |               |
| RC externo       | ~16,000,000*     | No                                    |       N/A        | 500                 |               |               | |

#### Tabla 3: Deriva

| Modo de oscilador |RC0 deriva (Hz) |
|------------------|--------------------|
| INTOSC (interno) |                    |                
| HS (cristal externo 16 MHz) |                |                |
| RC externo       |                 |                


<!-- Agregar tablas para valores usando PLL -->

<!-- Complemente con análisis de lo registrado en tablas -->

## 2.4 Diagramas

## 2.5 Formas de onda

### INTOSC (interno) 


### HS

## RC

## 3. Evidencias de implementación

## 4. Preguntas

* ¿En qué modo se obtuvo la medición más cercana a la frecuencia teórica?

* ¿Fue posible evidenciar el fenómeno de deriva? ¿Qué factores podrían explicar la variación de frecuencia al calentar el PIC?

* ¿Cuál es más preciso en cuanto a frecuencia teórica vs. medida?


* Explique cómo usar RC0 para estimar la frecuencia del oscilador cuando RA6 no está disponible.

* Si se quisiera duplicar la frecuencia del PIC usando PLL, ¿en qué modos se podría aplicar?

* Enliste ventajas y desventajas de cada modo.

## 5. Referencias