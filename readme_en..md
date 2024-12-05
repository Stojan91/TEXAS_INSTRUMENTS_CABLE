Oczywiście! Poniżej znajduje się przetłumaczony plik README w formacie Markdown (`README_EN.md`), gotowy do edycji na GitHubie. Pseudo-kod oraz inne sekcje zostały zachowane, a całość jest przygotowana, abyś mógł łatwo wkleić zawartość do edytora na GitHubie.

---

# ESP8266 and TI-85 Communication

![ESP8266](example.jpg)

To ensure communication between the ESP8266 microcontroller and the Texas Instruments TI-85 calculator, several technical aspects need to be considered. The TI-85 uses a serial port (RS232) for communication with a computer, so this port must be connected to the ESP8266 microcontroller. Using this port, we can transfer data between the calculator and the computer, with the ESP8266 acting as an interface.

## Assumptions

1. **Communication via RS232**: The TI-85 uses the RS232 standard for serial communication with a computer.
2. **UART Interface in ESP8266**: The ESP8266 is capable of UART (Universal Asynchronous Receiver/Transmitter) communication, which is compatible with RS232 at the correct voltage levels.
3. **Data Transmission**: We will read data from the calculator and then display it on a computer using a WiFi connection.

## What We Will Need:

- **ESP8266**: A microcontroller with built-in WiFi that will serve as an intermediary for communication.
- **RS232 to TTL Converter Module** (e.g., MAX232) – because the ESP8266 uses TTL voltage levels (0V/3.3V), while the TI-85 uses RS232 voltage levels (-12V to +12V).
- **RS232 Serial Cable** – to connect the TI-85 to the converter.
- **WiFi Connection** – for transmitting data to a computer or a computer application.

## Connections:

### 1. **ESP8266 and RS232 to TTL Converter**:
- The RS232 converter (MAX232) has two main pins: **`TX`** (transmit) and **`RX`** (receive).
- Connect the **`TX`** pin of the converter to the **`RX`** pin of the ESP8266.
- Connect the **`RX`** pin of the converter to the **`TX`** pin of the ESP8266.
- Connect the GND of the converter to the GND of the ESP8266.
- Be sure to connect the voltage levels correctly, as the ESP8266 operates at 3.3V, and RS232 uses voltage levels that require conversion.

### 2. **TI-85 and RS232 to TTL Converter**:
- Connect the serial port of the TI-85 (typically DB9) to the input of the RS232 to TTL converter.
- The relevant lines are TX, RX, and ground (GND).

### 3. **ESP8266 and Computer**:
- The ESP8266 will act as a WiFi bridge, transmitting data from the TI-85 to the computer (e.g., using WebSockets, HTTP, or MQTT).

## Sample Code for ESP8266 (Arduino IDE)

This code assumes we will be transmitting data from the TI-85 to the computer through the ESP8266. We use serial communication (UART) between the ESP8266 and the calculator, then transmit the data via WiFi (e.g., to a web application on the computer).

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "Your_SSID";
const char* password = "Your_Password";
WiFiServer server(80);

#define RX_PIN 3   // RX Pin on ESP8266 (D3)
#define TX_PIN 1   // TX Pin on ESP8266 (D10)

HardwareSerial mySerial(1);  // Use hardware serial port 1 on the ESP8266

