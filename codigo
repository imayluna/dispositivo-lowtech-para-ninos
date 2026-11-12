#include <WiFi.h>
#include <esp_now.h>

// ====== CAMBIA ESTO: 1 en la Esp1 (Esp-Padre), 2 en la Esp2 (Esp-Hijo) ====== 
(Tienes que saber el mac de tu Esp32, para cambiar el codigo a los mac que estan definidos en Mac de cada placa)
#define ESP_NUMERO 1

// MAC de cada placa
uint8_t mac1[] = {0xD4, 0xE9, 0xF4, 0xB3, 0xAD, 0x0C};  // Esp1
uint8_t mac2[] = {0xB0, 0xCB, 0xD8, 0x9C, 0x35, 0x58};  // Esp2

// Cada placa le habla a la OTRA
#if ESP_NUMERO == 1
  uint8_t *peerMAC = mac2;
#else
  uint8_t *peerMAC = mac1;
#endif

// ====== PINES ======
#define BUZZER_PIN            25   // Esp1: buzzer
#define LED_EMERGENCIA_PIN    26   // Esp1: LED (con resistencia de ~220 ohm a GND)
#define BOTON_APAGAR_PIN      33   // Esp1: boton entre el pin y GND (con resistencia de ~220 ohm a GND)
#define BOTON_EMERGENCIA_PIN  27   // Esp2: boton entre el pin y GND (con resistencia de ~220 ohm a GND)

// ====== AJUSTES (aquí calibras) ======
const unsigned long INTERVALO_ENVIO  = 50;    // ms entre mensajes (20 por segundo)
const unsigned long INTERVALO_PRINT  = 100;   // ms entre impresiones
const unsigned long TIMEOUT_SENAL    = 1500;  // ms sin recibir = fuera de alcance

// Umbrales de distancia (deben cumplir: MUY_CERCA > CERCA > LEJOS > MUY_LEJOS)
// Cada umbral es el límite inferior de su estado:
//   RSSI > UMBRAL_MUY_CERCA                      -> MUY CERCA
//   entre UMBRAL_CERCA y UMBRAL_MUY_CERCA        -> CERCA
//   entre UMBRAL_LEJOS y UMBRAL_CERCA            -> LEJOS
//   entre UMBRAL_MUY_LEJOS y UMBRAL_LEJOS        -> MUY LEJOS
//   RSSI <= UMBRAL_MUY_LEJOS                     -> FUERA DE ALCANCE (señal casi perdida)
// Para que el buzzer suene desde más lejos, baja UMBRAL_CERCA y ajusta los demás debajo de él
const int UMBRAL_MUY_CERCA = -50;
const int UMBRAL_CERCA     = -75;
const int UMBRAL_LEJOS     = -85;
const int UMBRAL_MUY_LEJOS = -95;

// ====== Anti falsas alarmas ======
const unsigned long TIEMPO_CONFIRMACION = 1500; // ms que debe durar "lejos" para sonar
const int HISTERESIS = 5;                       // dB extra de señal para dejar de sonar

// ====== Sonidos del buzzer (alarmas de distancia) ======
// LEJOS: pitido rapido y agudo
const int TONO_LEJOS = 3000;
const unsigned long DURACION_LEJOS = 140;
const unsigned long PERIODO_LEJOS  = 250;

// MUY LEJOS / FUERA DE ALCANCE: pitidos lento y grave
const int TONO_MUY_LEJOS = 1700;
const unsigned long DURACION_MUY_LEJOS = 400;
const unsigned long PERIODO_MUY_LEJOS  = 1000;

// ====== Emergencia ======
const unsigned long DURACION_AVISO   = 3000;  // ms que la Esp2 repite la emergencia en sus paquetes
const unsigned long ANTIRREBOTE_MS   = 50;    // ms que el boton debe estar estable
const int TONO_EMERG_GRAVE           = 1000;  // sirena: tono grave
const int TONO_EMERG_AGUDO           = 1500;  // sirena: tono agudo
const unsigned long PERIODO_SIRENA   = 200;   // ms que dura cada tono de la sirena

typedef struct {
  uint32_t contador;
  uint8_t  emergencia;   // 1 = emergencia activa, 0 = normal
} Mensaje;

uint32_t contador = 0;
unsigned long tUltimoEnvio = 0;
unsigned long tUltimoPrint = 0;
unsigned long tUltimoPitido = 0;

// Estado de la alarma de distancia
bool alarmaActiva = false;
unsigned long tInicioLejos = 0;

// --- RSSI ---
volatile int rssiActual = -100;
float rssiSuave = -100;
volatile unsigned long tUltimoRecibido = 0;
volatile bool hayNuevo = false;
volatile bool hayConexion = false;   // true tras recibir el primer paquete

