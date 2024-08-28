# Robo Seguidor de Linha

## Explicação do código


**1. Portas do Driver de Motor:**
```ino
#define PININ1 2
#define PININ2 4
#define PININ3 5
#define PININ4 7
#define PINENA 3
#define PINENB 6
```
Define os pinos do Arduino usados para controlar o driver do motor. Esses pinos são responsáveis por controlar a direção e a velocidade dos motores.

**2. Portas do Sensor QTR:**

```ino
#define S1 A0
#define S2 A1
#define S3 A2
#define S4 A3
#define S5 A4
#define S6 A5
```
Define os pinos analógicos aos quais os sensores QTR (sensores de linha) estão conectados.

**3. Valores de Ajustes para o Seguidor de Linha:**

```ino
#define TRESHOLD 700
#define SPEED0 255
#define SPEED1 220
#define SPEED2 150
#define SPEED3 100
#define SPEED4 80
#define SPEED5 50
#define SPEED6 0
#define SPEED7 200
#define RUNTIME 21500
```

Define os valores de referência para detectar a linha e os valores de velocidade dos motores em diferentes situações. **TRESHOLD** é o valor usado para determinar se o sensor está lendo a linha (preto) ou o fundo (branco). **SPEED0** a **SPEED7** são velocidades que o robô deve usar para diferentes leituras dos sensores. **RUNTIME** é o tempo que o robô deve seguir a linha antes de parar.

