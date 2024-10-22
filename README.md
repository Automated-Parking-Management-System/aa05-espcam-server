# ESP32 Camera Web Server

This project sets up an **ESP32-CAM** as a web server that captures images at different resolutions and serves them via HTTP. Users can connect to the ESP32 via WiFi and view images captured by the camera at low, medium, or high resolutions.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Libraries Used](#libraries-used)
3. [WiFi Setup](#wifi-setup)
4. [Camera Configuration](#camera-configuration)
5. [Image Serving](#image-serving)
6. [HTTP Endpoints](#http-endpoints)
7. [Main Code Functions](#main-code-functions)
    - [Setup Function](#setup-function)
    - [Loop Function](#loop-function)

## Prerequisites
- **ESP32-CAM** module.
- **Arduino IDE** with ESP32 board support.
- WiFi credentials to allow the ESP32 to connect to a network.

## Libraries Used
The following libraries are used to manage WiFi and camera operations:
- **WiFi.h**: For connecting the ESP32 to a WiFi network.
- **esp32cam.h**: For handling camera operations and capturing images.
- **WebServer.h**: To create a simple web server that handles HTTP requests.

## WiFi Setup
Before running the code, insert your **WiFi credentials** in the `setup()` function:

```cpp
const char* WIFI_SSID = "Your-WiFi-SSID";
const char* WIFI_PASS = "Your-WiFi-Password";
```
The ESP32 will connect to this WiFi network and host a web server, allowing you to access images from the camera.
## Camera Configuration
The camera is configured using the esp32cam library. The following settings are defined:
- Resolution: The resolution can be set to low, medium, or high, depending on the endpoint the user accesses (`/cam-lo.jpg`, `/cam-mid.jpg`, `/cam-hi.jpg`).
- JPEG Quality: The image quality is set to 80% by default.

```cpp
cfg.setPins(pins::AiThinker);
cfg.setResolution(hiRes);
cfg.setBufferCount(2);
cfg.setJpeg(80);
```

### Supported Resolutions
- Low Resolution: 320x240
- Medium Resolution: 350x530
- High Resolution: 800x600

## Image Serving
The code defines three functions that handle the capturing and serving of images based on the selected resolution. Each function changes the camera resolution and serves the captured image in JPEG format via HTTP.

`serveJpg()`
This function captures an image from the camera, checks if the capture was successful, and then serves the image as an HTTP response.
```cpp
void serveJpg() {
  auto frame = esp32cam::capture();
  if (frame == nullptr) {
    server.send(503, "", "");
    return;
  }
  server.setContentLength(frame->size());
  server.send(200, "image/jpeg");
  WiFiClient client = server.client();
  frame->writeTo(client);
}
```

### Resolution Handlers
Each resolution has its own handler that changes the camera's resolution and serves the image.
- **Low Resolution:**

  ```cpp
  void handleJpgLo() {
    esp32cam::Camera.changeResolution(loRes);
    serveJpg();
  }
  ```

- **Medium Resolution:**

  ```cpp
  void handleJpgMid() {
    esp32cam::Camera.changeResolution(midRes);
    serveJpg();
  }
  ```

- **High Resolution:**

  ```cpp
  void handleJpgHi() {
    esp32cam::Camera.changeResolution(hiRes);
    serveJpg();
  }
  ```

## HTTP Endpoints
The following endpoints are available for capturing and viewing images:

- **/cam-lo.jpg:** Serves a low-resolution image (320x240).
- **/cam-mid.jpg:** Serves a medium-resolution image (350x530).
- **/cam-hi.jpg:** Serves a high-resolution image (800x600).

Accessing these endpoints in a browser will display the corresponding image captured by the ESP32 camera.

## Main Code Functions
### Setup Function
The `setup()` function initializes the camera, connects the ESP32 to WiFi, and sets up the web server.
```cpp
void setup() {
  Serial.begin(115200);

  // Camera configuration
  esp32cam::Config cfg;
  cfg.setPins(pins::AiThinker);
  cfg.setResolution(hiRes);
  cfg.setBufferCount(2);
  cfg.setJpeg(80);
  
  bool ok = esp32cam::Camera.begin(cfg);
  Serial.println(ok ? "CAMERA OK" : "CAMERA FAIL");

  // WiFi setup
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
  Serial.println(WiFi.localIP());

  // Server endpoints
  server.on("/cam-lo.jpg", handleJpgLo);
  server.on("/cam-hi.jpg", handleJpgHi);
  server.on("/cam-mid.jpg", handleJpgMid);

  // Start the server
  server.begin();
}
```

### Loop Function
The `loop()` function continuously checks for incoming client requests and handles them using the web server.
```cpp
void loop() {
  server.handleClient();
}
```

## Conclusion
This code provides a simple web server for capturing and viewing images from an ESP32-CAM at different resolutions. By connecting to the ESP32 via WiFi and accessing the provided endpoints, users can view live camera captures in their browser. The setup allows for flexible resolution selection and serves images directly via HTTP.


This `README.md` explains each block of the code and how the camera server operates, providing detailed instructions on how to set up and access the camera images from a web browser.
