# Informační dokument k DEP pro PIC32MK - nejčastější otázky
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

