# ESP32-Arduino-MoT
This repositorie was created to aid me during my studies of MoT protocol + LQE.

## Theory
* LQE- Link Quality Estimator.
* MoT - Management over Tunneling.
* PSR - Packet Success Ratio.
* RSSI - Received Signal Strength Indication.

## Coding

### ESP32-Arduino-MoT

#### RSSI

Two main LQE are the RSSI reading and the LQE. Starting by the RSSI, the first challenge is to pass it correctly to the N3. 

![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/d69a76b3-e13f-4b65-ad48-261da25e512f)

```cpp
  int RSSI_ul = WiFi.RSSI();
  int rssi = WiFi.RSSI();
  Serial.print("RSSI: ");
  Serial.println(rssi);
```
Wrong results:
![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/0236a170-ec72-41dd-9a94-c9f60c77193c)

My coleague Montali already studied this, so I just studied and tested this code. At this point, I am not separing the uplink (ESP32 to AP) and downlin (AP to ESP32) metrics.

```ccp
// Global Variables
int RSSI_dl = 0; 
int RSSI_ul = 0;
int rssi = 0;
float RSSI_dBm_ul = 0;  //RSSI de Uplink em dBm
float RSSI_dBm_dl = 0;

void Phy_dBm_to_Radiuino(){// Funcao que transforma RSSI em dBm da leitura do WiFi para RSSI utilizada no radiuino (complemento de 2 com passo 1/2 e 74 de offset)
   /*tabela usada durante a criacao da funcao
   *   dBm     RSSI  
   *  -10,5   127
   *  -74     0
   *  -138    128
   *  -74,5   255
   */
   
  if(RSSI_dBm_ul > -10.5){ // Caso a RSSI medida esteja acima do valor superior -10,5 dBm
   RSSI_dl = 127; // equivalente a -10,5 dBm 
  }

  if(RSSI_dBm_dl <= -10.5 && RSSI_dBm_dl >= -74){ // Caso a RSSI medida esteja no intervalo [-10,5 dBm e -74 dBm]
   RSSI_dl = ((RSSI_dBm_dl +74)*2) ;
  }

  if(RSSI_dBm_dl < -74){ // Caso a RSSI medida esteja no intervalo ]-74 dBm e -138 dBm]
   RSSI_dl = (((RSSI_dBm_dl +74)*2)+256) ;
  }
}
```

![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/8da39440-aab3-4606-a184-f1e0ef1b8a5c)

The N3 Python code also performed some calculation.

```py
         # RSSI Uplink
         byte0 = Pacote_RX[0] # RSSI de Downlink (Router -> ESP32)
         if byte0 > 128:
            RSSId = ((byte0-256)/2.0)-74
         else:
            RSSId = (byte0/2.0)-74
         # RSSI Downlink
         byte1 = Pacote_RX[1] # RSSI de Uplink (ESP32 -> Router)
         if byte1 > 128:
            RSSIu = ((byte2-256)/2.0)-74
         else:
            RSSIu = (byte1/2.0)-74
```

#### MoT Libraries

To make the libaries, that already work on the Arduino IDE also work on the Visual Studio Code IDE, I have to do two things:

First, prepare the libs according to documentation (README file inside the lib tree):

![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/62afe583-1c8f-469a-8d81-73ee3780eb17)

Then I had to make use of the `extern` to [Share A Global Variable Across Multiple Files](https://www.youtube.com/watch?v=Wqm1tSzmqkY).

#### LED Control

![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/b5024985-f667-444c-a015-cd99cab65ec1)

The code programmed on the ESP32 is showed next:

```c
void App_initialize(){ // Função de inicialização da camada de Aplicação
  pinMode(LED, OUTPUT);  // Define o pino do LED como saída
}

void App_send(){ // Função de envio da Camada de Aplicação  
  Pacote_TX [17] = byte17; // Status do LED
  
  Transp_send(); //Chama a função de envio da Camada de Transporte
}

void App_receive(){
  byte17 = Pacote_RX[17]; // Retira do Pacote o Status do LED verde
  digitalWrite(LED , byte17);    // Liga ou desliga o LED verde de acordo com o status
 
  App_send(); // Chama a função de envio da Camada de Aplicação
}
```
And the N3 code is:
```py
# Camada de Aplicação
byte16 = 0 # Tipo de pacote
byte17 = 0 # LED
status_LED = 0

      arquivo = open('comandos.txt', 'r') # Arquivo comandos.txt seta status do LED
      byte17 = int(arquivo.readline())
      arquivo.close()
      
      PacoteTX[17] = byte17
         status_LED = Pacote_RX[17]
```

As result:

![menor8s](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/271bbe31-11d0-4747-84eb-456809dce0a5)

### N3_Python_RSSI_PSR.py
