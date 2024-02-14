# Informační dokument k DEP pro PIC32MK
_jedná se o pracovní verzi dokumentu, který má odpovědět na nejčastější dotazy při zpracovávání laboratorních úloh a práci s PIC32MK procesorem_

# Co jsou to „uživatelské funkce“, jejichž prototypy jsou v `platformDEP32mk.h`?
Jedná se o funkce, které jsou definované v knihovně `platformDEPLib.X.a` a usnadňují uživatelovi práci.

Funkce? Místo toho, aby bylo třeba nutné nastavovat výstupní PINy přímo (jako např u _PIC18F87J11_ `PORTHbits.RH1 = 1;`) je možné zavolat funkci `setLedV13(1)` a výstupní PIN se automaticky nastaví.

Takto to funguje i u ostatních uživatelských funkcí např. pro čtení hodnoty z ADC, z Dekodéru, apod.

### Deklarované funkce u kterých je třeba doplnit funcionalitu

Zavádějící můžout být následující prototypy funkcí.

```c
void configApplication(void);
void runApplication(void);
```
Tyto funkce je třeba doplnit o vlastní funkcionalitu.

# Taktovací frekvence aneb co nastavit v simulátoru

Taktovací frekvence čipu je 120 MHz. Tuto hodnotu je třeba nastavit dle přednášek do simulátoru jako `Fcyc`.

Ovšem protože je v kostře programu v souboru `configCPU32mk.h` nastaveno `FPLLODIV = DIV_2`, je nastavena předdělička taktovacího signálu do stystému na `1/2`. Viz registr `SPLLCON` a schéma na straně 166, Figure 9-1 [Datasheetu](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU32/ProductDocuments/DataSheets/PIC32MK_GP_MC_Familly_Datasheet_60001402G.pdf). Tudíž např. Timery pracují s frekvencí 60 MHz.

# Interrupts

