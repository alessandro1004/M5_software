#if defined(ARDUINO)
#define WIFI_SSID     "WKVE-Alessadro-Paloma-2024 2."
#define WIFI_PASSWORD "19091004"
#define NTP_TIMEZONE  "UTC-8"
#define NTP_SERVER1   "0.pool.ntp.org"
#define NTP_SERVER2   "1.pool.ntp.org"
#define NTP_SERVER3   "2.pool.ntp.org"
#include <WiFi.h>
#if __has_include(<esp_sntp.h>)
#include <esp_sntp.h>
#define SNTP_ENABLED 1
#elif __has_include(<sntp.h>)
#include <sntp.h>
#define SNTP_ENABLED 1
#endif
#endif
#ifndef SNTP_ENABLED
#define SNTP_ENABLED 0
#endif
//=============================== M5 stack decalaração =============================================
#include <M5StickCPlus2.h>
#include "M5StickCPlus2.h"
#include <IRremote.hpp> 

#define BUTTON_CHANGE_PIN  39  // Pino para o botão de mudar opções
#define BUTTON_EXECUTE_PIN 35  // Pino para o botão de executar função
#define BUTTON_REBOOT_PIN  37  // Pino para o botão de reboot/desligar
#define BUZZER_SOUND 2 //Pino do Buzzer
#define DISABLE_CODE_FOR_RECEIVER  // Disables restarting receiver after each
                                   // send. Saves 450 bytes program memory and
                                   // 269 bytes RAM if receiving functions are
                                   // not used.
#define SEND_PWM_BY_TIMER
#define IR_TX_PIN 19

#include <RH_ASK.h>
#include <SPI.h>
#define RF433_PIN 26  // Define o pino G26 como entrada do receptor RF433
RH_ASK driver(2000, RF433_PIN, -1); // Frequência 2kHz, pino DATA 26, sem pino TX

uint8_t sCommand = 0x34;
uint8_t sRepeats = 0;

int rotation = 1;
int selected = 0;
int entered = 0;
int menu_selec = 0;
int i = 0;
int j = 0;
int color_letra = YELLOW;
int color_fundo = BLACK;
String selecionado = " Hub ";

