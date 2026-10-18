# tp3-codigo-ledesma-juan
código Arduino control de temperatura

// =====================================
// TP CONTROL DE TEMPERATURA
// VERSION CONTROL PROPORCIONAL
// =====================================

#include <LiquidCrystal.h>

LiquidCrystal lcd(12, 13, 4, 5, 6, 7);

// Variables del sistema
float T_in = 20.0;
float T_ext = 20.0;

int N = 0;

// Potencia climatizador
float P = 0;

// Temperatura deseada
float Tref = 22.0;

// Ganancia proporcional
float Kp = 3.0;
float Ki = 0.02;
float Kd = 1.0;

// Error de control
float error;
float integral = 0;
float derivada = 0;

float errorAnterior = 0;

// Parámetros del modelo
float alpha = 0.05;
float beta = 0.1;
float gamma = 0.001;

// Pulsadores
int lastAdd = HIGH;
int lastSub = HIGH;

void setup()
{
  Serial.begin(9600);

  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);

  pinMode(9, OUTPUT);
  pinMode(10, OUTPUT);
  pinMode(11, OUTPUT);

  lcd.begin(16, 2);

  lcd.clear();

  lcd.setCursor(0,0);
  lcd.print("Iniciando...");
  delay(1000);

  lcd.clear();
}

void actualizarTemperatura()
{
  float dT;

  dT =
      alpha * (T_ext - T_in)
    + beta  * P
    + gamma * N;

  T_in += dT;
}

void loop()
{
  // ======================
  // Leer temperatura exterior
  // ======================

  int lectura = analogRead(A0);

  T_ext = map(lectura, 0, 1023, 0, 40);

  // ======================
  // Leer pulsadores
  // ======================

  int add = digitalRead(2);
  int sub = digitalRead(3);

  if(add == LOW && lastAdd == HIGH)
  {
    N++;
  }

  if(sub == LOW && lastSub == HIGH)
  {
    if(N > 0)
    {
      N--;
    }
  }

  lastAdd = add;
  lastSub = sub;

  // ======================
  // CONTROL PROPORCIONAL
  // ======================

error = Tref - T_in;

// Integral
integral += error;

if(integral > 100)
  integral = 100;

if(integral < -100)
  integral = -100;

// Derivada
derivada = error - errorAnterior;

// PID
P =
    Kp * error
  + Ki * integral
  + Kd * derivada;

// Guardar error anterior
errorAnterior = error;

  // Limitar potencia

  if(P > 100)
  {
    P = 100;
  }

  if(P < -100)
  {
    P = -100;
  }

// ======================
// INDICADOR CLIMATIZADOR
// ======================

if(P > 5)
{
  // Rojo = calefacción

  digitalWrite(9, HIGH);
  digitalWrite(10, LOW);
  digitalWrite(11, LOW);
}
else if(P < -5)
{
  // Azul = refrigeración

  digitalWrite(9, LOW);
  digitalWrite(10, LOW);
  digitalWrite(11, HIGH);
}
else
{
  // Verde = zona confortable

  digitalWrite(9, LOW);
  digitalWrite(10, HIGH);
  digitalWrite(11, LOW);
}

  // ======================
  // Actualizar habitación
  // ======================

  actualizarTemperatura();

  // ======================
  // Mostrar datos
  // ======================

  Serial.print("Tin: ");
  Serial.print(T_in);

  Serial.print("  Text: ");
  Serial.print(T_ext);

  Serial.print("  N: ");
  Serial.print(N);

  Serial.print("  P: ");
  Serial.print(P);

  Serial.print("  Error: ");
  Serial.println(error);

  delay(500);

  Serial.print("  COLOR: ");

if(P > 5)
{
  Serial.println("ROJO");
}
else if(P < -5)
{
  Serial.println("AZUL");
}
else
{
  Serial.println("VERDE");
}

// ======================
// LCD
// ======================

lcd.setCursor(0,0);

lcd.print("Ti:");
lcd.print(T_in,1);

lcd.print(" N:");
lcd.print(N);
lcd.print("   ");

lcd.setCursor(0,1);

lcd.print("Te:");
lcd.print((int)T_ext);

lcd.print(" P:");
lcd.print((int)P);
lcd.print("   ");

}