// --- Emergencia ---
// Esp2 (emisor del aviso)
bool emergenciaEnviando = false;
unsigned long tEmergencia = 0;

// Esp1 (receptor del aviso)
volatile bool emergenciaRecibida = false;  // se levanta cuando llega una NUEVA emergencia
bool emergenciaPrevia = false;             // ultimo valor del campo emergencia recibido
bool alarmaEmergencia = false;             // true = sirena y LED activos hasta apagar
unsigned long tSirena = 0;
bool sirenaAguda = false;

// --- Boton con antirrebote ---
typedef struct {
  uint8_t pin;
  bool estadoEstable;        // HIGH = suelto (INPUT_PULLUP)
  bool ultimaLectura;
  unsigned long tCambio;
} Boton;

Boton botonEmergencia = {BOTON_EMERGENCIA_PIN, HIGH, HIGH, 0};
Boton botonApagar     = {BOTON_APAGAR_PIN,     HIGH, HIGH, 0};

// Devuelve true UNA sola vez por cada pulsacion (cuando pasa de suelto a presionado)
bool botonPresionado(Boton &b) {
  bool lectura = digitalRead(b.pin);

  if (lectura != b.ultimaLectura) {
    b.ultimaLectura = lectura;
    b.tCambio = millis();             // cambio detectado: reinicia la espera
  }

  if (millis() - b.tCambio >= ANTIRREBOTE_MS && lectura != b.estadoEstable) {
    b.estadoEstable = lectura;
    if (b.estadoEstable == LOW) return true;   // flanco: se acaba de presionar
  }
  return false;
}

// Se ejecuta al recibir un mensaje (core ESP32 3.x)
void onRecv(const esp_now_recv_info_t *info, const uint8_t *data, int len) {
  if (len < (int)sizeof(Mensaje)) return;   // paquete de otra version del codigo: se ignora

  Mensaje m;
  memcpy(&m, data, sizeof(m));

  rssiActual = info->rx_ctrl->rssi;
  tUltimoRecibido = millis();
  hayConexion = true;
  hayNuevo = true;

  // Emergencia por flanco: solo avisa cuando el campo pasa de 0 a 1.
  // Asi, si se apaga la alarma mientras la Esp2 sigue repitiendo el aviso, no vuelve a sonar.
  if (m.emergencia && !emergenciaPrevia) emergenciaRecibida = true;
  emergenciaPrevia = m.emergencia;
}

// Filtro adaptativo: estable con ruido normal, rápido ante saltos grandes
void actualizarFiltro() {
  if (rssiSuave <= -100) {
    rssiSuave = rssiActual;   // primera lectura
    return;
  }

  float diferencia = fabs(rssiActual - rssiSuave);
  float peso;

  if (diferencia > 12)      peso = 0.8;    // salto grande: seguir casi directo
  else if (diferencia > 6)  peso = 0.5;    // cambio medio
  else                      peso = 0.15;   // ruido normal: filtrar fuerte

  rssiSuave = (1 - peso) * rssiSuave + peso * rssiActual;
}

const char* estadoDistancia() {
  if (!hayConexion) return "SIN CONEXION";
  if (millis() - tUltimoRecibido > TIMEOUT_SENAL) return "FUERA DE ALCANCE";
  if (rssiSuave > UMBRAL_MUY_CERCA) return "MUY CERCA";
  if (rssiSuave > UMBRAL_CERCA)     return "CERCA";
  if (rssiSuave > UMBRAL_LEJOS)     return "LEJOS";
  if (rssiSuave > UMBRAL_MUY_LEJOS) return "MUY LEJOS";
  return "FUERA DE ALCANCE";   // señal tan débil que casi se pierde
}

// Decide si el buzzer debe sonar por distancia (LEJOS, MUY LEJOS o FUERA DE ALCANCE)
// con confirmación por tiempo e histéresis para evitar falsas alarmas
bool debeSonar() {
  if (!hayConexion) {
    alarmaActiva = false;
    tInicioLejos = 0;
    return false;
  }

  // Señal perdida: suena de inmediato (el timeout ya es una espera de 1,5 s)
  if (millis() - tUltimoRecibido > TIMEOUT_SENAL) {
    alarmaActiva = true;
    return true;
  }

  // Histéresis: para empezar usa el umbral, para dejar de sonar exige recuperar más señal
  bool condicionLejos = alarmaActiva ? (rssiSuave <= UMBRAL_CERCA + HISTERESIS)
                                     : (rssiSuave <= UMBRAL_CERCA);

  if (condicionLejos) {
    if (tInicioLejos == 0) tInicioLejos = millis();
    if (millis() - tInicioLejos >= TIEMPO_CONFIRMACION) alarmaActiva = true;
  } else {
    tInicioLejos = 0;
    alarmaActiva = false;
  }
  return alarmaActiva;
}