//====================================== Bateria ==============================================
int readBatteryPercentage() {
  // Ler a tensão da bateria (em milivolts)
  int batteryVoltage = M5.Power.getBatteryVoltage();
  // Limiar de corte inferior da tensão da bateria (em milivolts)
  int batteryVoltageMin = 3300; // Por exemplo, 3.3V
  // Limiar de corte superior da tensão da bateria (em milivolts)
  int batteryVoltageMax = 4200; // Por exemplo, 4.2V
  // Converter a tensão da bateria em uma porcentagem
  int batteryPercentage = map(batteryVoltage, batteryVoltageMin, batteryVoltageMax, 0, 100);
  batteryPercentage = constrain(batteryPercentage, 0, 100); // Garantir que a porcentagem esteja dentro do intervalo válido
  return batteryPercentage;
};
int BatteryPercentageDisplay() {
  int batteryPercentage = readBatteryPercentage();
  int color_batery = color_letra;
  if (batteryPercentage >= 60) {
    color_batery = color_letra;
  } else if (batteryPercentage < 60 && batteryPercentage > 30) {
    color_batery = YELLOW;
  } else {
    color_batery = RED;
  };
  return color_batery;
};
//====================================== Wifi connect função ==================================
int connectar_Wifi() {
  StickCP2.begin();
  StickCP2.Display.setRotation(1);
  StickCP2.Display.setTextColor(color_letra);
  if (!StickCP2.Rtc.isEnabled()) {
      StickCP2.Display.println("RTC not found.");
      StickCP2.Display.println("RTC not found.");
      for (;;) {
          vTaskDelay(1000);
      }
  }
  StickCP2.Display.println("RTC found.");

  StickCP2.Display.setCursor(5, 10);
  StickCP2.Display.print("WiFi:");
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
      StickCP2.Display.print('.');
      delay(1000);
  }
  StickCP2.Display.println("\r\n WiFi Connected.");
  StickCP2.Display.setCursor(5, 20);
  StickCP2.Display.println(WIFI_SSID);
  StickCP2.Display.setCursor(5, 30);
  StickCP2.Display.println(" Connected.");
  configTzTime(NTP_TIMEZONE, NTP_SERVER1, NTP_SERVER2, NTP_SERVER3);
  #if SNTP_ENABLED
    while (sntp_get_sync_status() != SNTP_SYNC_STATUS_COMPLETED) {
        StickCP2.Display.print('.');
        delay(1000);
    }
  #else
    delay(1600);
    struct tm timeInfo;
    while (!getLocalTime(&timeInfo, 1000)) {
        StickCP2.Display.print('.');
    };  
  #endif
  StickCP2.Display.println("\r\n NTP Connected.");
  time_t t = time(nullptr);  // Advance one second.
  while (t > time(nullptr))
      ;  /// Synchronization in seconds
  StickCP2.Rtc.setDateTime(gmtime(&t));
  StickCP2.Display.clear();
}
void hora_global() {
  M5.Display.setTextSize(0.5);
  // Dias da semana
  static constexpr const char* const wd[7] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
  // Obter o tempo do sistema (tempo Unix)
  auto t = time(nullptr);
  auto tm = localtime(&t);  // Converter para estrutura tm para manipulação mais fácil
  if (tm != nullptr) {  // Verificar se 'tm' foi obtido corretamente
    StickCP2.Display.setCursor(140, 1);
    // Exibir data e hora no formato desejado
    StickCP2.Display.printf(
      "%02d/%02d/%02d",  
      tm->tm_mday - 1 ,       // Dia
      tm->tm_mon + 1,      // Mês
      tm->tm_year + 1900  // Ano
    );
    StickCP2.Display.setCursor(140, 11);
    // Exibir data e hora no formato desejado
    StickCP2.Display.printf(
      "%02d:%02d:%02d", 
      tm->tm_hour+1,         // Hora
      tm->tm_min,          // Minuto
      tm->tm_sec           // Segundo
    );
  };  
}
//====================================== Desligar =============================================
void verificarBotaoParaDesligar() {
  static unsigned long tempoInicial = 0;
  static bool botaoPressionado = false;

  int estadoBotao = digitalRead(BUTTON_REBOOT_PIN); // Lê o estado do botão no pino 37

  if (estadoBotao == LOW) { // Verifica se o botão está pressionado
    if (!botaoPressionado) {
      tempoInicial = millis(); // Armazena o tempo inicial quando o botão é pressionado
      botaoPressionado = true;
    }

    // Verifica se o botão foi pressionado por mais de 10 segundos
    if (millis() - tempoInicial >= 10000) {
      esp_deep_sleep_start(); // Desliga o sistema e coloca o ESP32 em deep sleep
      botaoPressionado = false; // Reseta a variável para não alternar continuamente
    }
  } else {
    botaoPressionado = false; // Reseta quando o botão é solto
  }
}
//====================================== Funções do menu principal ============================
void hub() {
    // Código para funcionalidade Hub
}

void RF433() {
    uint8_t buf[RH_ASK_MAX_MESSAGE_LEN]; // Buffer para armazenar a mensagem recebida
    uint8_t buflen = sizeof(buf);        // Tamanho do buffer
    StickCP2.Display.setCursor(2, 50);
    // Verifica se há uma mensagem recebida
    if (driver.recv(buf, &buflen)) {
        buf[buflen] = '\0'; // Garante que a mensagem é uma string terminada em null
        String mensagemRecebida = String((char*)buf); // Converte para String
        M5.Display.println("Mensagem RF433: " + mensagemRecebida);
    } else {
        M5.Display.println("Nenhuma mensagem RF433 recebida");
    }
    delay(100); // Pequena pausa para evitar sobrecarga
}