void setup() {
  Serial.begin(115200);  // Communication with the computer
  mySerial.begin(9600, SERIAL_8N1, RX_PIN, TX_PIN);  // Communication with the TI-85 (baud rate 9600 bps)
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    Serial.println("New connection");
    while (client.connected()) {
      if (mySerial.available()) {
        char data = mySerial.read();  // Read data from the TI-85
        client.write(data);  // Send data to the client
        Serial.print(data);  // Print data to the serial monitor
      }
    }
    client.stop();
    Serial.println("Connection closed");
  }

  // Transmit data from the computer to the TI-85
  if (Serial.available()) {
    char data = Serial.read();  // Read data from the serial port
    mySerial.write(data);  // Send data to the TI-85
  }
}
```

## Operation Description:

### 1. **WiFi**: The code connects to a WiFi network using the provided SSID and password.
### 2. **Serial Communication**:
   - **mySerial** is the serial instance used for communication with the TI-85. We read data from the TI-85 and send it to the client (e.g., a web browser).
   - **Serial** is used for communication with the computer, allowing commands to be sent to the TI-85.
### 3. **HTTP Server**: A simple HTTP server is started on port 80, waiting for client connections (e.g., a web browser).
### 4. **Data Transmission**: Once a connection is made, the ESP8266 receives data from the TI-85 and sends it to the client via WiFi. The data can be displayed on a web page or received by an application on the computer.

## Summary:

- **HardwareSerial mySerial** enables communication between the ESP8266 and the TI-85 via UART.
- **WiFiClient** allows the transmission of data to the computer via WiFi, enabling visualization of TI-85 data.
- Using an HTTP server in the ESP8266 allows easy data retrieval on the computer via a web browser.

The code above is a basic foundation that can be expanded with additional features, such as file handling or advanced interaction with the TI-85 calculator.

To read data from the **TI-85** on a computer, you need to create an appropriate application or use an existing solution that facilitates communication via the serial port. Below are two main methods:

### 1. **Using a Serial Terminal (e.g., PuTTY)**
### 2. **Writing a Simple Python Program to Read Data from the Calculator**

Both methods can use the **ESP8266** as an intermediary in the communication between the TI-85 calculator and the computer (via WiFi).

### 1. Read Data Using a Serial Terminal (e.g., PuTTY)

If you just want to read data from the calculator on your computer without writing your own application, the easiest solution is to use **PuTTY** or **Tera Term**, which allow you to connect to a serial port via WiFi (with the ESP8266 acting as a communication bridge).

#### Steps:

1. **Install PuTTY or Tera Term**:
   - [Download PuTTY](https://www.putty.org/) or Download Tera Term.
2. **Configure WiFi Connection with ESP8266**:
   - Ensure the ESP8266 is properly configured and connected to a WiFi network. It should transmit data from the TI-85 to the computer via WiFi, as shown in the example code.
3. **Configure PuTTY/Tera Term for Serial Port Connection**:
   - **ESP8266 IP Address**: Open PuTTY and set the ESP8266 IP address and port (default port 80 for HTTP).
   - You can also configure PuTTY to connect to the serial port directly if you have a direct connection to the TI-85 calculator via an RS232-USB converter.
4. **Open the Connection**:
   - Once the connection to the ESP8266 (or directly to the device) is established, PuTTY/Tera Term should begin receiving data from the calculator and display it on the computer screen.

### 2. Read Data Using a Python Program

If you want to create your own software to read data from the TI-85 calculator through the ESP8266, the best solution is to use **Python** and the **pySerial** library for serial communication.

#### Steps:

1. **Install Python and the pySerial Library**:
   - If you don’t have Python installed, download it from the [official website](https://www.python.org/downloads/).
   - Install the **`pySerial`** library, which allows serial communication:

   ```
   pip install pyserial
   ```

2. **Create a Simple Python Program**:

   Below is an example Python script that reads data transmitted via the serial port from the TI-85 calculator through the ESP8266 (in this case, over WiFi; communication can happen via serial or HTTP depending on ESP8266 configuration).

```python
import serial
import time

# Set up the serial port (check your device's port)
ser = serial.Serial('COM3', 9600, timeout=1)  # Change COM3 to your correct port (e.g., /dev/ttyUSB0 on Linux)
time.sleep(2)  # Wait for the connection to stabilize

while True:
    if ser.in_waiting > 0:
       

 data = ser.readline().decode('utf-8').strip()  # Read data from the serial port
        print(f"Data received: {data}")
```

#### Explanation:

- **serial.Serial('COM3', 9600, timeout=1)**: Creates a serial connection to the device. Here, `COM3` is the port on Windows, or `/dev/ttyUSB0` on Linux.
- **ser.readline()**: Reads the data from the serial port. The data is decoded to UTF-8, and **`strip()`** removes extra white spaces.
- The program continuously reads and prints the data from the serial port.

#### Connecting:

- **Serial Port on Windows**: If you are using Windows, make sure the RS232 to USB converter is assigned to the correct COM port.
- **Serial Port on Linux**: On Linux, the device might be assigned to `/dev/ttyUSB0` or `/dev/ttyAMA0`.

3. **Run the Script**:
   - After running the Python script on your computer, it will start receiving data from the TI-85 calculator and display it in the console.

**Sample Data That May Appear on the Screen:**

```
Data received: 8
Data received: 5
Data received: 3
```

### 3. Alternative Solution – Using Pre-made Programs for TI-85 Communication

If you don’t want to write your own software, there are pre-made solutions that can assist in communicating with the TI-85 calculator. One of them is **TI Connect**, the official software from Texas Instruments, which allows synchronization of TI calculators with a computer. It typically supports USB cables, but if you use the ESP8266 as a serial interface, this might be feasible via the serial port.

## Summary

- **Serial Terminal (e.g., PuTTY)**: A quick solution to read data without writing software. Use this if you just want to read data from the calculator.
- **Python Program**: Use this solution if you want more control over the data and its processing. You can easily expand it with additional features.
- **Pre-made Software (e.g., TI Connect)**: Can be useful but requires compatible connections that fully support Texas Instruments devices.

---

You can now copy this code directly into a file called `README_EN.md` on GitHub. If you need any further adjustments, let me know!
