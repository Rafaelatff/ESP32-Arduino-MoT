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

#### MoT Libraries

To make the libaries, that already work on the Arduino IDE also work on the Visual Studio Code IDE, I have to do two things:

First, prepare the libs according to documentation (README file inside the lib tree):

![image](https://github.com/Rafaelatff/ESP32-Arduino-MoT/assets/58916022/62afe583-1c8f-469a-8d81-73ee3780eb17)

Then I had to make use of the `extern` to [Share A Global Variable Across Multiple Files](https://www.youtube.com/watch?v=Wqm1tSzmqkY).


### N3_Python_RSSI_PSR.py