void rfid() {
    // Código para funcionalidade RFID
}
void emitir_sinal() {
    StickCP2.Display.println();
    StickCP2.Display.print(F("Send now: address=0x1111, command=0x"));
    StickCP2.Display.print(sCommand, HEX);
    StickCP2.Display.print(F(", repeats="));
    StickCP2.Display.print(sRepeats);
    StickCP2.Display.println();
    StickCP2.Display.drawString("IR NEC SEND", StickCP2.Display.width() / 2,
                                StickCP2.Display.height() / 2 - 20);
    StickCP2.Display.drawString("ADDR:0x1111", StickCP2.Display.width() / 2,
                                StickCP2.Display.height() / 2);
    StickCP2.Display.drawString("CMD:0x" + String(sCommand, HEX),
                                StickCP2.Display.width() / 2,
                                StickCP2.Display.height() / 2 + 20);
    StickCP2.Display.println(F("Send standard NEC with 16 bit address"));
    StickCP2.Display.fillCircle(40, 65, 20, GREEN);
    IrSender.sendNEC(0x1111, sCommand, sRepeats);
    sCommand += 1;
    delay(10);
    StickCP2.Display.fillCircle(40, 65, 20, YELLOW);
    delay(10);
}
void gps() {
    // Código para funcionalidade GPS

}
void config() {
    // Código para funcionalidade Configuração

}
void IMU() {
  auto imu_update = StickCP2.Imu.update();
  if (imu_update) {
      StickCP2.Display.setCursor(5, 20);
      auto data = StickCP2.Imu.getImuData();
      // The data obtained by getImuData can be used as follows.
      // data.accel.x;      // accel x-axis value.
      // data.accel.y;      // accel y-axis value.
      // data.accel.z;      // accel z-axis value.
      // data.accel.value;  // accel 3values array [0]=x / [1]=y / [2]=z.
      // data.gyro.x;      // gyro x-axis value.
      // data.gyro.y;      // gyro y-axis value.
      // data.gyro.z;      // gyro z-axis value.
      // data.gyro.value;  // gyro 3values array [0]=x / [1]=y / [2]=z.
      // data.value;  // all sensor 9values array [0~2]=accel / [3~5]=gyro /
      //              // [6~8]=mag
      StickCP2.Display.printf("ax:%f  ay:%f  az:%f\r\n", data.accel.x, data.accel.y,
                    data.accel.z);
      StickCP2.Display.printf("gx:%f  gy:%f  gz:%f\r\n", data.gyro.x, data.gyro.y,
                    data.gyro.z);
      StickCP2.Display.printf("IMU:\r\n");
      StickCP2.Display.printf("%0.2f %0.2f %0.2f\r\n", data.accel.x,
                              data.accel.y, data.accel.z);
      StickCP2.Display.printf("%0.2f %0.2f %0.2f\r\n", data.gyro.x,
                              data.gyro.y, data.gyro.z);
  }
  delay(10);
}
void plotGraph(float accelX, float accelY, float accelZ, float gyroX, float gyroY, float gyroZ) {
    // Limpa a área de gráfico, definindo uma seção específica na tela
    StickCP2.Display.fillRect(0, 80, 160, 80, BLACK);  // por exemplo, de 0x80 a 0x160

    // Desenhar uma linha ou ponto para cada eixo de aceleração
    StickCP2.Display.drawLine(10, 100, 10 + (int)(accelX * 10), 100, RED);   // X em vermelho
    StickCP2.Display.drawLine(10, 120, 10 + (int)(accelY * 10), 120, GREEN); // Y em verde
    StickCP2.Display.drawLine(10, 140, 10 + (int)(accelZ * 10), 140, BLUE);  // Z em azul

    // Desenhar uma linha ou ponto para cada eixo do giroscópio, em outra parte da tela
    StickCP2.Display.drawLine(80, 100, 80 + (int)(gyroX * 10), 100, YELLOW); // Gyro X
    StickCP2.Display.drawLine(80, 120, 80 + (int)(gyroY * 10), 120, CYAN);   // Gyro Y
    StickCP2.Display.drawLine(80, 140, 80 + (int)(gyroZ * 10), 140, MAGENTA);// Gyro Z
}

// Configurações do gráfico
const int graphX = 160;       // Coordenada X inicial do gráfico
const int graphY = 30;        // Coordenada Y inicial do primeiro gráfico
const int graphWidth = 70;    // Largura do gráfico
const int graphHeight = 28;   // Altura do gráfico