// true en MUY LEJOS y FUERA DE ALCANCE (sonido lento y grave)
bool esMuyLejos() {
  if (millis() - tUltimoRecibido > TIMEOUT_SENAL) return true;
  return rssiSuave <= UMBRAL_LEJOS;
}

// Sirena de emergencia: alterna tono grave y agudo, distinta a las alarmas de distancia
void sirenaEmergencia() {
  if (millis() - tSirena >= PERIODO_SIRENA) {
    tSirena = millis();
    sirenaAguda = !sirenaAguda;
    tone(BUZZER_PIN, sirenaAguda ? TONO_EMERG_AGUDO : TONO_EMERG_GRAVE, PERIODO_SIRENA);
  }
}

// Buzzer sin delay: no bloquea la recepción ni el filtro
// La emergencia tiene prioridad sobre las alarmas de distancia
void actualizarBuzzer() {
  if (alarmaEmergencia) {
    sirenaEmergencia();
    return;
  }

  if (!debeSonar()) {
    noTone(BUZZER_PIN);
    return;
  }

  int tono;
  unsigned long duracion, periodo;

  if (esMuyLejos()) {
    tono = TONO_MUY_LEJOS;
    duracion = DURACION_MUY_LEJOS;
    periodo = PERIODO_MUY_LEJOS;
  } else {
    tono = TONO_LEJOS;
    duracion = DURACION_LEJOS;
    periodo = PERIODO_LEJOS;
  }

  if (millis() - tUltimoPitido >= periodo) {
    tUltimoPitido = millis();
    tone(BUZZER_PIN, tono, duracion);
  }
}

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.printf("\n--- Iniciando Esp%d ---\n", ESP_NUMERO);

#if ESP_NUMERO == 1
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_EMERGENCIA_PIN, OUTPUT);
  digitalWrite(LED_EMERGENCIA_PIN, LOW);
  pinMode(BOTON_APAGAR_PIN, INPUT_PULLUP);
#else
  pinMode(BOTON_EMERGENCIA_PIN, INPUT_PULLUP);
#endif

  WiFi.mode(WIFI_STA);
  Serial.print("Mi MAC: ");
  Serial.println(WiFi.macAddress());

  if (esp_now_init() != ESP_OK) {
    Serial.println("Error iniciando ESP-NOW");
    return;
  }

  esp_now_register_recv_cb(onRecv);

  esp_now_peer_info_t peer = {};
  memcpy(peer.peer_addr, peerMAC, 6);
  peer.channel = 0;
  peer.encrypt = false;

  if (esp_now_add_peer(&peer) != ESP_OK) {
    Serial.println("Error agregando la otra placa");
    return;
  }
  Serial.println("Listo, enviando y escuchando...");
}

void loop() {
  // ---- Esp2: boton de emergencia ----
#if ESP_NUMERO == 2
  if (botonPresionado(botonEmergencia)) {
    emergenciaEnviando = true;
    tEmergencia = millis();
    Serial.println(">>> EMERGENCIA enviada");
  }
  if (emergenciaEnviando && millis() - tEmergencia >= DURACION_AVISO) {
    emergenciaEnviando = false;
  }
#endif

  // ---- Enviar ----
  if (millis() - tUltimoEnvio >= INTERVALO_ENVIO) {
    tUltimoEnvio = millis();
    Mensaje m;
    m.contador = contador++;
    m.emergencia = emergenciaEnviando ? 1 : 0;
    esp_now_send(peerMAC, (uint8_t *)&m, sizeof(m));
  }

  // ---- Filtrar cada paquete recibido ----
  if (hayNuevo) {
    hayNuevo = false;
    actualizarFiltro();
  }

  // ---- Esp1: emergencia, LED, boton de apagar y buzzer ----
#if ESP_NUMERO == 1
  if (emergenciaRecibida) {
    emergenciaRecibida = false;
    alarmaEmergencia = true;
    Serial.println("!!! EMERGENCIA recibida");
  }

  if (botonPresionado(botonApagar) && alarmaEmergencia) {
    alarmaEmergencia = false;
    noTone(BUZZER_PIN);
    Serial.println("Alarma de emergencia apagada");
  }

  digitalWrite(LED_EMERGENCIA_PIN, alarmaEmergencia ? HIGH : LOW);
  actualizarBuzzer();
#endif

  // ---- Imprimir a ritmo legible ----
  if (millis() - tUltimoPrint >= INTERVALO_PRINT) {
    tUltimoPrint = millis();
    Serial.printf("RSSI: %d dBm | Suave: %.1f dBm | %s\n",
                  rssiActual, rssiSuave, estadoDistancia());
  }
}
