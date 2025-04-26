# PRÁCTICA 6: BUSES DE COMUNICACIÓN II (SPI) 
En esta práctica tenemos como objetivo comprender el funcionamiento del bus SPI. Para ello estaremos trabajando con tarjetas tanto SD como con tarjetas RFID y sus respectivos lectores (buses SPI).

## EJERCICIO PRÁCTICO 1
En este primer ejercicio práctico hemos realizado el montaje de un bus SPI el cual consiste en un lector de tarjetas SD.
A continuación hemos modificado el código que se nos ha proporcionado en la práctica de manera que se cree un objeto de tipo “File” el cual se usará para leer y escribir en la tarjeta SD. Una vez declarado, el programa inicializa la comunicación serial para iniciar la tarjeta SD mediante el GPIO 10, usado como pin de “Chip Select” (CS). Posteriormente el programa abre un archivo .txt dentro de la tarjeta y en caso de que no exista ese archivo o el nombre no coincida con ninguno de los archivos existentes se crea uno nuevo en el cuál se escribe una frase (En este caso: “Hola desde ESP32-S3”) y cierra el archivo para guardar los cambios. Finalmente vuelve a abrir el archivo .txt para leer el contenido y mostrarlo por el serial monitor. 

El código es el siguiente:

```
#include <SPI.h>
#include <SD.h>


File myFile;


void setup() {
    Serial.begin(115200);
    Serial.print("Iniciando SD...");


    if (!SD.begin(10)) {  // CS en GPIO 10
        Serial.println("No se pudo inicializar");
        return;
    }
    Serial.println("Inicialización exitosa");


    myFile = SD.open("/archivo.txt", FILE_WRITE);
    if (myFile) {
        myFile.println("Hola desde ESP32-S3");
        myFile.close();
        Serial.println("Escritura exitosa");
    } else {
        Serial.println("Error al abrir el archivo");
    }


    myFile = SD.open("/archivo.txt");
    if (myFile) {
        Serial.println("Contenido del archivo:");
        while (myFile.available()) {
            Serial.write(myFile.read());
        }
        myFile.close();
    } else {
        Serial.println("Error al leer el archivo");
    }
}


void loop() {
}

```

## EJERCICIO PRÁCTICO 2
En este segundo ejercicio hemos logrado que nuestra ESP32-S3 pueda leer tarjetas RFID usando un lector RC522 y muestre por el terminal la UID (su identificador único) de dicha tarjeta.
Para ello hemos vuelto a modificar un programa otorgado en el enunciado de la práctica de manera que después de definir las librerías necesarias (en este caso hemos tenido que instalar una librería desde el IDE de PlatformIO para el lector RC522) hemos definido los pines (en este caso hemos usado el GPIO 8 y GPIO 9, donde GPIO 9 ha sido el pin de “Chip Select”). Una vez definidos el programa crea un objeto para manejar el lector e inicia la comunicación serial, la comunicación SPI y el lector de tarjetas RFID. Por último hemos determinado un loop que detecte una tarjeta, lea su UID y lo imprima en hexadecimal. Una vez se obtiene el UID se detiene la comunicación con esa tarjeta para que pueda detectar una nueva.

El código es el siguiente:

```
#include <SPI.h>
#include <MFRC522.h>


#define RST_PIN 8    // Pin de reset del RC522
#define SS_PIN  9    // Pin SS (SDA) del RC522


MFRC522 mfrc522(SS_PIN, RST_PIN);  // Crear objeto RFID


void setup() {
    Serial.begin(115200);  // Iniciar la comunicación serial
    SPI.begin();           // Iniciar el bus SPI
    mfrc522.PCD_Init();    // Iniciar el módulo RFID
    Serial.println("Escaneando tarjetas RFID...");
}


void loop() {
    // Verificar si hay una nueva tarjeta presente
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
        Serial.print("UID de tarjeta: ");


        // Leer el UID de la tarjeta y mostrarlo en hexadecimal
        for (byte i = 0; i < mfrc522.uid.size; i++) {
            Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
            Serial.print(mfrc522.uid.uidByte[i], HEX);
        }
        Serial.println();


        mfrc522.PICC_HaltA();  // Detener la lectura de la tarjeta
    }
}

```