// Buffers circulares e índices para os três gráficos
int accelDataX[graphWidth];    // Buffer para o eixo X
int accelDataY[graphWidth];    // Buffer para o eixo Y
int accelDataZ[graphWidth];    // Buffer para o eixo Z
int indexX = 0, indexY = 0, indexZ = 0;  // Índices dos buffers

/**
 * Função para desenhar um gráfico de aceleração.
 * @param accelValue O valor atual de aceleração.
 * @param buffer O buffer circular para armazenar os dados do gráfico.
 * @param index O índice atual do buffer.
 * @param offsetY O deslocamento vertical do gráfico.
 * @param lineColor A cor da linha do gráfico.
 */
void plotGraph(int accelValue, int buffer[], int &index, int offsetY, uint16_t lineColor, uint16_t bgColor) {
    // Limpa a área do gráfico
    StickCP2.Display.fillRect(graphX, offsetY, graphWidth, graphHeight, bgColor);

    // Mapear o valor de aceleração para a altura do gráfico, respeitando os limites
    int mappedValue = map(accelValue, -1100, 1100, offsetY + graphHeight, offsetY);
    mappedValue = constrain(mappedValue, offsetY, offsetY + graphHeight);

    // Adiciona o valor atual ao buffer circular
    buffer[index] = mappedValue;

    // Incrementa o índice circularmente
    index = (index + 1) % graphWidth;

    // Desenha o gráfico conectando pontos consecutivos no buffer
    for (int i = 0; i < graphWidth - 1; i++) {
        int currentIndex = (index + i) % graphWidth;
        int nextIndex = (index + i + 1) % graphWidth;

        StickCP2.Display.drawLine(
            graphX + i, buffer[currentIndex],      // Ponto atual
            graphX + i + 1, buffer[nextIndex],     // Ponto seguinte
            lineColor                              // Cor da linha
        );
    }
}
/**
 * Atualiza os gráficos com dados do IMU.
 */
void updateIMUGraph() {
    // Atualiza os dados do IMU
    if (StickCP2.Imu.update()) {
        auto data = StickCP2.Imu.getImuData();
        // Converte os valores de aceleração para mG (miligravidade)
        int accelX = static_cast<int>(data.accel.x * 1000);
        int accelY = static_cast<int>(data.accel.y * 1000);
        int accelZ = static_cast<int>(data.accel.z * 1000);
        // Desenha os gráficos com cores diferentes
        plotGraph(accelX, accelDataX, indexX, graphY, StickCP2.Display.color565(255, 0, 0),StickCP2.Display.color565(100, 100, 100));  // Vermelho
        plotGraph(accelY, accelDataY, indexY, graphY * 2, StickCP2.Display.color565(0, 255, 0),StickCP2.Display.color565(120, 120, 120));  // Verde
        plotGraph(accelZ, accelDataZ, indexZ, graphY * 3, StickCP2.Display.color565(0, 0, 255),StickCP2.Display.color565(140, 140, 140));  // Azul
    }
    // Controla a velocidade de atualização
    delay(100);  // Ajuste conforme necessário
}

