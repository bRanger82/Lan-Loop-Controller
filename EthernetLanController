#include <SPI.h>
#include <Ethernet.h>

#define AWS     4      // Anwesenheitsschleife
#define DS      5      // Durchfahrtsschleife
#define AWS_INV 7      // if set to GND, the AWS output is inverted (HIGH will be LOW and other way round)
#define DS_INV  8      // if set to GND, the DS output is inverted (HIGH will be LOW and other way round)

// Variable to store the HTTP request
String header;

// Enter a MAC address and IP address for your controller below.
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0x0E, 0x0D };

//void begin(uint8_t *mac_address, IPAddress local_ip, IPAddress dns_server, IPAddress gateway, IPAddress subnet);
IPAddress ip(10, 16, 41, 101);  // IP-Address of the device
IPAddress ds(10, 16, 41, 101);  // Domain Name Server (in this case the local device)
IPAddress gw(10, 16, 201, 254); // Gateway
IPAddress sn(255, 255, 0, 0);   // Subnet-Mask

// Initialize the Ethernet server library
// with the IP address and port you want to use
// (port 80 is default for HTTP):
EthernetServer server(80);

void setup() {
  // Open serial communications and wait for port to open:
  Serial.begin(9600);
  // Start the Ethernet connection and the server:
  Ethernet.begin(mac, ip, ds, gw, sn);
  server.begin();
  Serial.print("IP:");
  Serial.println(Ethernet.localIP());
  // Prepare the ports
  analogWrite(AWS, 0);
  pinMode(AWS, OUTPUT);
  analogWrite(DS, 0);
  pinMode(DS, OUTPUT);
  // Prepare the invert pins, usually set to Vcc via pullup, but can be set to GND
  pinMode(AWS_INV, INPUT_PULLUP);
  pinMode(DS_INV,  INPUT_PULLUP);
}

/*
 * Redirect method, when button on the HTML page is pressed the site is reloaded.
*/
void redirect(EthernetClient client) 
{
  client.println("HTTP/1.1 307 Temporary Redirect");
  client.println("Location: /");
  client.println("Connection: Close");
  client.println();
  delay(1);
  client.stop();
  header = "";
}

/*
 * Sets the state of a pin. 
 * Active can be either HIGH or LOW.
*/
void SetPin(int pin, int active)
{
  /*
   * Some relais needs an inverted signal (some of them are activated if the output is set to LOW.
   * If the relais is inverted this does not match the state overview on the HTML page (page will show ACTIVE but the relais is not active).
   * So if the _INV pin is set to GND, the output is inverted.
   * E.g. 
   *      if AWS_INV => GND then AWS activate command will set AWS pin from Vcc to GND.
   *      if AWS_INV => not connected then AWS activate command will set AWS pin from GND to Vcc
   * Same for the DS pin.
  */
  if (pin == AWS)
  {
    if (digitalRead(AWS_INV) == LOW) // usually internal_pullup, so high. If connected to GND, invert the AWS signal
    {
      digitalWrite(pin, !active);
    } else
    {
      digitalWrite(pin, active);
    }
  } else if (pin == DS)
  {
    if (digitalRead(DS_INV) == LOW)// usually internal_pullup, so high. If connected to GND, invert the DS signal
    {
      digitalWrite(pin, !active);
    } else
    {
      digitalWrite(pin, active);
    }
  }
}

int CheckPin(int pin)
{
  if (pin == AWS)
  {
    if (digitalRead(AWS_INV) == LOW) // usually internal_pullup, so high. If connected to GND, invert the AWS signal
    {
      return !(digitalRead(pin));
    } else
    {
      return digitalRead(pin);
    }
  } else if (pin == DS)
  {
    if (digitalRead(DS_INV) == LOW)// usually internal_pullup, so high. If connected to GND, invert the DS signal
    {
      return !(digitalRead(pin));
    } else
    {
      return digitalRead(pin);
    }
  } 
}

void loop() 
{
  // listen for incoming clients
  EthernetClient client = server.available();
  if (client) {
    while (client.connected()) 
    {
      if (client.available()) 
      {
        char c = client.read();
        header += c;
        Serial.write(c);
        // if you've gotten to the end of the line (received a newline
        // character) and the line is blank, the http request has ended,
        // so you can send a reply
        if (c == '\n') 
        {
          if (header.indexOf("GET /4/on") >= 0) 
          {
            SetPin(AWS, HIGH);
            delay(1);
            redirect(client);
            return;
          } else if (header.indexOf("GET /4/off") >= 0) 
          {
            SetPin(AWS, LOW);
            delay(1);
            redirect(client);
            return;
          } else if (header.indexOf("GET /5/on") >= 0) 
          {
            SetPin(DS, HIGH);
            delay(1);
            redirect(client);
            return;
          } else if (header.indexOf("GET /5/off") >= 0) 
          {
            SetPin(DS, LOW);
            delay(1);
            redirect(client);
            return;
          }
          header = "";
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println("Connection: close");  // the connection will be closed after completion of the response
          
          //client.println("Refresh: 5");  // refresh the page automatically every 5 sec
          client.println();
          client.println("<!DOCTYPE HTML>");
          client.println("<html>");
          client.println("<head><meta name=\"viewport\" content=\"width=device-width,initial-scale=1\">");
          client.println("<link rel=\"icon\" href=\"data:,\">");
          client.println("<style>html {font-family: Helvetica;display:inline-block;margin:0px auto;text-align:center;}");
          client.println(".btn {background-color:#195B6A;border:none;color:white;padding:16px 40px;");
          client.println("text-decoration:none;font-size:30px;margin:2px;cursor:pointer;}");
          client.println(".btn2 {background-color:#77878A;}</style></head>");
          
          // Web Page Heading
          client.println("<body>");
          // Display current state, and ON/OFF buttons for GPIO 5  
          // If the output0State is off, it displays the ON button       
          if (CheckPin(AWS) == LOW)
          {
            client.println("<p>AWS State NOT ACTIVE</p>");
            client.println("<p><a href=\"/4/on\"><button class=\"btn\">ACTIVATE AWS</button></a></p>");
          } else 
          {
            client.println("<p>AWS State ACTIVE</p>");
            client.println("<p><a href=\"/4/off\"><button class=\"btn btn2\">DEACTIVATE AWS</button></a></p>");
          } 
          if (CheckPin(DS) == LOW)
          {
            client.println("<p>DS State NOT ACTIVE</p>");
            client.println("<p><a href=\"/5/on\"><button class=\"btn\">ACTIVATE DS</button></a></p>");
          } else 
          {
            client.println("<p>DS State ACTIVE</p>");
            client.println("<p><a href=\"/5/off\"><button class=\"btn btn2\">DEACTIVATE DS</button></a></p>");
          } 
          client.println("</body>");
          client.println("</html>");
          break;
        }
      }
    }
    // give the web browser time to receive the data
    delay(1);
    // close the connection:
    client.stop();
    header = "";
  }
}