Nalezitelné informace např. zde (proc PIC32MX, ale je používán PIC32MK) [https://developerhelp.microchip.com/xwiki/bin/view/products/mcu-mpu/32bit-mcu/PIC32/mx-arch-cpu-overview/exception-mechanism/usage/](https://developerhelp.microchip.com/xwiki/bin/view/products/mcu-mpu/32bit-mcu/PIC32/mx-arch-cpu-overview/exception-mechanism/usage/).

Informace ohledně vektorů pro přerušení jsou v [DS60001402G](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU32/ProductDocuments/DataSheets/PIC32MK_GP_MC_Familly_Datasheet_60001402G.pdf) v tab. 8-3 (stránka 125–133).

## Ukázková definice ISR

```c
void __ISR(_INT_VECTOR_NUMBER, IPLx[SRS|SOFT|AUTO]) isrName(void)
{
    /* Clear the cause of the interrupt (if required) */32bit:mx-code-examples-general-exception-usage
    /* Clear the interrupt flag */
    /* ISR-specific processing */
}        
```

- kde `_INT_VECTOR_NUMBER` je unikátní identifikátor vektoru přerušení z Tab. 8-1, makra jsou přiřazená v odpovídajícím header souboru (pro PIC 32MK1024MCF100 na POSIX systému v `/opt/microchip/xc32/v4.30/pic32mx/include/proc/PIC32MK-MC/p32mk1024mcf100.h`) (podle `IRQ#` z tabulky 8-3)
- kde `IPLx[SRS|SOFT|AUTO]` určuje prioritu a způsob uložení dat v případě přerušení
    - `x` - priorita `0-7`
    - `SRS` - uložení kontextu pomocí shadow registrů
    - `SOFT` - užití softwarových instrukcí pro uložení kontextu na zásobník
    - `AUTO` - kompiler automaticky podle run-time hodnoty SRSctl v registru CP0 určí, zda je vhodnější `SRS` nebo `SOFT` způsob

název funkce `isrName` je volitený

# Periferie

## Timer

_kde hledat dokumentaci k Timers_

- přímo pro periferie Timeru [DS60001105G](https://ww1.microchip.com/downloads/en/DeviceDoc/60001105G.pdf)
- obecná dokumentace pro PIC32MK [DS60001402G](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU32/ProductDocuments/DataSheets/PIC32MK_GP_MC_Familly_Datasheet_60001402G.pdf)

_informace k přerušení_

- [DS60001108H](https://ww1.microchip.com/downloads/en/DeviceDoc/60001108H.pdf)

### Přerušení
- vektory přerušení pro timery např.

```c
 _TIMER_1_VECTOR; // 4
 _TIMER_2_VECTOR; // 9
 _TIMER_3_VECTOR; // 14
 _TIMER_3_VECTOR; // 19
 _TIMER_3_VECTOR; // 24
 _TIMER_6_VECTOR; // 76
 _TIMER_8_VECTOR; // 84
```

### Timer registry a bity

#### TxCON
- configurační registr

##### Bity
- TCS - Timer Clock Source (0 - interní hodiny, 1 - externí)
- TCKPS - Timer Input Clock Prescale Select (bity dle dokumentace)
- TGATE - Timer Gated Time Accumulation Enable (pro akumulování pouze při určité podmínce)
- T32 - 32 bit timer (0 - 16 bit timer, 1 - 32 bit timer, toto nastavení možné pouze u sudých TimerX, Timer2, Timer4 atd. dojde totiž ke spojení dvou timer registrů - např. Timer2 a Timer3)
- ON - enable timer (0 - disable, 1 - enable)

#### TMRx
- counter register
- x 1...5

#### IFS0
- interrupt flag status


#### IPCx
- interrupt priority control
- x 1...5

##### Bity

- TxIP - interrupt priority bit (1..7)
- TxIS - interrupt subpriority bit (0..3)

#### IEC0

- interrupt enable control

##### Bity

- TxIE - interrupt enable bit
- x 1...5

### Ukázkový příklad pro Timer 2 s prioritou č. 3, kdy dochází k vyvolání žádosti o přerušení každou periodu, resp. 500 ms

```c
// Timer initialization function
void initTime2(void) {
    T2CON = 0; // Stop Timer
    T2CONbits.TCS = 0; // Timer Clock Source fclk = PCLK2 = 60 MHz - clk is set like it because of platform settings
    T2CONbits.TCKPS = 0; // 1:1 Prescale value
    T2CONbits.TGATE = 0; // Disable gated time accumulation
    T2CONbits.T32 = 1; // Set 32 bit mode
    TMR2 = 0; // Pass zero in counter register
    PR2 = 30000000 - 1; // Period register - when IRQ happens, fclk = 60 MHz, 500 ms => 30000000
    // For simulator debugging use PR2 = 6000;

    IFS0bits.T2IF = 0; // Interrupt flag status bit
    IPC2bits.T2IP = 3; // Interrupt priority
    IPC2bits.T2IS = 0; // Interrupt subpriority
    IEC0bits.T2IE = 1; // Enable interrupt in timer
}


// Enable timer
void startTimer2(void) {
    T2CONbits.ON = 1; // Enable timer
}

void __ISR(__TIMER_2_VECTOR, IPL3SOFT) timer2Handler(void) {
    is500ms = true;
    IFS0bits.T2IF = 0;
}
```
## Output Compare Unit

_kde hledat dokumentaci ke Compare_

- přímo pro periferie compare [DS61111E](https://ww1.microchip.com/downloads/en/DeviceDoc/61111E.pdf)
- obecná dokumentace pro PIC32MK [DS60001402G](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU32/ProductDocuments/DataSheets/PIC32MK_GP_MC_Familly_Datasheet_60001402G.pdf)

_informace k přerušení_

- [DS60001108H](https://ww1.microchip.com/downloads/en/DeviceDoc/60001108H.pdf)

### Registry a bity

#### OCxCON
- konfigurační registr

##### Bity
- OC32 - 32 bit compare mode (0 - 16 bit, 1 - 32 bit)
- OCM - Output Compare Mode Selection (2 bity, 010 - compare event forces OCx pin low, 001 - compare event forces OCx pin high)

#### OCxR
- compare register

#### IPCx
- interrupt priority

##### Bity
- OC1IP - interrupt priority
- OC1IS - interrupt subpriority

#### IEC0
- interrupt enable control

##### Bity

- OCxIE - interrupt enable

### Přerušení
- vektory přerušení pro compare např.


```c
_OUTPUT_COMPARE_1_VECTOR; // 7
_OUTPUT_COMPARE_2_VECTOR; // 13
_OUTPUT_COMPARE_3_VECTOR; // 17
_OUTPUT_COMPARE_4_VECTOR; // 22
_OUTPUT_COMPARE_5_VECTOR; // 27
_OUTPUT_COMPARE_7_VECTOR; // 83
_OUTPUT_COMPARE_8_VECTOR; // 86

```

### Ukázkový příklad pro mód průběžného času s využitím T2
```c
    void initTimer2Base (void){
    T2CON = 0; // Stop Timer
    T2CONbits.TCS = 0; // Timer Clock Source fclk = PCLK2 = 60 MHz - clk is set like it because of platform settings
    T2CONbits.TCKPS = 0; // 1:1 Prescale value
    T2CONbits.TGATE = 0; // Disable gated time accumulation
    T2CONbits.T32 = 1; // Set 32 bit mode
    TMR2 = 0; // Pass zero in counter register

    IFS0bits.T2IF = 0; // Interrupt flag status bit
    IPC2bits.T2IP = 0; // Interrupt priority
    IPC2bits.T2IS = 0; // Interrupt subpriority
}

void initOutputCompare1 (void) {
    OC1CON = 0; // Disable OC1
    OC1CONbits.OC32 = 1; // Sets 32 bit mode
//  OC1CONbits.OCM = 001; // Compare event forces OCx pin high
    OC1CONbits.OCM = 011; // Compare event forces OCx pin to toggle
    OC1R = 0x00500000; // Initialize compare register
    

    IFS0bits.OC1IF = 0; // Delete flag
    IPC1bits.OC1IP = 7; // Priority
    IPC1bits.OC1IS = 0; // Subpriority

    IEC0bits.OC1IE = 1; // Enable interrupt
    // CFGCONbits.0CACLK = 1; // Probably set in a platform settings, Input Capture modules use an alternative Timer pair as their timebase clock
}
    
    void startTimerAndEnableCompare (void) {
    T2CONbits.ON = 1; // Enable free running timer
    OC1CON.ON = 1; // Enable the compare unit
}
    
    // ISR
    void __ISR(_OUTPUT_COMPARE_1_VECTOR, IPL0SOFT) OC1Handler (void) {
        IFS0bits.OC1IF = 0;
    }
```

## Input Capture

_kde hledat dokumentaci ke capture_ 

- přímo pro periferie capture [DS60001122G](https://ww1.microchip.com/downloads/en/DeviceDoc/60001122G.pdf)
- obecná dokumentace pro PIC32MK [DS60001402G](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU32/ProductDocuments/DataSheets/PIC32MK_GP_MC_Familly_Datasheet_60001402G.pdf)

_informace k přerušení_

- [DS60001108H](https://ww1.microchip.com/downloads/en/DeviceDoc/60001108H.pdf)

### Přerušení
- vektory přerušení pro capture např.
```c
_INPUT_CAPTURE_1_VECTOR; // 6
_INPUT_CAPTURE_2_VECTOR; // 11
_INPUT_CAPTURE_3_VECTOR; // 16
_INPUT_CAPTURE_4_VECTOR; // 21
_INPUT_CAPTURE_5_VECTOR; // 26
```

### Registry a bity

#### ICxCON
- konfigurační registr


## Ukázkový příklad využití Capture jednotky č. 7 a Timerů 6 a 7
_zpracování naměřené doby mezi jednotlivými událostmi není provedeno_

```c
    void initTimer6Base (void){
    T6CON = 0; // Stop Timer
    T6CONbits.TCS = 0; // Timer Clock Source fclk = PCLK2 = 60 MHz - clk is set like it because of platform settings
    T6CONbits.TCKPS = 0; // 1:1 Prescale value
    T6CONbits.TGATE = 0; // Disable gated time accumulation
    T6CONbits.T32 = 1; // Set 32 bit mode
    TMR6 = 0; // Pass zero in counter register
    IFS2bits.T6IF = 0; // Interrupt flag status bit
    IPC19bits.T6IP = 5; // Interrupt priority
    IPC19bits.T6IS = 0; // Interrupt priority
}

void initCapture7 (void) {
    IC7CON = 0; // Disable capture unit
    IC7CONbits.C32 = 1; // Enable 32 bit mode
    // IC7CONbits.ICTMR = 1; // Use higher number timer mode for capture, does not affect timer selection when C32 = 1;, does not affect timer selection when C32 = 1;
    IC7CONbits.ICI = 0; // Interrupt on very capture event
    // CFGCONbits.ICACLK = 1; // Probably set in platform  Input Capture modules use an alternative Timer pair as their timebase clock
    IC7CONbits.ICM = 011; // Input Capture Mode Select - 011 Simple Capture Event - every rising enge, 010 Simple Capture Event - every falling edge, 001 Edge Detect Mode - every edge
    IFS2bits.IC7IF = 0; // Remove flag
    IPC20bits.IC7IP = 6;
    IPC20bits.IC7IS = 0; // Subpriority
    IEC2bits.IC7IE = 1; // Interrupt enable
}

void startTimerAndCapture (void) {
    T6CONbits.ON = 1; // Start Timer
    IC7CONbits.ON = 1; // Enable Capture Unit
}


void __ISR(_INPUT_CAPTURE_7_VECTOR) capture7Handler (void)
{
    IFS2bits.IC7IF = 0; // Remove flag
}
```


# Changelog

v0.0.1 - vypsání prvních informací a příkladů používaných periferií, odpovězění na základní otázky