//====================================== Menu ==============================================
void displayMenuInit() {
  int up = digitalRead(BUTTON_CHANGE_PIN);
  int down = digitalRead(BUTTON_EXECUTE_PIN);
  int enter = digitalRead(BUTTON_REBOOT_PIN);
  StickCP2.Display.display(); 
  StickCP2.Display.setTextColor(color_letra);
  StickCP2.Display.setFont(&fonts::Orbitron_Light_24);
  StickCP2.Display.setTextSize(0.5);
  // ARMAZENA VARIÁVEIS PELOS BOTÕES 
  if (up == LOW && down == LOW) {
    return;
  };
  if (up == LOW) {
    selected = selected - 1;
    StickCP2.Speaker.tone(6000, 100);
    delay(100);
  } else if (down == LOW) {
    selected = selected + 1;
    StickCP2.Speaker.tone(9000, 100);
    delay(100);
  } else if (enter == LOW) {
    entered = selected;
    menu_selec = 1;
    StickCP2.Speaker.tone(5555, 200);
    selected = 0;
  };
  StickCP2.Display.setCursor(0, 0);
  StickCP2.Display.println(F(" Menu principal "));
  StickCP2.Display.println("");
  //MENUS 
  for (int i = 0; i < 7; i++) {
    if (i == selected) {
      StickCP2.Display.setTextColor(color_fundo, color_letra);
    } else {
      StickCP2.Display.setTextColor(color_letra);
    };
    if (entered == 0 || selecionado == " Hub ") { // MENU PRINCIPAL
      const char *options_menu[7] = {
        " Hub ",
        " Bluetooth ",
        " RF433 ",
        " IMU ",
        " Emitir sinal ",
        " GPS ",
        " Config. "
      };
      if (menu_selec == 1) {
        selecionado = options_menu[entered];
        menu_selec = 0;
      };
      StickCP2.Display.println(options_menu[i]);
      if (selected < 0) {
        selected = 6;
      } else if (selected > 6) {
        selected = 0;
      };
    } else if (entered == 6 || selecionado == " Config. ") { // MENU CONFIGURAÇÃO
      const char *options_menu_config[3] = {
        " Voltar ",
        " Mudar a cor ",
        " Mudar hora"
      };
      if (menu_selec == 1) {
        menu_selec = 0;
        selecionado = options_menu_config[entered];
      };
      StickCP2.Display.println(options_menu_config[i]);
      if (i == (sizeof(options_menu_config) / sizeof(options_menu_config[0]))-1) {
        i = 7;
      };
      if (selected < 0) {
        selected = 2;
      } else if (selected > 3) {
        selected = 0;
      };
    } else {
      //nada
    };
  };
  StickCP2.Display.println(""); 
  StickCP2.Display.setTextColor(color_letra);
  hora_global();
  StickCP2.Display.setCursor(100, 119);
  StickCP2.Display.println(selected);
  StickCP2.Display.setCursor(130, 119);
  StickCP2.Display.println(entered);
  StickCP2.Display.setCursor(100, 105);
  StickCP2.Display.println(selecionado);
  StickCP2.Display.setCursor(1, 119);
  StickCP2.Display.setTextColor(BatteryPercentageDisplay());
  StickCP2.Display.printf(
      "Batery: %02d \% ",
      readBatteryPercentage()
  );
  if (selecionado == " Emitir sinal ") {
    emitir_sinal();
  };
  if (selecionado == " IMU ") {
    IMU();
    updateIMUGraph();
  };
  if (selecionado == " RF433 ") {
    RF433();  // Verifica se há sinais RF433 disponíveis
    delay(10);  // Pequena pausa para evitar sobrecarga
  };
  delay(300);
  StickCP2.Display.clearDisplay();
};
//====================================== Iniciarlização ====================================
void setup() {
  auto cfg = M5.config();
  M5.begin();
  M5.update();
  StickCP2.Display.setRotation(rotation);
  StickCP2.Display.setTextColor(color_letra);
  M5.Display.setFont(&fonts::Orbitron_Light_24);
  M5.Display.setTextSize(0.5);
  pinMode(BUTTON_CHANGE_PIN, INPUT_PULLUP);
  pinMode(BUTTON_EXECUTE_PIN, INPUT_PULLUP);
  pinMode(BUTTON_REBOOT_PIN, INPUT_PULLUP);
  IrSender.begin(DISABLE_LED_FEEDBACK);  // Start with IR_SEND_PIN as send pin
  IrSender.setSendPin(IR_TX_PIN);
  connectar_Wifi();  
  StickCP2.Display.setCursor(10, 50);
  M5.Display.println("Iniciando RF433...");
  // Inicializa o driver RF433
  if (!driver.init()) {
      M5.Display.println("Falha ao inicializar o driver RF433");
      while (true); // Trava o programa se a inicialização falhar
  }
  M5.Display.println("RF433 inicializado com sucesso");
};
//====================================== Loop ====================================
void loop() {
  verificarBotaoParaDesligar();
  // Chamadas para outras funções
  displayMenuInit();
  readBatteryPercentage();
  // Pequeno atraso para evitar atualizações muito rápidas
  delay(800);  // Usei 1000 ms (1 segundo)
}
