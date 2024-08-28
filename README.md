# Robo Seguidor de Linha

## Equipe

[Guilherme Halter Nunes](https://github.com/GuilhermeHalter) <br>
[João Vitor Bagatoli](https://github.com/joaobagatoli07) <br>
[Wedley Silva Schmoeller](https://github.com/WedleySilva)

## Explicação do código

### Definições e Configurações:
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

**4. Variáveis do PID:**
```ino
int Ki = 0.05;
int Kp = 0.3;
int Kd = 0.3;
int I = 0, P = 0, D = 0, PID = 0;
int velesq = 0, veldir = 0;
int erro, erro_anterior = 0;
```
Define as variáveis e constantes para o controle PID (Proporcional, Integral e Derivativo). Esses valores são usados para ajustar a velocidade dos motores para seguir a linha de forma precisa.

### Funções

**1. setup()**
```ino
void setup() {
  Serial.begin(9600);
}
```

Inicializa a comunicação serial com a taxa de 9600 bps, permitindo que você envie dados para o monitor serial.

**2. loop()**
```ino
void loop() {
  followLineMEF();
  calcula_PID();
}
```
A função principal que é executada continuamente. Aqui, o robô segue a linha e calcula o PID.

**3. motorControl()**

```ino
void motorControl(int speedLeft, int speedRight) {
  // Função para controle do driver de motor
  pinMode(PININ1, OUTPUT);
  pinMode(PININ2, OUTPUT);
  pinMode(PININ3, OUTPUT);
  pinMode(PININ4, OUTPUT);
  pinMode(PINENA, OUTPUT);
  pinMode(PINENB, OUTPUT);
  
  if (speedLeft <= 0) {
    speedLeft = -speedLeft;
    digitalWrite(PININ3, HIGH);
    digitalWrite(PININ4, LOW);
  } else {
    digitalWrite(PININ3, LOW);
    digitalWrite(PININ4, HIGH);
  }
  
  if (speedRight < 0) {
    speedRight = -speedRight;
    digitalWrite(PININ1, LOW);
    digitalWrite(PININ2, HIGH);
  } else {
    digitalWrite(PININ1, HIGH);
    digitalWrite(PININ2, LOW);
  }
  analogWrite(PINENA, speedLeft);
  analogWrite(PINENB, speedRight);
}
```
Controla a direção e a velocidade dos motores com base na velocidade esquerda (speedLeft) e direita (speedRight).

**4. motorOption()**

```ino
void motorOption(char option, int speedLeft, int speedRight) {
  switch (option) {
    case '8': // Frente
      motorControl(-speedLeft, speedRight);
      break;
    case '2': // Tras
      motorControl(speedLeft, -speedRight);
      break;
    case '4': // Esqueda
      motorControl(-speedLeft, -speedRight);
      break;
    case '6': // Direita
      motorControl(speedLeft, speedRight);
      break;
    case '0': // Parar
      motorControl(0, 0);
      break;
  }
}
```
Função que define a direção dos motores com base em uma opção ('8' para frente, '2' para trás, '4' para esquerda, '6' para direita e '0' para parar).

**5. motorStop()**
```ino
bool motorStop(long runtime, long currentTime) {
  if (millis() >= (runtime + currentTime)) {
    motorOption('0', 0, 0);
    int cont = 0;
    while (true) {
      cont++;
    }
    return false;
  }
  return true;
}
```
Verifica se o tempo atual excedeu o tempo de execução (runtime). Se sim, para os motores e entra em um loop infinito.

**6. readSensors()**
```ino
void readSensors(bool readSerial, int *sensors) {
  sensors[0] = analogRead(S1);
  sensors[1] = analogRead(S2);
  sensors[2] = analogRead(S3);
  sensors[3] = analogRead(S4);
  sensors[4] = analogRead(S5);
  sensors[5] = analogRead(S6);
  if (readSerial) {
    Serial.print(sensors[0]);
    Serial.print(' ');
    Serial.print(sensors[1]);
    Serial.print(' ');
    Serial.print(sensors[2]);
    Serial.print(' ');
    Serial.print(sensors[3]);
    Serial.print(' ');
    Serial.print(sensors[4]);
    Serial.print(' ');
    Serial.println(sensors[5]);
  }
}
```
Lê os valores dos sensores QTR e, se readSerial for verdadeiro, envia esses valores para o monitor serial.

**7. followLineMEF()**
```ino
void followLineMEF(void) {
  bool flag = true;
  long currentTime = millis();
  
  while (flag) {
    flag = motorStop(RUNTIME, currentTime);
    int s[6];
    readSensors(false, s);
    
    if (s[0] <= TRESHOLD && s[1] <= TRESHOLD && s[2] <= TRESHOLD && s[3] <= TRESHOLD && s[4] <= TRESHOLD && s[5] <= TRESHOLD) {
      motorOption('8', SPEED0, SPEED0);
      erro=0;
    } else if (s[0] >= TRESHOLD && s[1] <= TRESHOLD && s[2] <= TRESHOLD && s[3] <= TRESHOLD && s[4] <= TRESHOLD && s[5] >= TRESHOLD) {
      motorOption('8', SPEED0, SPEED0);
      erro=0;
    } else if (s[0] >= TRESHOLD && s[1] >= TRESHOLD && s[2] <= TRESHOLD && s[3] <= TRESHOLD && s[4] >= TRESHOLD && s[5] >= TRESHOLD) {
      motorOption('8', SPEED0, SPEED0);
      erro=0;
    } // Outros casos omitidos para brevidade...
  }
}
```
Controla o robô para seguir a linha com base nas leituras dos sensores e ajusta a velocidade dos motores conforme necessário.

**8. calcula_PID()**
```ino
void calcula_PID() {
  if (erro == 0) {
  }
  I = 0;
  P = erro;
  I = I + erro;
  if (I > 255) {
    I = 255;
  } else if (I < -255) {
    I = -255;
  }
  D = erro - erro_anterior;
  PID = (Kp * P) + (Ki * I) + (Kd * D);
  erro_anterior = erro;
}
```
Calcula o valor PID baseado no erro atual, no erro acumulado (Integral) e na diferença do erro (Derivativo). Esse valor é usado para ajustar as velocidades dos motores e melhorar a precisão do controle do robô.
